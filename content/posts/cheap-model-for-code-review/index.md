---
title: Why I use a cheaper model for code review
description: 'Data from more than 2,000 tasks: why a top-tier model for execution
  and a cheaper model for review gives better value, and what that depends on.'
date: '2026-10-29'
draft: false
---

The usual thinking goes like this: use a top-tier model to break down tasks, and a model one tier down can execute them. Review, though, has to be top-tier.

But here is the combination I use most right now:

- Task breakdown: opus-5.5-high
- Execution: opus-5.5-medium
- Review: gpt-6.1-sol-high

Most people's first reaction: "You're letting gpt-6.1-sol, a high schooler, grade the homework of opus-5.5, a grad student?

"Give me a break."

But my data from more than 2,000 tasks says otherwise. With the right hard gates, skills, and prompts, here is what works:

A top-tier model breaks down and executes the task, and a model one tier down reviews it. That keeps the task success rate and quality where they need to be, and it gives clearly better value.

That is the core point of this post.

> Data source: [Kander, a task orchestration tool](https://github.com/dualface/kander)
>
> A large card is a task card that the model rated as harder or riskier (it spans modules, changes a protocol, and so on). Every other card is a small card.

Below is the actual data Kander recorded (as of October 29, 2026).

**Execution: how often review sends task cards back**

A send-back means one review round found an issue that must be fixed. The higher the large-card share, the harder the overall workload.

| Execution model   | Send-backs per card | Sent back at least once | Must-fix issues per card | Large-card share |
| ----------------- | ------------------: | ----------------------: | -----------------------: | ---------------: |
| opus-5.5          |                0.78 |                     61% |                     1.29 |              88% |
| gpt-5.6-sol       |                0.84 |                     72% |                     1.78 |              94% |
| swe-2             |                1.02 |                     65% |                     1.79 |              73% |
| deepseek-v4-flash |                1.18 |                     74% |                     3.13 |              51% |

**Review: how each review model performs**

| Review model        | Reviews | Reviews with must-fix | Must-fix per review | Must-fix rejected |
| ------------------- | ------: | --------------------: | ------------------: | ----------------: |
| **gpt-6.1-sol**     | **422** |         **223 (53%)** |            **1.09** |          **4.8%** |
| gpt-6-astra         |     231 |             127 (55%) |                1.69 |              2.0% |
| grok-4.7-build-fast |     422 |             189 (45%) |                0.70 |              5.1% |
| grok-4.7            |     345 |             159 (46%) |                0.74 |              2.3% |
| swe-2-max           |     112 |              23 (21%) |                0.31 |              3.8% |

> The table shows that gpt-6-astra is the most likely to find must-fix issues. gpt-6.1-sol comes right behind it, only two percentage points lower.
>
> At OpenAI's official API pricing, gpt-6-astra's unit price is five times that of gpt-6.1-sol. In third-party benchmarks, the same task actually costs about 4.5 times as much.
>
> So once you factor in cost, gpt-6.1-sol has a very clear advantage.
>
> More than 1,900 of the tasks in this data come from [QuickTUI](https://quicktui.ai/), a commercial product.

~

## Why tasks go wrong

Without a review stage, tasks mostly go off track in two places:

- Planning
- Execution

Often, a task may go off track as early as the first step, when the requirement is turned into an execution plan. The two most common causes:

- The requirement leaves details unclear
- The requirement is too complex

~

## How to get a clear goal

The first cause, unclear details, is easy to understand. When we state a requirement, we usually describe it in the abstract. Nobody can think of every detail. If the plan doesn't settle those gray areas, the model is likely to drift during execution, and the result won't match what you expected.

So when you discuss a requirement, use a top-tier model with the right prompt. Have the model raise the questions that come up when you actually try to carry out the plan, and have it confirm each answer with you. The superpowers skill, which was very popular a while back, exists to solve this problem. It makes the model keep asking the user questions until the requirement is completely clear.

Models have improved since then. Much of what those skills did is now built into the models or the software that runs the agents, and we should make the most of it.

For example, when I discuss a requirement in Codex, I always turn on plan mode. In this mode, the model asks a series of questions that help me and the model work out the requirement together.

With fable-5.1 or opus-5.5, I also add one line to the prompt: "Work out the task goal with me, and pin down its details and acceptance criteria."

After a few rounds of back-and-forth, the model and I usually both have a clear picture of the goal and how to reach it. That's when you can start creating the plan.

~

## Break down complex tasks

Sometimes a requirement is too complex. If you do it as one task, the session context grows large and may trigger auto-compaction several times along the way. Context compaction always loses information, and that hurts the quality of the work. An overly complex task is also harder to review, and usually takes more review rounds.

So the best option is to break the complex task into a series of simpler tasks.

The basic principles of breaking down a task:

- Each task has a single goal.
- A "goal" is a result the user can verify and accept on its own.

Here is a real task:

> QuickTUI is a tool for controlling the agents on your computer from your phone. The app has a quick toolbar.
>
> The toolbar has two tools, Quick Answer and D-Pad. Quick Answer shows five buttons for the numbers 1 to 5; picking one sends that number plus Enter to the agent. D-Pad is a 3x3 grid panel with four arrow keys and two or three commonly used keys.
>
> The two features were added at different times, and each has its own implementation. To unify the look and the interaction, refactor them into a base floating panel component, then derive the two components from it.

This task really has three goals:

- A shared floating panel component that provides features such as dragging and locking.
- A Quick Answer component built on the floating panel component.
- A D-Pad component built on the floating panel component.

In the end, the task was split into three subtasks, which made up a task group.

This is the same way people solve problems: **split a complex task into simpler tasks that are easier to finish.**

Whatever model you use, the task success rate and quality depend on this step. This is where a top-tier model is worth the most.

~

## Create a plan you can verify

Plenty of skills can help you create an execution plan. Whatever form the plan takes, you should be able to verify it. Here is a simplified task file template:

```
## Goal

A summary of the task goal.

## Expected results

What the task should achieve:

- Result 1
- Result 2

## Acceptance criteria

[ ] Acceptance item 1
[ ] Acceptance item 2
[ ] Acceptance item 3
```

What makes the whole plan verifiable is the "Acceptance criteria" section.

Each item there should check exactly one thing that can be verified. Once every item in the list passes, the task has a baseline of quality.

Here is the acceptance checklist for one of the tasks from the real requirement above:

```
[ ]  client/protocol/ui-flow-design-tokens.yml  has token  shortcutPickerPanel , dark  rgba(35,41,35,0.4) , light rgba(255,255,255,0.4) .
...
...
[ ]  client/protocol/ui-flow-terminal.yml  describes both pickers with one shared renderer.
[ ]  client/protocol/ui-flow-terminal.yml  specifies the three layouts, empty grid cells, panel background, entrance order, and the rule for the 300 ms total duration.
[ ] On iOS and Android, when the d-pad opens, the keys pop in one after another, and the panel no longer scales as a whole.
[ ] On iOS and Android, when unlocked, tapping a d-pad key shows a visible bounce on that key.
[ ] On iOS and Android, when unlocked, tapping the blank background or an empty grid cell does not close the panel and sends nothing.
```

I've quoted only a few items. The first is a mechanical check; in principle, a script could check it. The rest need someone to read the code, but the code involved is tiny and has no complex logic.

The full list looks long, but **each item has a very clear acceptance standard**. It either passes or it doesn't, with no gray area.

A simple, clear checklist like this is exactly what lets me pick gpt-6.1-sol, one tier below opus-5.5, for review. In practice, the success rate and quality have been entirely acceptable.

~

## Improve the quality of the plan

At this point the task plan is written. But models always hallucinate, and a plan a model writes may well have problems. So ask the model to self-review the plan it just wrote.

Self-review needs the right prompt. Here is a simple example: "Based on the task goal we agreed on, check that the plan matches the goal. Make sure the plan has no errors or gaps, and no unrelated goals. Fix any problems you find."

Self-review of a complex task's plan usually turns up some problems. Hand the plan to another agent for review, and it may dig up more.

In practice, even a single round of self-review is clearly worth it.

~

## Keep execution on track

The usual view is that execution can use a model one tier down. There's a document and acceptance criteria, after all, so the result should land close enough.

In practice, though, the biggest problems with a lower-tier model are drift and omissions:

- The goal was A, and the result came out a little short of A or a little past it (A- or A+)
- The goal had A1, A2, and A3, and only A1 and A3 got done

Review catches these problems later and asks the executing agent to redo the work. But across a large number of tasks, I've seen this significantly increase the number of review rounds and the time each task takes.

The data at the top of this post shows the same thing:

Cards executed by models two tiers down, such as deepseek and swe-2, average more than 1.00 send-backs, well above opus-5.5's 0.78. Cards executed by gpt-5.6-sol, one tier down, average 0.84. That closes much of the gap, but 72% of them were still sent back at least once. The best performer was opus-5.5: only 0.78 send-backs per card, and only 61% of its cards sent back at all.

Each review round isn't just a model checking the work. It also runs unit tests, end-to-end automated tests, and so on, so one round takes at least a few minutes. Run a few extra rounds, and a single task takes much longer.

So going by the data, the better choice is a top-tier model for execution. It lowers the chance of drift and omissions and cuts the number of execute-and-review rounds.

~

## How to review

I said to use a top-tier model for execution. But even the best current models still drift and miss things, so you can't skip the review stage.

So how do you review?

One approach is to ask the agent to review the task in the session that executed it. That only works for mechanical checks, not for review that needs judgment. For details, see this doc: [Give Claude a check to run](https://code.claude.com/docs/en/best-practices#give-claude-a-check-to-run).

Any review that needs judgment must run in a new session. Whether that session is a separate agent or a subagent doesn't matter.

**In the new session, give it only the goal and acceptance parts of the task document.** Then tell it which code to review (usually a git commit). **Never send the new session anything about how the task was executed. Otherwise its context gets polluted all the same.**

Back to the key point from earlier: clear acceptance criteria are the precondition for using a lower-tier model for review. Without an acceptance checklist, having gpt-6.1-sol review opus-5.5's work can go like this:

- It disagrees with opus-5.5's output, even when the output was correct
- It misses parts of the task that execution skipped, and details that drifted

A lower-tier model can review a top-tier model's output. It needs hard criteria to do it.

A top-tier reviewer also has a problem beyond cost: it is more likely to chase edge cases. (Looking at you, gpt-6-astra.) So as long as hard constraints keep the delivered quality acceptable, a lower-tier reviewer gives you a clear cost advantage and also speeds up delivery.

~

## The review loop

The review comes back. What next?

Send the result back to the executing agent, and have it do the following:

- Confirm that each reported issue really exists and should be handled
- Fix the confirmed issues

One step here is key: **confirm that each reported issue really exists and should be handled**.

A lower-tier model in the execution stage may just accept everything without thinking, or make the wrong call. A top-tier model is less likely to get this step wrong.

In the real data that Kander has collected, 6.2% of review comments were rejected and 15.5% were deferred (every deferred item was a non-blocking suggestion). The executing model must be able to decide what must be fixed and what can be left alone. This is one more reason I think execution should use a top-tier model.

Skip this step, and you get this pattern:

- The reviewer agent finds an issue that doesn't need handling, or doesn't exist at all.
- The executing agent starts fixing it right away.
- After the fix, the next review finds another issue, and the fixing starts again.

This problem is especially visible with gpt-6-astra as the reviewer. The data shows gpt-6-astra reports 1.69 must-fix issues per review, against 1.09 for gpt-6.1-sol. The main reason is that gpt-6-astra finds more extreme edge cases.

Keep in mind that even with opus-5.5, 61% of task cards were sent back at least once. So, once again: you can't skip review.

~

From breakdown to final delivery, a task goes through this process:

```
Break down --> Execute --> Review --> Deliver
                 ^           |
                 |           |
                 +- Issues --+
```

I call this back-and-forth between execution and review the **review loop**.

A task that gets through the review loop has, at minimum, passed every acceptance criterion listed in its task document. That gives it a baseline of quality.

~

## The full orchestration system

A complete orchestration system has many parts:

- The right skills and prompts
- A task template
- A task report template
- Scripts that control the task execution flow

One short post can't cover all of it. If you're interested, take a look at [Kander, a task orchestration tool](https://github.com/dualface/kander).

It already includes enough skills, prompts, and tools to cover these parts, and it enforces strict gates to keep task quality as high as it can.

I'll also write a detailed post on how to use Kander later.

~

Back to the claim at the start of this post: with the right tools, a lower-tier model is the more cost-effective choice for review.

> This post was written in Chinese and translated into English with AI assistance.
