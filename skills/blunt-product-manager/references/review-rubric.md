# Review Rubric

Use this rubric when the user provides a PRD, feature idea, MVP scope, roadmap item, or launch plan that needs a fuller teardown.

## Fast Triage

Answer these in order:

1. What user pain is being solved?
2. Who feels that pain often enough to care?
3. Why is the current workaround not good enough?
4. What is the smallest version that proves demand?
5. What metric would change if this worked?

If more than two answers are weak or missing, say so plainly. The proposal is probably premature.

## Chinese PM Shortcut

When the user wants a faster, harsher Chinese teardown, answer these four questions first:

1. 这到底在解决谁的什么痛点？
2. 不做会怎样，做了又具体会改变什么？
3. 为什么要现在做，而不是更小、更便宜地验证？
4. 这件事成功以后，哪个指标会真的动？

If any answer is missing, call it out early instead of pretending the proposal is mature.

## Product Teardown Dimensions

### Problem

- Is the problem concrete, frequent, and expensive?
- Is it a user problem or just an internal wish?
- Does the proposal confuse feature demand with real need?

### User

- Is the target user narrow enough to make decisions?
- Who is excluded on purpose?
- Who feels the pain most acutely?

### Value

- What outcome improves for the user?
- Can the user describe the benefit in one breath?
- Is the promise strong enough to change behavior?

### Differentiation

- What is meaningfully better than the status quo?
- Why would a user switch now?
- Is the idea just a weaker copy of existing products?

### Scope

- What can be removed without killing the value?
- What is MVP versus vanity garnish?
- Which parts create support or operational drag?

### Metrics

- What leading signal appears quickly?
- What guardrail metric prevents local optimization?
- What metric would prove this was a bad idea?

### Delivery Risk

- What data, engineering, content, legal, or ops dependency is hidden?
- What assumption breaks the plan if false?
- What must be manually handled before automation works?

### Priority

- Why is this more important than the next best alternative?
- What are we not building because this exists?
- Is timing driven by users, strategy, or internal excitement?

## Common China-Style Scenarios

### Boss One-Liner Requirement

When the request sounds like "做个会员体系", "加个 AI", or "我们也要做社区":

- Translate the order into a product hypothesis before evaluating it.
- Separate leader preference from validated user demand.
- Ask what business goal is actually being chased.
- Call out "industry trend following" when the idea lacks user pull.

### PRD Review

Check whether the doc is missing:

- target user and scenario
- trigger and entry point
- baseline behavior or current workaround
- scope boundaries
- success and guardrail metrics
- exception handling
- launch dependencies

If the PRD is long but still misses these, say the document is detailed but not decision-grade.

### MVP Review

Check whether the MVP is secretly a phase-3 vision deck:

- too many personas
- too many entry points
- too many features for first launch
- no explicit learning goal
- no success threshold

Use a blunt line if needed: "This is not MVP. This is scope inflation wearing a startup hoodie."

### Roadmap Review

Check whether roadmap sequencing is fake:

- upstream dependencies unresolved
- capacity assumed rather than planned
- strategic theme too broad
- no reason why Q1 item is above Q2 item
- no kill criteria for weak bets

If roadmap logic is weak, say it is a backlog pile, not a roadmap.

## Verdict Labels

Use one of these labels to make the punchline crisp:

- Strong signal, weak plan
- Real problem, bloated solution
- Nice feature, not a product
- Good intent, zero focus
- Metrics cosplay
- Strategy by vibes
- Expensive distraction
- 老板一句话，团队一个月
- 文档很满，价值很空
- 需求挺热闹，闭环不存在
- 优先级失踪，排期先行

## Rewrite Formula

When fixing the proposal, rewrite it into five lines:

1. For `[specific user]`
2. Who struggles with `[specific problem]`
3. We will ship `[smallest useful intervention]`
4. Success means `[metric and time window]`
5. We will not do `[explicit non-goals]`

For Chinese responses, this can be rendered as:

1. 给 `[明确用户]`
2. 解决 `[具体痛点]`
3. 先做 `[最小可行方案]`
4. 用 `[指标/时间窗口]` 判断是否有效
5. 明确不做 `[非目标/延后项]`
