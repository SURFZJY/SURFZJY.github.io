---
layout: post
title: "The Session Should Not Be the Product"
date: 2026-11-04 18:00:00
description: A reading of the public dsh-superagent prototype, and what its split between routing and execution suggests for training multimodal agents.
tags:
  - agents
  - multi-agent
  - llm
  - harness
categories:
  - research
---

I have been reading [deepseek-ai/dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype) as a design, not as a product. The README is plain about the status: research code, not an official DeepSeek release, adapted for publication, and not identical to whatever runs internally. Some ideas around it are future work. What is in the tree is already enough to argue with. It is a proactive coding agent for one GitHub repository, with optional Feishu, that opens a work item, does the work in its own session and directory, and answers in the conversation where the request was made.

## What the harness is for

A model with tools can finish a hard task with less step-by-step steering. The harder problem is a whole working day. Work arrives as events, not as a person sitting in one chat, and those events do not share a context just because they share a human. If every event lands in the same session, unrelated tasks collide and the loop blocks. If every event opens a fresh session, nothing remembers the last decision and several sessions answer at once. The prototype's bet is that the session should stop being the thing the user manages.

Its own [philosophy](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/000-philosophy.md) states the objective in one line: take as little human attention as possible, and give as much value as possible. The agent acts only when asked, or when a written rule allows it, and it speaks only when a person must act or can use a result. A connector on a local loop does not get you there. The loop is still synchronous, and the agent stops when the machine sleeps. The chat API glued the product to the context window. That glue is the bottleneck.

## How the design is split

The diagram is mine. It follows the domain model, not a figure from the repository.

<img src="{{ '/assets/img/blog/superagent-harness-diagram.svg' | relative_url }}" alt="Super-agent harness: intake, decide, execute; fast path for subscribed threads, slow path through Eval" width="1600" height="1000" style="display: block; width: 100%; max-width: 100%; height: auto;" />

**Router and workers.** In the [domain model](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/001-domain.md) the main session is the orchestrator. It evaluates triggers, accepts work, and replies. The executor session is the worker: one session and one directory per ticket, outside the domain, reached only through an Assignment. Workers do not own the conversation. Results go back through a Channel on the thread, which is how a pull request or a topic reply stays in the place the person already is. A worker can be handed off or resumed. The ticket keeps its own state instead of following the session, so the session is replaceable and the work is not.

This is not an agent team. A team lead plans inside one task. Here the main session decides whether new work starts, continues, or stops, and the executor does the task. Prompting a single chat to "delegate when appropriate" rebuilds that split by instruction. The prototype makes it structural: subscription fan-out does not go through Eval at all.

**Events, then a judgment.** A Source turns GitHub or Feishu into Triggers. A Trigger belongs to one or more Threads. Eval runs on a batch, one thread at a time, and triggers that arrive while it runs are merged into the next batch. The first layer is a gate: who may originate work, in that source's own identities, and with what strength. The second layer is the model's assessment against a versioned Policy. The outputs are small: accept, stop, ignore, or unavailable. An accept creates a Ticket and an immutable Mandate. A wider scope is a new ticket, not a silent expansion.

Once a ticket is subscribed to a thread, later triggers fan out to it with no second evaluation. A person's message on that thread can resume an ended ticket under the same mandate. Bot and CI noise does not. That is the fast path. The main session stays on the slow path: new work, and threads nobody has claimed. Subscriptions are many-to-many, held by the thread. Two tickets can hear the same pull request without talking to each other. Correlation between threads adds context and is explicitly not allowed to route.

**The window as a budget.** The prototype does not spell out a cache calculus in those words. What it does say is which facts are allowed into an executor. A subscription delivers triggers as context. A thread the executor follows for background does not grant authority and does not resume the ticket. Policy changes that bear on a live ticket are delivered; they do not revoke the mandate. I read that as the same discipline: put the next fact in the window, leave the rest as an id the model can pull. Wasting the window still wastes the model. The design just enforces the boundary in types instead of in a prompt.

