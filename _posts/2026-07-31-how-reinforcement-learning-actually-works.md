---
layout: post
title: "How reinforcement learning actually works"
author: "Kuba"
comments: true
description: A visual, ground-up walkthrough of RL for LLMs — how GRPO turns a group of rollouts into a gradient, why the absolute reward never matters, what reward hacking looks like up close, and how to train a reward model from nothing but human rankings.
related: [
  ["2026-07-27/how-supervised-fine-tuning-actually-works", "How supervised fine-tuning actually works", "A visual, ground-up walkthrough of supervised fine-tuning for LLMs — what the loss actually measures, why the off-by-one trips everyone up, and how much of this you can do on one GPU."],
  ["2026-01-26/new-talent-bar-for-software-engineers", "The new talent bar for software engineers", "Web software engineering is changing rapidly, and the bar for what makes a strong engineer is moving with it."],
  ["2026-07-21/building-a-watering-system-for-my-balcony", "Building a watering system for my balcony", "A weekend electronics project to keep my Berlin balcony's plants alive through the summer heat — an ESP32-controlled drip system, the components, and the wiring."],
]
---
This is the sequel to [my post on supervised fine-tuning]({{ '/2026-07-27/how-supervised-fine-tuning-actually-works' | relative_url }}). If you have not read that one, I encourage you to go give it a try for some basics, but overall this one should be understandable if you have at least some basic instinct on how ML training works.

