---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Before starting, estimate the maximum number of questions needed. Never exceed that limit.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask *now* without guessing at answers you haven't heard yet. Infer obvious answers from context and treat those decisions as settled. Put only non-obvious decisions on the frontier.

Ask the whole frontier in one round unless doing so would exceed the question limit: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```text
**Q1 - <question title>**: <question body, might be multiple paragraphs, including multiple choices>

Recommended answer: <your recommended answer>

---

**Q2 - <question title>**: <question body, might be multiple paragraphs, including multiple choices>

Recommended answer: <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a *later* round, not this one.

Finding *facts* is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The *decisions* are the user's, but obvious ones can be inferred from context. Put only non-obvious decisions to them and wait.

The session is done when the frontier is empty or the question limit is reached. At the limit, stop grilling, assess whether all high-risk decisions were covered, and list any unresolved ones in the summary. Otherwise, every branch of the design tree should be visited with nothing non-obvious left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