**Asynchrony and where the work runs.** [Execution](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/003-execution.md) moves a worker's foreground shell to a background job after five seconds, so the session can take a new message instead of blocking on a long command. The router preset keeps a smaller tool set. The heavy environment is the executor's directory and the variables that sources register, not a copy of the user's laptop rebuilt in a sandbox. The panel shows what is waiting on a person and stays empty when nothing is. That is the whole persistence story the prototype actually commits to: work items outlive a single shell call, and the person is interrupted only for a decision or a result.

## What is actually new

Fast paths, batched evaluation, and a worker that does not own the chat are old systems ideas. What is worth keeping is the composition. The user is not asked to manage sessions. The orchestrator is not the one writing the patch. A subscription, not another model call, carries the rest of a thread.

The costs are in that split, even without a benchmark. More sessions mean more cached context and a stabler schedule. Fan-out saves the orchestrator from re-deciding events it has already assigned; it does not, by itself, make the patch better. Batching helps only while one evaluation can still read a pile of triggers. The domain model already says a thread runs one Eval at a time and folds arrivals into the next batch. Past some arrival rate, that batch is a long input the model has to get right. Splitting the orchestrator to buy throughput would spend the global view the design exists to protect. The repository does not do that split.

The failures follow from the same choices. Two tickets hear a conflict only if it shows up as a trigger on a thread they share. Speculative work saves attention only while the agent stays silent, and the philosophy doc treats noise as a bug: if the agent is noisy, shorten the prompt or pause it, do not add messages to explain the noise. Authorization is a mandate, not a vibe. The model does not get to declare that a person told it to stop; only Eval decides how strong an ending is. A credential that reaches the context can still leave in a normal tool call. The prototype does not pretend to have solved that. It narrows the blast radius: one repository, an allow-list per source, proactive work recorded before it is allowed to act.

## What I would change in training

These are opinions from foundation-model work on video understanding. They are not a claim about what DeepSeek is training.

The harness is a policy, and most agent traces do not contain it. A ReAct log teaches a model to continue. This design also needs it to refuse, dispatch, and stay quiet. If post-training data is only successful tool traces, we keep building the agent that does the work in the main session because nobody structured the alternative. I would keep the negative decisions: triggers the gate dropped, evaluations that returned ignore, reports not worth showing. Those are the product.

Mid-training is where that selectivity either sticks or fights the model. A router batch is a long input and a short decision, so context length still matters, but the skill is knowing what the mandate does not include. A subscription that adds context without adding authority is a useful training target: the model has to read the extra thread and not treat it as permission.

For tool-use reinforcement learning I would not score only task success. The prototype's objective is attention. A fix that also pings a channel three times is a worse policy than a slower run that posts one pull request. The people who pay are outside the rollout, which makes the reward awkward. It is still the objective the system is built around.

The prototype's triggers are GitHub and Feishu events. A video or a meeting is not a message with an allow-list. I do not think that retires the split between Eval and the executor. It changes the gate. Code can still drop events by source, but "is this frame worth an expensive model" is a perception problem, so the fast path has to be a cheap model, or a cheap head of the same omni model. I would let the harness speculate on text and on short clips tied to an explicit thread, and require a stronger signal before a full visual pass.

Workers that only share a thread will not be enough once one of them is watching video and another is editing a document. I would not add a chat channel between them by default. I would pass a typed artifact — a clip id, a caption, a mandate — that the orchestrator can inline, so they share a reference instead of a conversation. That is a preference. It is the same boundary the domain model already draws between context and authority, applied to modalities this repository does not implement.

## What is still open

The prototype leaves the scaling questions open by being small on purpose: one repository, sources that do not map identities to each other, no long-term memory beyond tickets and threads. From training, "stay quiet" has no natural label. A missed request and a lucky silence look the same unless someone marks them. A video stream also has no obvious Thread. Define threads badly and the fast path either floods the workers or hides the event that mattered.

The point I want to keep is narrow. The session is an implementation detail, and a harness that leaves delegation to the model's mood will not get it. Most of the machinery is classical. What should change training is the policy around attention: what the model is allowed to see, when it must be silent, and which of those decisions are worth learning. This repository is a place to read that policy. It is not, on its own account, a finished agent.

## References

[deepseek-ai/dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype)
