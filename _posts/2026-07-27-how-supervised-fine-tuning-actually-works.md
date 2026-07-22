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
I spent my garden leave between companies diving deep into how transformer models work — how the architectures differ and how they are actually trained. The best way I know to check whether you understand something is to try to explain it, so this is me explaining supervised fine-tuning to myself, in public. And I already started writing another post about Reinforcement Learning.

Everything I learned comes from two courses, and I would recommend both without reservation: HuggingFace's [LLM Course](https://huggingface.co/learn/llm-course), which is free, genuinely excellent, and the best explanation of transformers and tokenizers I have found anywhere; and DeepLearning.AI's [Fine-tuning & RL for LLMs: Intro to Post-training](https://www.deeplearning.ai/courses/fine-tuning-and-reinforcement-learning-for-llms-intro-to-post-training), which expanded specifically on RL and more math and theory behind it. If any part of this post leaves you wanting more depth or try these in practice, those are where to go.

It is written for people with a solid technical background / math understanding but no particular machine learning one. There is some maths, but I have tried to keep it to the parts you can see on the peripherals of the models, rather than actual internals, like how the attention mechanism works, etc.

## 1. What a language model actually does

I am not going to dive into transformer internals. There is a lot of nuance in the architectures, and [HuggingFace's material](https://huggingface.co/learn) covers it far better than I would. (Although I will probably write another post about that at some point!)

The short version is that a language model does one thing: it understands, predicts, and ultimately picks the next token.

Predicting the next token means **producing a score** for every single token in the model's vocabulary - this is called **distribution**. The vocabulary is the fixed list of text chunks a model can read and emit — typically 50,000 to 250,000 of them, and they are fragments rather than whole words, so a real one holds entries like `" the"`, `" Berlin"`, `" fine"`, `"tun"`, `"ing"` and `"!"`.

For computational reasons those scores are not probabilities — they are unbounded floats called **logits**. What the model emits at each position is a **logit vector**: one number per vocabulary entry, so a vector 150,000-odd numbers long. In code you will meet it with a batch and a sequence wrapped around it, as a tensor of shape `[batch, sequence, vocab_size]`, but at any single position it is just that one long vector.

To actually generate text, you need to sample from that, and to sample you first need real probabilities. **Softmax** is the transformation that gets you there: it exponentiates every logit and then divides by the total. After softmax, the whole vocabulary sums to 1.

{% include figures/softmax.html %}

{% include figures/sampling.html %}

What you then do with that distribution is a separate choice. You can take the most probable token every time — **greedy decoding** — or you can genuinely draw from it, which is what temperature and top-p are shaping. Those are the knobs you meet on day one of any inference API.

Taking the most probable token has a name worth knowing, because it turns up in the figures below and in every piece of ML writing you will read: the **argmax**. Literally "the argument that maximises" — not the largest probability itself, but *which* token it belongs to. When a figure marks a row as argmax, it just means "this is the token the model would have picked, if you had asked it to generate here"

Here is the thing to carry forward: **sampling is an inference-time activity**. It plays no part in SFT training. We never need the model to pick anything. We only ever need the distribution — and in a moment you will see why that is enough.

## 2. What supervised fine-tuning is for

The goal of SFT is to teach the model format, conversational ability, and an awareness of who is saying what. Base models do not have this. They do not know that "user" means you and "assistant" means them. Frontier labs fix that by showing the model an enormous number of example conversations, and out the other side comes something that can hold a coherent chat.

{% include figures/base-vs-sft.html %}

The base model has read millions of dialogues; it already knows what a helpful answer looks like. What it lacks is the knowledge that it should be producing one *right now*, rather than continuing your text with three more plausible user questions. SFT helps with exactly this.

Keep this in mind when you get to the dataset sizes below. A thousand samples cannot teach a model to converse from scratch. It can absolutely tell it which of its existing behaviours to bring to the front.

{% include figures/pipeline.html %}

Essentially every model you actually chat with has been through this. It is the standard path, and in practice it gets repeated many times, with different datasets, at different stages of a model's life.

## 3. What an SFT dataset looks like

Conceptually a dataset is a list of input–output pairs. In practice you will almost never see it stored as raw concatenated strings. It is structured messages — a list of `{"role": ..., "content": ...}` objects — and the flattening into a single string happens at tokenization time, via the model's **chat template**.

{% include figures/dataset.html %}

Chat template is the model-specific recipe for which special tokens wrap each role, and every model family has its own.

So where the data comes from? Broadly, two sources: (1) humans writing the answers — expensive, high quality, and that's what initial GPT-3 was trained on; (2) **distillation** - where you generate answers with a stronger model and train your smaller one on them. Most open datasets you will find today are the second kind.

Using the right techniques, even small numbers of samples can have an observable impact. The rule of thumb I learned — though I have not personally gone above a few hundred — looks like this:

{% include figures/ladder.html %}

## 4. What actually happens during training

It is called *supervised* because we are directly teaching the model to increase the probability of specific tokens given specific inputs. "Here is the answer; be more likely to say it."

Let us do this with an example small enough to hold in your head. Imagine a language model with a five-token vocabulary that is quite good at repeating exactly what the user said. We want to teach it something new: answer the user with their tokens in reverse order.

- **Vocabulary:** `<user>`, `<asst>`, `tic`, `tac`, `<eos>` — that last one means end of sequence, and tells generation to stop
- **Input:** `<user> tic tac <asst>`
- **Expected output:** `tac tic <eos>`

First we need to know how the model is doing on this sample. And here is the part that surprises people: **we never let the model generate anything.** We hand it the expected output and ask how likely it was to have produced it. How surprised it was is the loss.

Models predict one token at a time, but our training samples are multiple tokens long. So the loss is calculated for each token separately and then aggregated over the sample.

Because we do not need the model to generate, we can give it the entire concatenated sequence at once:

<div class="fig">
  <div class="fig__frame">
    <p class="fig__eyebrow">What the model is shown during training</p>
    <div class="tok-row">
      <span class="tok tok--prompt">&lt;user&gt;</span>
      <span class="tok tok--prompt">tic</span>
      <span class="tok tok--prompt">tac</span>
      <span class="tok tok--prompt">&lt;asst&gt;</span>
      <span class="tok tok--completion">tac</span>
      <span class="tok tok--completion">tic</span>
      <span class="tok tok--completion">&lt;eos&gt;</span>
    </div>
  </div>
</div>

That is it. The whole thing, prompt and answer together, in one go. We then ask the model what it would have predicted at each position, based on tokens before it (differently said, **conditioned only on the tokens before it**). What comes back, for every position in the sequence, is a distribution over the vocabulary.

**The distribution at position *n* is a prediction of the token at position *n+1*.** So the distribution that ought to be putting its mass on our first output token `tac` does not live at `tac` — it lives one slot to the left, at `<asst>`. Every position predicts its successor.

{% include figures/shift.html %}

On the left you see the logits, the actual output of the model. On the right, those logits after softmax. Note again that plain normalisation would also make things sum to 1; it is the exponentiation that makes the *gaps* meaningful.

The loss for a single token is `-log(probability of the expected token)`. Higher probability, smaller loss. Lower probability, larger loss. That is the whole idea. It is called **cross-entropy loss**, and it is the name you will see in every training codebase.

If you ever want to translate a loss number into something intuitive, exponentiate it: `perplexity = exp(mean loss)`. A loss of 2.3 is a perplexity of about 10, which reads as "the model was about as unsure as if it were picking uniformly between 10 options". It is the same number your tooling reports, and a friendlier scale to eyeball.

### Teacher forcing

We then move to the next token — but we do not let the model's own pick from the previous step influence anything. We ask it to predict token *n+1* at every position *n*, always assuming the previous tokens are the ground-truth ones from our sequence. This is called **teacher forcing**. The model cannot cheat by peeking ahead, because LLM has a causal mask baked into it: each position sees only itself and its predecessors (well, this is true only about decoder-type models, but that's for different post).

The main reason for doing it this way is that it makes the whole thing parallel. All of this happens in a *single forward pass*. We do not run the model once per token; every position gets its prediction simultaneously, because every position's input is already known up front. That is what makes training feasible on today's compute. There is a secondary benefit too: if we fed the model its own picks, early-training mistakes would compound and drag the rest of the sequence somewhere unrelated to what we are trying to teach.

### Masking

Not every position counts. We want to teach the model the answer, not to re-teach it the user's question, so the losses on prompt tokens are masked out and thrown away. In code you will see them labelled `-100`, which is the value PyTorch's cross-entropy ignores by default (assuming that you use HuggingFace Transformers library).

One thing that is worth calling out here is `<eos>`. It is a real token, it is the last one in our target, and it is trained exactly like the rest: the model learns to predict "stop here" from the tokens before it. If you don't teach your models to generate end of sequence tokens, you end up with a model that answers your question correctly and then keeps going forever, inventing a new user turn and answering that too.

### Adding it up

The loss for the whole sequence is the sum of all the surviving tokens' losses.

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

- **Batch size** — how many samples you push through before touching the weights at all. You run the forward pass on all of them, aggregate their losses into a single number, and backpropagate *once*. That one gradient is therefore an average over many examples rather than a reaction to a single one, so an odd or badly written sample cannot yank the weights on its own — that averaging is the whole of why batching stabilises training. The cost is fewer updates per epoch: at batch size 8, a 1,000-sample dataset gives you 125 optimizer steps instead of 1,000. On a small dataset that can leave you without enough updates to teach the model anything at all.
- **Loss normalisation** — which is the question the batch immediately raises: aggregate *how*? You sum the per-token losses, but what you divide that sum by is a choice — the number of tokens in the batch, or the number of samples. Divide per token and a long answer contributes more gradient than a short one, so length quietly becomes weight. Most libraries default to per-token. It only bites if your dataset mixes one-line answers with long ones — but then it decides which of them your model is really learning from.
- **Learning rate** — how much we adjust the weights each step. Higher can mean faster learning, or oscillation. For SFT it is typically `1e-5` to `2e-5`, roughly an order of magnitude below pretraining, because we are nudging an already-capable model rather than building one.
- **Epochs** — how many passes over the same data. Usually 1 to 3. Beyond that you start memorising.

It is also worth saying how you know it worked. Loss going down on a held-out set is necessary but not sufficient — a model can get better at reproducing your dataset while getting worse at everything else. You have to actually look at the outputs and ideally have a set of evals that proves that your model continues to work well in other domains that it previously worked well in.

That failure mode has a name: **catastrophic forgetting**, sometimes called the alignment tax. Train hard on a narrow dataset and the model gets great at your task while quietly losing capability everywhere else. It was never told to preserve anything, only to lower loss on what you showed it.

The standard mitigation is boring and effective: mix general-purpose data into your narrow dataset, so the training signal keeps pulling in both directions. The other half is evaluation. Keep a small set of prompts that have nothing to do with your task, and check them after every run.

## 7. Can you do this yourself?

Yes. The default path is just expensive.

To adjust every parameter, you need to hold three things in memory at once: the weights, the gradients for those weights, and the **optimizer state**.

That last one deserves a sentence, because it is where most of the memory goes. The optimizer is the piece from the end of section 4 — the thing that takes backprop's verdict on each weight and decides how far to actually move it. In practice that optimizer is almost always [**Adam**](https://arxiv.org/abs/1412.6980), and Adam does not just apply the raw gradient. For every single weight it keeps a running average of that weight's recent gradients, and a running average of their squared magnitudes. It uses the two together to give each parameter its own effective step size, so weights with small or noisy gradients still move sensibly instead of being drowned out. It works very well. It also means two extra numbers stored per parameter, forever, plus a full-precision master copy of the weights themselves.

Add it all up and the rule of thumb is around 16 bytes per parameter.

{% include figures/memory.html %}

The cheaper way is [**LoRA**](https://arxiv.org/abs/2106.09685). You freeze the original weights entirely and train small adapter matrices alongside them, which ends up being well under 1% of the parameters. Nothing frozen needs a gradient or an optimizer slot, so the entire optimizer tower — the biggest block in that bar — simply disappears, and you are left holding little more than the weights themselves.

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
