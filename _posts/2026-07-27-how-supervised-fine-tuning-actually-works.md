---
layout: post
title: "How supervised fine-tuning actually works"
author: "Kuba"
comments: true

# ── Unpublished: reachable at its URL, hidden everywhere else. ──
# On 27 July 2026, delete the four lines below and push.
unlisted: true          # keeps it off /blog and the prev/next arrows
draft: true             # keeps it out of feed.xml (jekyll-feed filters on this)
robots: noindex, nofollow
sitemap: false

description: A visual, ground-up walkthrough of supervised fine-tuning for LLMs — what the loss actually measures, why the off-by-one trips everyone up, what masking really does, and how much of this you can do on one GPU.
---
I spent my garden leave between companies learning, in depth, how transformer models work — how the architectures differ and how they are actually trained. The best way I know to check whether you understand something is to try to explain it, so this is me explaining supervised fine-tuning to myself, in public.

Almost none of this is original to me. Most of it comes from two courses, and I would recommend both without reservation: HuggingFace's [LLM Course](https://huggingface.co/learn/llm-course), which is free, genuinely excellent, and the best explanation of transformers and tokenizers I have found anywhere; and DeepLearning.AI's [Fine-tuning & RL for LLMs: Intro to Post-training](https://www.deeplearning.ai/courses/fine-tuning-and-reinforcement-learning-for-llms-intro-to-post-training), which is where the post-training picture finally clicked for me. If any part of this post leaves you wanting more depth, those are where to go.

It is written for people with a solid technical background but no particular machine learning one. There is some maths, but I have tried to keep it to the parts you can see rather than the parts you have to take on faith, and everything that matters has a picture next to it.

## 1. What a language model actually does

I am not going to dive into transformer internals. There is a lot of nuance in the architectures, and [HuggingFace's material](https://huggingface.co/learn) covers it far better than I would.

The short version is that a language model does one thing: it understands, predicts, and ultimately samples the next token.

Predicting the next token means producing a score for every single entry in the model's vocabulary. For computational reasons those numbers are not probabilities — they are unbounded floats called **logits**. That is what the model literally outputs: a vector with one number per vocabulary entry.

To actually generate text, you need to sample from that, and to sample you first need real probabilities. **Softmax** is the transformation that gets you there: it exponentiates every logit and then divides by the total. After softmax, the whole vocabulary sums to 1.

{% include figures/softmax.html %}

{% include figures/sampling.html %}

What you then do with that distribution is a separate choice. You can take the most probable token every time — **greedy decoding** — or you can genuinely draw from it, which is what temperature and top-p are shaping. Those are the knobs you meet on day one of any inference API.

Taking the most probable token has a name worth knowing, because it turns up in the figures below and in every piece of ML writing you will read: the **argmax**. Literally "the argument that maximises" — not the largest probability itself, but *which* token it belongs to. When a figure marks a row as argmax, it just means "this is the token the model would have picked, if you had asked it to generate here rather than scoring it against an answer you already had."

Here is the thing to carry forward: **sampling is an inference-time activity**. It plays no part in SFT training. We never need the model to pick anything. We only ever need the distribution — and in a moment you will see why that is enough.

## 2. What supervised fine-tuning is for

The goal of SFT is to teach the model format, conversational ability, and an awareness of who is saying what. Base models do not have this. They do not know that "user" means you and "assistant" means them. Frontier labs fix that by showing the model an enormous number of example conversations, and out the other side comes something that can hold a coherent chat.

{% include figures/base-vs-sft.html %}

It is worth being precise about what "teach" means here, because it shapes everything downstream. The prevailing view is that SFT mostly **elicits** abilities the model already picked up during pretraining, rather than installing new ones. The base model has read millions of dialogues; it already knows what a helpful answer looks like. What it lacks is the knowledge that it should be producing one *right now*, rather than continuing your text with three more plausible user questions.

Keep this in mind when you get to the dataset sizes below. A thousand samples cannot teach a model to converse from scratch. It can absolutely tell it which of its existing behaviours to bring to the front.

{% include figures/pipeline.html %}

Essentially every model you actually chat with has been through this. It is not a hard law of training — we will see a counterexample in section 5 — but it is the standard path, and in practice it gets repeated many times, with different datasets, at different stages of a model's life.

## 3. What an SFT dataset looks like

Conceptually a dataset is a list of input–output pairs. In practice you will almost never see it stored as raw concatenated strings. It is structured messages — a list of `{"role": ..., "content": ...}` objects — and the flattening into a single string happens at tokenization time, via the model's **chat template**.

{% include figures/dataset.html %}

That template is not a detail you can skip. It is the model-specific recipe for which special tokens wrap each role, and every model family has its own. If you fine-tune with one template and then serve the model with a different one, you get a model that is quietly worse for reasons that never show up in your loss curve — the training loss will look perfect, because the mismatch only exists at inference. Use the tokenizer's own `apply_chat_template` and do not hand-roll the string.

It is also worth asking where the data comes from. Broadly, two sources: humans writing the answers — expensive, high quality, what the early instruct models were built on — or **distillation**, where you generate answers with a stronger model and train your smaller one on them. Most open datasets you will find today are the second kind. That is fine, but it means your ceiling is roughly the teacher model's behaviour.

Using the right techniques, even small numbers of samples can have an observable impact. The rule of thumb I learned — though I have not personally gone above a few hundred — looks like this:

{% include figures/ladder.html %}

## 4. What actually happens during training

It is called *supervised* because we are directly teaching the model to increase the probability of specific tokens given specific inputs. No exploration, no reward, no preferences. Here is the answer; be more likely to say it.

Let us do this with an example small enough to hold in your head. Imagine a language model with a five-token vocabulary that is quite good at repeating exactly what the user said. We want to teach it something new: answer the user with their tokens in reverse order.

- **Vocabulary:** `<user>`, `<asst>`, `a`, `b`, `<eos>` — that last one means end of sequence, and tells generation to stop
- **Input:** `<user> a b <asst>`
- **Expected output:** `b a <eos>`

First we need to know how the model is doing on this sample. And here is the part that surprises people: **we never let the model generate anything.** We hand it the expected output and ask how likely it was to have produced it. How surprised it was is the loss.

Models predict one token at a time, but our training samples are multiple tokens long. So the loss is calculated for each token separately and then aggregated over the sample.

Because we do not need the model to generate, we can give it the entire concatenated sequence at once:

<div class="fig">
  <div class="fig__frame">
    <p class="fig__eyebrow">What the model is shown during training</p>
    <div class="tok-row">
      <span class="tok tok--prompt">&lt;user&gt;</span>
      <span class="tok tok--prompt">a</span>
      <span class="tok tok--prompt">b</span>
      <span class="tok tok--prompt">&lt;asst&gt;</span>
      <span class="tok tok--completion">b</span>
      <span class="tok tok--completion">a</span>
      <span class="tok tok--completion">&lt;eos&gt;</span>
    </div>
  </div>
</div>

That is it. The whole thing, prompt and answer together, in one go. We then ask the model what it would have predicted at each position, conditioned only on the tokens before it. What comes back, for every position in the sequence, is a distribution over the vocabulary.

There is an off-by-one here that trips up nearly everyone, so let us be explicit: **the distribution at position *n* is a prediction of the token at position *n+1*.** So the distribution that ought to be putting its mass on our first output token `b` does not live at `b` — it lives one slot to the left, at `<asst>`. Every position predicts its successor.

{% include figures/shift.html %}

On the left you see the logits, the actual output of the model. On the right, those logits after softmax. Note again that plain normalisation would also make things sum to 1; it is the exponentiation that makes the *gaps* meaningful.

The loss for a single token is `-log(probability of the expected token)`. Higher probability, smaller loss. Lower probability, larger loss. That is the whole idea. It is called **cross-entropy loss**, and it is the name you will see in every training codebase.

If you ever want to translate a loss number into something intuitive, exponentiate it: `perplexity = exp(mean loss)`. A loss of 2.3 is a perplexity of about 10, which reads as "the model was about as unsure as if it were picking uniformly between 10 options". It is the same number your tooling reports, and a friendlier scale to eyeball.

### Teacher forcing

We then move to the next token — but we do not let the model's own pick from the previous step influence anything. We ask it to predict token *n+1* at every position *n*, always assuming the previous tokens are the ground-truth ones from our sequence. This is called **teacher forcing**. The model cannot cheat by peeking ahead, because a decoder-only LLM has a causal mask baked into it: each position sees only itself and its predecessors.

The main reason for doing it this way is that it makes the whole thing parallel. All of this happens in a *single forward pass*. We do not run the model once per token; every position gets its prediction simultaneously, because every position's input is already known up front. That is what makes training feasible on today's compute. There is a secondary benefit too: if we fed the model its own picks, early-training mistakes would compound and drag the rest of the sequence somewhere unrelated to what we are trying to teach.

Teacher forcing does come at a cost worth naming, because it explains a lot about what comes after SFT. The model is only ever trained on perfect ground-truth prefixes. It never sees its own mistakes during training and never learns to recover from them — but at inference it is conditioning on its own output, mistakes included. That gap is called **exposure bias**, and it is a big part of why RL exists as a follow-up stage: RL trains on sequences the model actually produced.

### Masking

Not every position counts. We want to teach the model the answer, not to re-teach it the user's question, so the losses on prompt tokens are masked out and thrown away. In code you will see them labelled `-100`, which is the value PyTorch's cross-entropy ignores by default.

Mechanically, that mask is most of what separates SFT from simply continuing to pretrain on the concatenated text. The rest of the difference is the chat template, the data you chose, and much gentler training settings. It is also a choice rather than a law — training on the full sequence, prompt included, is defensible, and libraries expose a flag for it.

One thing that absolutely must stay unmasked is `<eos>`. It is a real token, it is the last one in our target, and it is trained exactly like the rest: the model learns to predict "stop here" from the tokens before it. Get this wrong and you get the classic first-fine-tune failure — a model that answers your question correctly and then keeps going forever, inventing a new user turn and answering that too. If your fine-tune rambles, check that the end token is in your labels and not masked away.

### Adding it up

The loss for the whole sequence is the sum of all the surviving tokens' losses. What you divide that sum by is a choice: per token, or per sample. It matters more than it looks.

{% include figures/ledger.html %}

So for each input–output sequence, we have a loss. Now we need to work out how each model weight — and there are billions of them, recently trillions — influenced the output, and therefore this loss. That is genuinely complex differential calculus, but let us take it on trust that it works. Using **gradient backpropagation**, we can evaluate whether any particular weight contributed positively or negatively to the loss. We want to minimise the loss, so that the model has the highest chance of producing the output we gave it, and backprop tells us whether to increase or decrease each weight.

Then the **optimizer** applies that adjustment. And we repeat.

{% include figures/deepdive-log.html %}

## 5. What about reasoning?

One of the big discoveries of 2022 was that you can dramatically improve a model's reasoning if you prompt it to think step by step before answering. That is really the core of what "reasoning models" are: models trained to think step by step so that you do not have to ask.

Training reasoning is a multi-step process, a mixture of SFT and reinforcement learning — more on RL another time. SFT is usually where it starts. You teach the model to wrap its thinking in `<think>...</think>` using samples like this:

{% include figures/reasoning.html %}

## 6. Knobs worth turning

There are a handful of things you can tune, and they interact more than you would like.

- **Loss normalisation** — per token or per sample, from the section above. It decides whether long samples get more say.
- **Learning rate** — how much we adjust the weights each step. Higher can mean faster learning, or oscillation. For SFT it is typically `1e-5` to `2e-5`, roughly an order of magnitude below pretraining, because we are nudging an already-capable model rather than building one.
- **Batch size** — larger batches stabilise training, but reduce the number of updates. On a small dataset you can end up without enough updates to teach the model anything at all.
- **Epochs** — how many passes over the same data. Usually 1 to 3. Beyond that you start memorising.

It is also worth saying how you know it worked. Loss going down on a held-out set is necessary but not sufficient — a model can get better at reproducing your dataset while getting worse at everything else. You have to actually look at the outputs.

That failure mode has a name: **catastrophic forgetting**, sometimes called the alignment tax. Train hard on a narrow dataset and the model gets great at your task while quietly losing capability everywhere else. It was never told to preserve anything, only to lower loss on what you showed it.

The standard mitigation is boring and effective: mix general-purpose data into your narrow dataset, so the training signal keeps pulling in both directions. The other half is evaluation. Keep a small set of prompts that have nothing to do with your task, and check them after every run. If your fine-tune nails your use case but has forgotten how to write an email, you would rather find that out from your own eval than from a user.

## 7. Can you do this yourself?

Yes. The default path is just expensive.

To adjust every parameter, you need to hold three things in memory at once: the weights, the gradients for those weights, and the **optimizer state**.

That last one deserves a sentence, because it is where most of the memory goes. The optimizer is the piece from the end of section 4 — the thing that takes backprop's verdict on each weight and decides how far to actually move it. In practice that optimizer is almost always **Adam**, and Adam does not just apply the raw gradient. For every single weight it keeps a running average of that weight's recent gradients, and a running average of their squared magnitudes. It uses the two together to give each parameter its own effective step size, so weights with small or noisy gradients still move sensibly instead of being drowned out. It works very well. It also means two extra numbers stored per parameter, forever, plus a full-precision master copy of the weights themselves.

Add it all up and the rule of thumb is around 16 bytes per parameter.

{% include figures/memory.html %}

The cheaper way is **LoRA**. You freeze the original weights entirely and train small adapter matrices alongside them, which ends up being well under 1% of the parameters. Nothing frozen needs a gradient or an optimizer slot, so the entire optimizer tower — the biggest block in that bar — simply disappears, and you are left holding little more than the weights themselves.

And honestly, this is not much code:

```python
from datasets import load_dataset
from peft import LoraConfig
from trl import SFTConfig, SFTTrainer

dataset = load_dataset("HuggingFaceH4/ultrachat_200k", split="train_sft[:2000]")

trainer = SFTTrainer(
    model="Qwen/Qwen2.5-1.5B",
    train_dataset=dataset,
    peft_config=LoraConfig(r=16, lora_alpha=32, task_type="CAUSAL_LM"),
    args=SFTConfig(
        output_dir="qwen-sft",
        num_train_epochs=2,
        learning_rate=1e-4,           # LoRA wants ~10x a full fine-tune
        per_device_train_batch_size=4,
        max_length=1024,
        completion_only_loss=True,    # mask the prompt, score only the answer
        bf16=True,
    ),
)

trainer.train()
```

Every concept in this post is somewhere in those twenty lines. `SFTConfig` picks the learning rate and the number of epochs. `completion_only_loss` is the mask. `LoraConfig` is the reason it fits on one GPU. The chat template comes along with the tokenizer, and the shift-by-one, the softmax, the negative log and the sum all happen inside `trainer.train()` without ever asking your permission.

Which is exactly why it is worth knowing what they are.
