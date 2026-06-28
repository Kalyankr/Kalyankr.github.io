---
layout: post
title: "Winning a bronze medal at the AI Mathematical Olympiad (AIMO Progress Prize 3)"
subtitle: "Teaching an open-source model to solve olympiad math under a no-internet, 5-hour GPU budget"
tags: [kaggle, llm, math reasoning, competition, vllm]
author: Kalyan Reddy Katla
---

I won a bronze medal in the AI Mathematical Olympiad Progress Prize 3 (AIMO3) on
Kaggle, and it was one of the most demanding competitions I have entered.

### The challenge

AIMO3 is part of a $2.2M prize fund aimed at closing the gap between open and closed
models on hard math. The hidden test set is 110 entirely original problems spanning
algebra, combinatorics, geometry, and number theory, ranging from national olympiad
level up to full IMO difficulty. Every problem is brand new, so there is zero risk of a
model having memorized the answer. Each answer is a single integer between 0 and 99999.

What makes it brutal is the constraints. You submit a Kaggle notebook with internet
disabled, only open-weight models allowed, and a GPU budget of about 5 hours for the
whole test set. So you are not calling a hosted model over an API. You run a model you
can actually fit and afford, and you have to be both fast and reliable. Scoring is
unforgiving too: each problem is run twice, and you only get full credit if both runs
are correct.

### The hardest part: compute and time

For me, the two real challenges were compute limitation and the time limit, and they
are deeply linked. The whole test set has to be solved inside roughly a 5 hour GPU
window, on hardware I had to fit my model into. Strong reasoning models like to generate
thousands of tokens per problem, so every extra token of thinking on one problem is time
taken away from another. Most of my effort went into spending that fixed budget wisely:
which model to run, how many samples to draw, and when to stop.

### Techniques I used and how they helped

- Open-weight reasoning model with vLLM. Serving a math focused open model through vLLM
  gave fast, batched generation, which is what made it possible to run many problems and
  many samples inside the time budget.
- Self consistency (majority voting). Instead of trusting a single answer, I sampled
  several solutions per problem and took the most common final integer. This was the
  biggest single reliability gain, and it directly fits the two run scoring, where both
  attempts must agree.
- Tool integrated reasoning (code execution). Letting the model write and run Python for
  the heavy arithmetic and search steps removed a whole class of errors where the
  reasoning was right but the mental calculation was wrong.
- Token and sampling budget control. Capping generation length and adapting the number
  of samples to the time remaining stopped a few hard problems from eating the entire
  budget.
- Robust answer parsing. Extracting the final integer from long LaTeX, applying any
  required modulus, and clamping to the 0 to 99999 range recovered points that were
  otherwise lost to formatting alone.

![My AIMO inference pipeline: sample, check, vote, parse](/assets/img/posts/aimo-pipeline.svg)

### How it could be done better

- A stronger or fine tuned base model. Fine tuning on high quality olympiad data is
  where the top teams pull ahead, and it is the clearest next step.
- Smarter compute allocation. Spend more samples on the problems the model is unsure
  about and fewer on the ones it solves confidently, instead of a fixed budget per
  problem.
- A real verifier. A separate checker or reward model that scores candidate solutions
  would beat plain majority voting.
- Faster inference. Better quantization and speculative decoding would buy more attempts
  inside the same time window.

### What I learned

This competition rewarded engineering discipline as much as math. With a fixed compute
and time budget, the win comes from spending it wisely: a good open model, voting for
stability, code execution for arithmetic, and ruthless answer validation. I also saw how
much of the gap to closed models is closed by careful inference, not just bigger weights.

Competition: [AIMO Progress Prize 3](https://www.kaggle.com/competitions/ai-mathematical-olympiad-progress-prize-3)
