---
layout: post
title: "LocalCowork: a private AI agent that runs entirely on your machine"
subtitle: "An experiment in how far a small, local model can be pushed on real tasks"
tags: [ai agents, ollama, local llm, react, privacy]
author: Kalyan Reddy Katla
---

I built LocalCowork to answer a question that kept nagging me: how far can a small,
local model actually be pushed? Not a giant hosted model, but something like a 7B model
running on my own laptop. How much real work can you squeeze out of one, and what level
of tasks can it reliably handle? LocalCowork is that experiment.

It is a privacy first AI assistant inspired by Claude's Cowork, with one rule: nothing
leaves your machine. It runs on a local Ollama model and uses a ReAct loop (observe,
think, act) to get real work done with the tools you already have. It can run shell
commands, execute Python in a sandbox, search the web, and build Excel, PowerPoint, and
Word documents with formulas and charts. You can even steer it mid task, just type a
correction while it is working and it adapts on the next iteration.

![The LocalCowork ReAct loop: think, act, observe, repeat](/assets/img/posts/localcowork-react.svg)

### What it can do

Point it at a task and it works out the steps. A few things it has handled for me:

- "organize my downloads by file type" (it runs the shell commands itself)
- "analyze sales.csv and show top customers" (it writes and runs Python)
- "search for asyncio best practices and summarize" (web search plus fetch)

Getting started is two commands:

```bash
ollama pull mistral
localcowork
```

### Challenges I faced

1. Local models are not GPT. Getting a 7B model to follow a strict reason and act format
   reliably took a lot of prompt engineering and guardrails, especially for tool calling
   and knowing when a task is actually done.
2. Executing arbitrary shell and Python safely is the scary part. I added path traversal
   protection, sensitive path blocking, and a Docker sandbox with no network, a non root
   user, dropped capabilities, a read only filesystem, and tight resource limits.
3. Mid task steering meant the agent had to read user input while a long task was
   running, which is a concurrency problem, not a prompt problem.
4. Per session rate limiting and timeouts were needed so a runaway loop could not hog the
   machine.

### What I learned

The short answer to my original question: small models can do a lot more than I expected,
as long as you do the work around them. Constrain the problem, give them sharp tools, and
keep each step simple, and a 7B model handles multi step tasks that feel out of reach in a
single prompt. The model is the easy part. The scaffolding (clear ReAct prompts, safe
tools, steering, and good error recovery) is what turns a small model into something
useful. I also learned that for a local agent, security and UX matter more than raw model
quality: the model is replaceable, but a careless delete is not.

Code: [github.com/Kalyankr/localCowork](https://github.com/Kalyankr/localCowork)