Same disclaimer as last time: everything I learned comes from HuggingFace's [LLM Course](https://huggingface.co/learn/llm-course) and DeepLearning.AI's [Fine-tuning & RL for LLMs: Intro to Post-training](https://www.deeplearning.ai/courses/fine-tuning-and-reinforcement-learning-for-llms-intro-to-post-training), and the best way I know to check whether I understand something is to try to explain it!

## What RL is, and where it sits

Reinforcement Learning is a type of training you apply to LLMs, primarily to make them more helpful, more aligned to human preferences, and increasingly to make them great at reasoning and problem solving — especially in the specific environments of your own software or systems.

It is different from SFT. In SFT we never let the model generate anything. Instead, we give it an expected answer and iteratively adapt its weights to make that exact answer more likely. In RL we do the opposite: we let the model generate, we score what it produced with a reward, and according to that reward we adapt its weights towards the behaviours that scored well, and away from those that didn't.

{% include figures/sft-vs-rl.html %}

RL is extremely powerful if the data is well prepared. It is the primary way of making models feel personal, helpful, safe and smart. It is also more expensive than SFT, less predictable, and far more experiment-driven. No free lunch.

It helps to see where all of this sits. Training a modern model runs roughly in this order: pretraining, then SFT, then — if you want human preferences — training a reward model, and finally RL. The previous post covered the second box; this one covers the last two.

{% include figures/rl-pipeline.html %}

Reasoning is the easiest category to understand. RL is powerful because you let the model explore plenty of potential solutions (because you ask it to generate tokens), and reinforce those that lead to successful answers. You don't care that much how it arrived at the solution. This also lets LLMs find solution strategies that nobody wrote down in the training data, sometimes ones that surprise the people running the training.

## Which architecture of RL?

There are plenty of different algorithms and types of reinforcement learning. We won't dive into them too much, because they share the same fundamental principles and differ in the details. I will focus on a particular subset — **GRPO**, trained against verifier feedback and against rewards learned from human feedback.

As I was learning about the other techniques — [PPO](https://arxiv.org/abs/1707.06347) and [DPO](https://arxiv.org/abs/2305.18290) — I felt that GRPO gives the right foundation, and based on a few signals about how software companies are doing RL for their own needs, it also feels the most common. It comes from the [DeepSeekMath paper](https://arxiv.org/abs/2402.03300) and it is the algorithm behind [DeepSeek-R1](https://arxiv.org/abs/2501.12948).

## What the data looks like

The data for SFT is pairs of inputs and outputs. The data for RL is quite different. To train the model with RL, you need a set of inputs (prompts), and a way to reward the model for what it generates from them. The key, unique insight is that **you don't need to know the solution** — you only need to know what to ask, and how to score whatever comes back.

{% include figures/rl-dataset.html %}

In this type of dataset, notice that while we do have the final answer in the row, we don't have the actual solution. We just know the answer is 5, or 10, or 72. Let's see how to design a reward for a dataset like that.

## Reward with verifiers

The most basic example of reward modelling for mathematical prompts is just checking the final answer of the model and giving it a point — or points. The absolute value doesn't really matter as long as it is consistent across the entire training run.

{% include figures/verifier.html %}

See one challenge with this scoring mechanism - the answer is either correct, or not. It's only binary. A model can reason in the right direction, or perform actions in the right direction, make a mistake at the last step and it's still a zero score. This is called **sparse reward**: it's all-or-nothing.

You can make a sparse reward denser, by handing out partial credit for getting parts of it right. This is called **reward shaping**. These can be more narrow checks like following the right format you expect or calling tools that you would expect to be called.

During the exercises in the HuggingFace and DeepLearning.AI courses you also experiment with rewarding content indications that the model is going in the right direction. Say 0.1 points if the answer uses reasoning words like "therefore" or "hence", 0.1 points if it contains actual mathematical operations. There is an inherent risk to that, called **reward hacking**, that we will dive into later in the post — and you can already see it in rollout R4 above. But it's a good example to visualise the technique, and it did give positive results during the course exercises.

Before we go further, let's align on some nomenclature that will come up from now on.

{% include figures/anatomy.html %}

## Training the model with RL

RL from human preferences is something that end-user-facing labs will do. My bet is that most of the RL training inside software companies will happen around verifiers instead, because there is far more value in agents performing actions and reasoning reliably in your environment than in adjusting their helpfulness or character. But it's still interesting to know how the frontier and open source models get to where they are.

We look into that later, after walking through the training loop, because the loop is exactly the same for rewards from verifiers and rewards from human feedback. Since we just talked through how to design rewards from verifiers, let's dive into the training.

### What you need for RL training

**(1) A reward signal.** That's either a verifier, which we just covered, or a Reward Model trained on human preferences, which we'll build later in this post.

**(2) A training dataset.** You need input prompts that represent the behaviour you want to teach your model. If it's about reasoning, these prompts need to be reasoning tasks — maths or coding — **with answers that you can verify**. If it's agentic behaviour navigating your product's tools, the prompts need to represent your users' needs and expect those tools to be used — and here too you don't need the solution, you need **an end state you can check**: the right record got created, the right tool got called with the right arguments, the tests pass. If it's helpfulness, the prompts need to represent when you expect your model to be more helpful — **but in this case, because you will use a Reward Model trained from human feedback, these prompts don't come with expected answers at all**.

RL input prompts are the most important thing to get right. You are always trying to teach your model something specific, and you want to make sure the prompts are specific enough.

**(3) A training environment.** If it's just maths reasoning, you only need inference of the model itself. But if you want to teach your model to use tools or interact with a sandbox through the filesystem, you will need to provide a fresh, throwaway environment **for every single rollout**. As you'll see in a moment, GRPO generates 8 rollouts from each prompt, which means 8 independent environments, and none of them can leak state into another. A lot of software engineering is hidden in creating reliable environments.

### The training loop

The training loop is basically about going through your input prompts, and each time giving the model a fresh environment to operate in.

**(1) Take a prompt from your training dataset and ask the model to generate N rollouts.** This might be a one-shot output, or multi-step tool use navigating your environment with seeded tools, files, or data.

As I said earlier, I am writing about GRPO. The core concept is that you generate N rollouts from the *same* prompt. Typically that's somewhere between 4 and 16 — it's 8 by default in [TRL](https://huggingface.co/docs/trl/main/en/grpo_trainer), and 8 is what this post assumes throughout. I will explain in a second why it matters so much, but it already highlights that RL training is computationally expensive.

**(2) Apply the reward to each rollout.** If you are using a verifier, parse the output and score it — checking the final answer to a maths problem, running the code to see whether the tests pass, counting which tools have been called, or checking that the output has the structure you asked for. If you are using a Reward Model, run the output through it.

**(3) Average the rewards across the group.** This is your **baseline**, and it describes what this model roughly scores on this prompt, given the shape of its weights right now.

**(4) For each rollout, subtract the baseline from its reward.** This number is the **advantage**, and it answers a very specific question: *compared to how this model usually does on this prompt, was this particular attempt better or worse?* Positive advantage means this rollout was above the model's own average. Negative means below it. We also divide by the standard deviation of the group, so that a prompt where all 8 rollouts scored between 0.42 and 0.48 produces advantages on a similar scale to a prompt where they scored between 0 and 1.

```
advantage = (reward - group_mean) / group_std
```

{% include figures/advantage.html %}

Why bother with a baseline at all? Because without it, we would only ever be pushing the model towards what it just did. If every reward is positive, every rollout gets reinforced — the good ones a bit more, the bad ones a bit less, but everything moves in the same direction. That still points roughly the right way on average, but it is an incredibly noisy way to get there. Subtracting the average is what lets us say "this one, more" and "that one, less" in the same update.

Notice that the absolute values of the reward has now disappeared. The only thing that stays consistently after this operation are gaps (differences) between these rewards. That's why I said earlier that absolute rewards don't matter as long as they're consistent within the run.

This is finally the answer to why we generate N rollouts instead of one. We need something to compare against. PPO trains a whole separate model, a "critic", whose only job is to predict what reward to expect for a given prompt — which means another model to train, tune and keep in memory. GRPO throws that away and does something much simpler: generate a group, use the group's own average as the baseline. That is literally what "Group Relative" in Group Relative Policy Optimization means. You pay for it with 8× the generation, but you don't need another model at all, which means less complexity and, most of all, less memory.

There is also a significant consequence here for what kind of prompts provide best value for buck. If all 8 rollouts get exactly the same reward — all of them solved the problem, or all of them failed — then the group average equals every individual reward, every advantage is zero, and this prompt teaches your model absolutely nothing. You only lost money on compute. That's why a lot of researcher instinct is in what kind of prompts to include in RL training set. Ideally those that your model sometimes gets right, sometimes wrong, and you want to make sure that it gets those right ones more often.

This is also the strong reason to apply reward shaping. Notice that the reward is still one number per finished rollout — shaping doesn't feed the model anything mid-generation, it just stops the score being all-or-nothing. What that buys you is **spread inside the group**. On a hard prompt where all 8 rollouts fail, a binary reward gives you eight zeros and zero gradient. A shaped reward separates them — this one at least got the format right, that one got three steps in — and suddenly the group has something to rank, so the prompt still teaches the model something instead of being thrown away.

**(5) Now calculate the loss.** In SFT we handed the model an expected answer and nudged its weights to make that answer more likely. We never let it generate. Here we do the opposite — we let the model generate, and then we nudge its weights to make what it generated more likely, or less likely, depending on the sign of the advantage for that specific rollout.

So we take each rollout, run it again through the model in a forward pass (exactly like SFT — the tokens are already fixed from the rollout, we just want to know how likely the model was to produce them), and get the log probability of each token it generated. Then we multiply by that rollout's advantage:

```
loss_for_rollout_k = -advantage_k * (sum of log probs of its tokens) / number_of_tokens_k
```

That's it. If the advantage is positive, minimising this loss pushes the probability of those exact tokens up — the model becomes more likely to produce that kind of output next time. If the advantage is negative, the sign flips and the same machinery pushes those tokens down. If the advantage is zero, the whole term is zero and nothing moves.

We divide by the number of tokens so that a long, rambling output doesn't get a bigger weight update just for being long.

This is somewhat simplified form. Real GRPO implementations wrap it in something called **clipping**, which exists because in practice you reuse the same batch of rollouts for a few gradient steps, and the model doing the updating slowly drifts away from the model that generated them. I am skipping this here as this is not that important to build instinct on how RL works.

That formula is for a single rollout, and we generated 8 of them. Each one has its own advantage, and each one produces its own loss. To get the loss for this prompt, we average across the group:

```
         1    N
loss  =  ─ ·  Σ   loss_for_rollout_k
         N   k=1

where N = 8, the number of rollouts in the group
```

The group is not 8 training examples — it is one training example made of 8 attempts.

Which gives us the shortest possible summary of what RL actually is: **it's SFT on the model's own output, where each sample is weighted by how much better or worse it was than the model's average attempt.**

**(6) There's one problem left.** If you just run steps 1–5 over and over, the model will happily optimise for higher rewards while quietly losing everything it learned during SFT or previous RL runs — it drifts towards whatever scores well and away from being a decent model in general. This is the same **catastrophic forgetting** from the SFT post, arriving by a different route.

The standard fix is the **KL penalty**. You keep a frozen copy of the model you started from — the **reference model** — and add a term to the loss that charges the model for moving too far from what the reference would have said. A parameter called **beta** sets the price. Too tight and your model can't learn anything new; too loose and it wanders off somewhere that scores beautifully and reads terribly. The nice property is that it's cheap: no extra generation is involved, just a second forward pass over tokens you already have. Keep in mind that forward pass is different than generating rollout, and generally is pretty cheap compared with generating a rollout. The details of how that distance actually gets measured are more maths than this post needs, so I'm leaving them for you to dig into.

{% include figures/kl-leash.html %}

Worth flagging that this is one of the places where practice has moved: TRL now ships with `beta=0.0` by default, on the grounds that for verifiable rewards over short runs the KL term costs memory and buys little. That is a defensible default for maths and code, and a much less defensible one when your reward is a model trained on human preferences.

Also worth noting that at this point you are holding the model you are training, a frozen reference copy of it, and possibly a reward model, all in memory at once — while also running generation. This is the real reason RL is so much more expensive than SFT.

## Reward hacking — the thing that will actually go wrong

Let's get back to our dense reward example from before, where we gave partial credit for using reasoning language in mathematical prompts. The inherent risk is that the model learns to repeat those reasoning words and collect the reward, instead of actually reasoning. That was rollout R4 in Figure 4, and it is not a contrived example — it is the first thing that happens.

This is **reward hacking**: the model optimises the thing you measured instead of the thing you meant, and it is very good at finding the gap between the two. It's basically Goodhart's law of Machine Learning.

Rewards from verifiers are the safest ground here, because "did the test suite pass" is hard to fake. There are far more variables in rewards from human feedback, and it can lead to models becoming sycophantic (remember GPT-4o?), repetitive, or too constrained.

The KL penalty from step (6) is a partial defence, in that it limits how far the model can go chasing the reward. The real defences are boring: look at the outputs, and evaluate with something other than the thing you trained on. Which brings us to the next section.

## How do you know it worked?

In SFT you can mostly watch the loss go down. In RL, we track the mean reward over time. However, mean reward over time is an indication of the model getting better at getting our rewards, not necessarily of the model getting better for the users. Those are the same thing only if your reward function is perfect, and it isn't.

So the eval setup needs:

- **A held-out set of prompts**, never trained on, scored the same way. It catches ordinary overfitting similarly to verfication set in SFT.
- **An eval that doesn't use your reward function at all.** If you trained against a verifier, evaluate with a benchmark, an LLM judge, or humans reading outputs. This is the one that catches reward hacking.
- **A general capability eval with nothing to do with your task**, to catch the model getting worse at everything else while it gets better at yours.
- **Actually reading the outputs.** Most reward hacks are instantly visible to a human and completely invisible in a metric.

## Reward from human feedback — training a reward model

Let's say your goal is to make the model more helpful, or more enjoyable to use.

{% include figures/helpfulness.html %}

How do we score rollouts like these? There's no answer to regex out, no test suite to run. And if you employed humans to score them live during training, that would be prohibitively expensive, slow and unreliable — a human would definitely have a different bar in the first 30 minutes of work than after 6 hours of reading AI outputs (um, what was score 4 again? what was 5?). All of this because, contrary to SFT, the reward has to be applied *during* training, on output the model has only just generated.

It turns out that LLMs make great reward models, and one LLM can read the generated answer from another and give it a score. The bet we're making is that judging which of two answers is better is an easier task than writing the better one — so we train that judge first, and then use it to RL the model we actually care about. The previous section on reward hacking is largely about the ways this bet goes wrong.

Training a reward model is still based on actual human preferences, but this time those preferences can be collected asynchronously, ahead of time, which makes it far more tractable than trying to put a human in the training loop. This is the shape [Christiano et al. proposed in 2017](https://arxiv.org/abs/1706.03741) and [InstructGPT](https://arxiv.org/abs/2203.02155) turned into the standard recipe.

### Data preparation for a reward model

To train a reward model, you need:

**(1) An LLM** — often your SFT checkpoint, the same model you're about to RL — with a scalar **reward head** instead of the usual language modelling head, so that instead of generating the next token, it returns a single number. However, I would expect that many companies figure out how to use smaller base models for reward models (like InstructGPT later mentioned).

What does that really mean? Again, I recommend [the SFT post]({{ '/2026-07-27/how-supervised-fine-tuning-actually-works' | relative_url }}) for some basics. I explained there that an LLM produces, for each token position, a vector of logits that can be softmaxed into a probability distribution over the entire vocabulary. So when we ask the model to generate text, it does that, we sample a token from that distribution, and repeat.

If you go one step earlier, it turns out these models are performing multiple layered matrix operations to produce what's called a **hidden state** for each token, which is later transformed by the **language modelling head** into the logits I mentioned. The reward head does the exact same job with a different output size: it takes that same hidden state and turns it into one number instead of one per token in the vocabulary. Same body, different head. Everything the model already learned about reading text stays exactly where it was, and we only swap the thing sitting on top of it.

**(2) A set of training examples.** This is a different training dataset than the one you will use for RL. These consist of an input prompt, a set of different outputs, and a human-picked ranking of those outputs from best to worst. In the early days of ChatGPT you would have seen it generate two outputs and ask which one you preferred. Or you may have used something like LMArena, which shows you outputs from different models and asks you to pick a winner. All of this data can be used to train a reward model.

Before we look at the training loop, let's talk about preparing the dataset. Say we have an input prompt and 4 outputs ranked by humans. From this data, we create all possible pairs — with 4 outputs, that's 6 pairs. Each pair is just "this one was preferred, this one was not".

{% include figures/pairs.html %}

Notice that we did not assign any score to any output. Nobody ever said that the best answer is worth 9 points and the worst one is worth 2. Our humans only gave us an ordering, so all we can extract from it is "A beats B".

The question we are teaching the model to answer is: *given these two answers to the same prompt, which one did the human prefer?* That is a yes/no question — first or second — and that's what makes this a classification problem. In essence, we will be training our base LLM to become a classifier of human preferences.

### Training the reward model

The actual loop is fairly simple and similar to SFT, with one important twist. In SFT we always had a ground truth to compare against — the expected token. Here we don't. There is no "correct reward" anywhere in our data, only the information that one output beat another. So the loop is built around pairs:

**(1) Take one pair** from your dataset — the preferred output and the dispreferred one, both answering the same prompt.

**(2) Run both of them through the model** in a forward pass. The model reads the whole prompt-plus-answer sequence, and the reward head takes the hidden state at the very last token — the point where the model has seen everything — and turns it into a single number. Two forward passes, two scalars. Call them `r_preferred` and `r_dispreferred`.

**(3) Subtract them:** `r_preferred - r_dispreferred`. This single number is what the entire training is about. If it's positive, our model agrees with the human. If it's negative, our model disagrees. If it's zero, our model has no opinion at all.

**(4) Now, some maths.** How do we take that scalar and turn it into a loss? We apply a **sigmoid** to it. The sigmoid returns 0.5 at 0, and asymptotically approaches 0 and 1 at −∞ and +∞. So a difference of zero gives 0.5 — a coin flip, the model telling us it has no idea which one the human liked. Anything positive gets closer to 1; anything negative gets closer to 0.

What the sigmoid gives us is exactly this: **the probability that our model would agree with the human on this pair.**

**(5) And now we calculate the loss the same way we did in SFT.** We have a probability, and we want it to be as close to 1 as possible, so we take the negative log of it:

```
loss = -log( sigmoid(r_preferred - r_dispreferred) )
```

If the model agrees strongly with the human, the sigmoid is close to 1, the log is close to 0, and the loss is tiny — nothing to fix. If the model got the pair backwards, the sigmoid is close to 0, the negative log explodes, and the loss is huge. Exactly the same shape of feedback as in SFT, just measured on a comparison instead of on an expected token. This is called the **Bradley-Terry** loss.

{% include figures/sigmoid.html %}

**(6)** We go through backpropagation and gradient calculation, and **(7)** we update the weights of the model to minimise the loss using an optimizer. Both exactly as in SFT.

## How much data do you actually need?

### For the reward model

The published numbers span a wide range, depending on how much you care about the result.

{% include figures/rm-data.html %}

The useful way to read that spread: a reward model for a narrow, well-defined preference is a much smaller project than a reward model for general helpfulness. If you only need "does this answer follow our support tone", you are closer to the low end than the high end.

InstructGPT's reward model was **6B parameters against a 175B policy (policy means the actual LLM being trained)**. And Llama 2 collected its annotations in weekly batches and retrained its reward models as the data arrived, across five successive RLHF rounds — because each new policy shifted the distribution of outputs away from what the previous reward model had been trained on. Reward models go stale, and they go stale because your training is working.

### For the RL run itself

The number that matters isn't prompts, it's total generations, because that's what you're paying for.

{% include figures/rl-cost.html %}

Two open reference points to anchor against. AI2's [Tülu 3](https://arxiv.org/abs/2411.15124) — the most completely documented open post-training recipe there is — runs its RL-with-verifiable-rewards stage on [29,946 prompts](https://huggingface.co/datasets/allenai/RLVR-GSM-MATH-IF-Mixed-Constraints): 7,473 from GSM8K, 7,500 from MATH, and 14,973 instruction-following prompts with programmatically checkable constraints bolted on. At the other end, [DeepSeekMath](https://arxiv.org/abs/2402.03300), the paper GRPO comes from, ran GRPO over roughly 144,000 chain-of-thought maths questions, sampling 64 outputs per question.

That is a factor of five in prompts and a factor of forty in total generations between two runs that are both, in the scheme of things, narrow and verifiable. Which is the honest summary of this section: there is no rule of thumb here comparable to the SFT ladder. Thirty thousand well-chosen, correctly-difficult prompts beat three hundred thousand that the model either always gets right or never does.

And it is worth remembering that none of these stages happen in isolation. DeepSeek-R1's pipeline alternates them — a small cold-start SFT, then RL, then ~600k reasoning samples collected by rejection-sampling from that RL checkpoint, mixed with ~200k non-reasoning samples, SFT on all 800k, and then RL again. The two posts describe two halves of one loop, and in a real recipe you go round it more than once.

## Where to go next

If you want to actually run this, TRL's [`GRPOTrainer`](https://huggingface.co/docs/trl/main/en/grpo_trainer) is about as few lines as [`SFTTrainer`](https://huggingface.co/docs/trl/main/en/sft_trainer) was — you supply a dataset of prompts and a Python function that returns a number, and the group generation, the baseline, the advantages and the loss all happen inside `trainer.train()`.

If you need the reward model half too, that is [`RewardTrainer`](https://huggingface.co/docs/trl/main/en/reward_trainer). You hand it a preference dataset of `chosen`/`rejected` pairs, it swaps in the scalar reward head for you, and it applies exactly the Bradley-Terry loss from the section above.

Which, exactly as with SFT, is why it is worth knowing what they are. The code will not tell you that a group of eight identical rewards taught your model nothing, that your reward function is being gamed, or that your prompts are the wrong difficulty. Those are the failures that actually happen, and every one of them is invisible in the loss curve.
