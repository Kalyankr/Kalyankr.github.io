---
layout: post
title: "Sintra: an autonomous agent that shrinks LLMs for the edge"
subtitle: "Building an agentic framework that plans, experiments, and learns how to compress models"
tags: [ai agents, langgraph, llm, quantization, edge ai]
author: Kalyan Reddy Katla
---

Sintra (Synthetic Intelligence for Targeted Runtime Architectures) started from a
simple frustration. Running a 70B parameter model on an 8GB device is physically
impossible, and when I compressed models by hand, aggressive pruning and quantization
often left them "lobotomized", fast but unable to reason.

So instead of hand tuning every model, I built an autonomous agent that treats
compression as a search problem. Sintra plans a strategy from your hardware
constraints, researches the model architecture, experiments with recipes
(quantization, pruning, and layer dropping), reflects on failures, and iterates until
it hits the accuracy and speed targets.

![Sintra optimizing a model from the command line](/assets/img/posts/sintra-demo.gif)

### How it works

Under the hood it is a LangGraph workflow. A planner sets the strategy, three domain
experts (quantization, pruning, integration) propose recipes, and a ReAct "architect"
uses six tools to introspect the model and look up benchmarks. A benchmarker runs the
recipe, a critic decides whether to stop, and a reflector learns from each failure
before looping again. Everything is logged to SQLite so the agent improves across runs.

It supports multiple backends (GGUF via llama.cpp for CPU, BitsAndBytes for GPU, ONNX
for cross platform) and compares every optimized model against the original to report
accuracy retention.

Running it takes a single command. Sintra auto-detects your hardware and sets targets:

```bash
uv run sintra --model-id microsoft/phi-2
```

### Results

Sintra drives three compression backends behind one interface, and always measures what
the compression cost you:

| Backend | Best for | Precision |
| --- | --- | --- |
| GGUF (llama.cpp) | CPU inference | 2 to 8 bit |
| BitsAndBytes | GPU inference | NF4, INT8 |
| ONNX | Cross platform | INT8 |

After every run it benchmarks the optimized model against the original. A run might
report:

```
Accuracy Comparison:
  Original:  85.0%
  Optimized: 81.2%
  Retention: 95.5%
```

### Challenges I faced

1. Keeping the agent honest. Early versions were confidently wrong about how much a
   recipe would help. Adding a reflection step and an adaptive calibration layer (where
   past experiments correct future estimates) was the difference between a demo and
   something useful.
2. Long runs needed to survive interruptions, so checkpointing and resume became first
   class features.
3. Integrating three very different compression backends behind one interface took more
   glue code than the agent logic itself.
4. Type safety and tests. The project grew to 404 tests with mypy and CI, because an
   agent that calls real tools breaks in subtle ways.

### What I learned

Agentic systems live or die by their feedback loop. The reasoning is easy to show off,
but the real value came from persistence and self correction: an agent that remembers
its mistakes and recalibrates beats a smarter one that forgets. I also learned to treat
model compression as an optimization search rather than a fixed pipeline.

Code: [github.com/Kalyankr/sintra](https://github.com/Kalyankr/sintra)
