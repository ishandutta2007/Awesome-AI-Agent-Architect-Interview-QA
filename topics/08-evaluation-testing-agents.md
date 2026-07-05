# ✅ Evaluation & Testing of Agentic Systems

[← Back to main README](../README.md)

---

### Q: Why is evaluating an agentic system significantly harder than evaluating a single-call LLM application (e.g., a simple Q&A or summarization prompt)?

**Answer:**
A single-call LLM application has a **relatively bounded, single input-output pair to evaluate**. An agentic system involves **many sequential decisions** (which tool to call, what arguments to use, when to stop, how to recover from errors) where the **final output quality depends on the correctness of the entire chain of intermediate decisions**, not just the final response — a technically fluent, well-formatted final answer could still result from a genuinely flawed intermediate process (e.g., the agent got lucky despite calling the wrong tool at one point, or skipped a step it should have taken) that a purely outcome-focused evaluation would miss entirely. This means agent evaluation needs to assess **both the final outcome and the quality/correctness of the intermediate process/trajectory**, and the combinatorial space of possible execution paths for any non-trivial agentic task is vastly larger than the space of possible outputs for a single-call application, making exhaustive test coverage fundamentally harder to achieve.

---

### Q: What is "trajectory evaluation" in agent testing, and what specific aspects of an agent's execution trace would you evaluate beyond just the final answer's correctness?

**Answer:**
Trajectory evaluation examines the **full sequence of an agent's intermediate steps** (its reasoning, tool calls, and observations) rather than only the final output. Specific aspects worth evaluating: **tool selection correctness** (did the agent choose appropriate tools for each sub-task, not just eventually arrive at a correct answer despite poor choices), **efficiency** (did the agent take a reasonably direct path, or did it take unnecessary, redundant, or wasteful steps that happened not to prevent eventual success but would be problematic at scale due to cost/latency), **error recovery quality** (when something went wrong mid-execution — a failed tool call, an unexpected result — did the agent recover sensibly, or did it flounder/repeat the same failed approach), and **faithfulness of stated reasoning to actual behavior** (does the agent's explicit reasoning trace genuinely reflect and justify the actions it actually took, which matters for interpretability/trust even beyond raw task success). Evaluating only the final answer would miss all of these process-quality dimensions, which matter significantly for understanding whether an agent will generalize reliably to new situations versus having gotten a specific test case right somewhat by chance.

---

### Q: What is a "golden trajectory" or reference execution path for agent evaluation, and what are the limitations of comparing an agent's actual trajectory against a single predefined reference path?

**Answer:**
A golden trajectory is a predefined, expert-validated sequence of steps considered a "correct" or exemplary way to accomplish a given test task, against which an agent's actual execution can be compared. The key limitation: for most non-trivial agentic tasks, there are often **multiple genuinely valid paths** to accomplish the same goal (different but equally reasonable tool orderings, different but equally valid intermediate approaches) — rigidly comparing an agent's actual trajectory against a single golden path risks **penalizing an agent for taking a different but equally valid approach**, producing a misleadingly low evaluation score for genuinely competent behavior that simply didn't match the specific reference path chosen. More robust evaluation approaches often focus on **validating specific required properties/milestones** the trajectory should satisfy (e.g., "the agent must have verified X before proceeding to Y," or "the final answer must be grounded in information actually retrieved during the trajectory") rather than requiring exact match against one specific predefined path.

---

### Q: What is "LLM-as-judge" evaluation for agentic systems, and what specific challenges arise when applying this technique to evaluate multi-step agent behavior compared to evaluating a single-turn LLM response?

**Answer:**
LLM-as-judge uses a (typically strong) LLM to evaluate another system's output against a rubric, as a scalable proxy for human judgment (discussed generally in the LLMOps repo's evaluation topic). For **multi-step agent behavior** specifically, additional challenges arise: the judge needs to evaluate a **potentially long, complex trace** (not just a short single response), which can itself run into context-length and "lost in the middle" issues if the trajectory is lengthy, and the judge needs a genuinely clear, well-specified rubric for **process quality** (tool selection appropriateness, reasoning coherence) which is inherently harder to specify precisely than rubrics for evaluating a single, self-contained text response — a vague "was this a good agent trajectory" judge prompt is likely to produce much less reliable, consistent scoring than a well-decomposed rubric evaluating specific, more narrowly-defined aspects of the trajectory (tool correctness, efficiency, final answer accuracy) somewhat independently before combining into an overall assessment.

---

### Q: What is a "test suite" for an agentic system meant to look like, and how would you structure test cases to cover both common/expected scenarios and edge cases/failure modes specifically relevant to agentic behavior (like tool failures or ambiguous instructions)?

**Answer:**
A comprehensive agent test suite should include: **happy-path scenarios** (common, well-defined tasks the agent is expected to handle reliably), **ambiguous/underspecified scenarios** (testing whether the agent appropriately asks for clarification or makes reasonable assumptions, rather than confidently proceeding on a likely-wrong interpretation), **tool failure scenarios** (deliberately simulating a tool returning an error, timing out, or returning unexpected/malformed data, verifying the agent handles this gracefully rather than crashing or producing a nonsensical response), **adversarial/edge-case inputs** (testing resistance to prompt injection or unusual, boundary-pushing requests, tying into the security topic), and **long-horizon/complex multi-step scenarios** (testing whether the agent maintains coherent behavior/context across many steps, not just short, simple tasks). A common architectural mistake is building a test suite that only covers happy-path scenarios, since this can give a false sense of confidence about an agent's real-world reliability, when in practice tool failures and ambiguous requests are common, expected occurrences in real production use, not rare edge cases.

---

### Q: What is regression testing for an agentic system, and why is it particularly important (and particularly tricky) to maintain as prompts, tools, or the underlying model version change over time?

**Answer:**
Regression testing verifies that previously-working behavior/scenarios still work correctly after a change (a prompt tweak, a new tool added, an underlying model upgrade), catching unintended side effects of changes intended to fix or improve something else. This is particularly important for agentic systems because of the **entanglement risk** discussed in the ML fundamentals context — a prompt change intended to fix one specific failure mode can subtly alter the agent's behavior in unrelated scenarios that weren't the target of the change, given how sensitive LLM behavior can be to prompt phrasing. It's particularly **tricky to maintain** because, unlike traditional software regression tests (which typically have deterministic pass/fail assertions), agent behavior is **non-deterministic and probabilistic** — a regression test needs to account for acceptable variation in exact wording/approach while still reliably catching genuine behavioral regressions, which usually means regression test assertions need to focus on **key properties/outcomes** (did the agent still successfully complete the task, did it still call the necessary tools) rather than expecting exact output matching, similar to the golden-trajectory limitation discussed above.

---

### Q: What is the difference between evaluating an agent's performance on a fixed, static benchmark dataset versus evaluating it through live, real-world traffic/usage, and why does an architect need both?

**Answer:**
A **static benchmark** provides a controlled, repeatable, and comparable evaluation — useful for tracking whether a specific change (prompt, model, tool) improves or regresses performance on a known, fixed set of scenarios, and essential for the kind of pre-deployment CI/CD evaluation gates discussed in the LLMOps repo. **Live, real-world evaluation** (monitoring actual production usage, collecting user feedback, tracking real task completion/success rates) captures the genuine, evolving diversity of real user requests that any necessarily-finite static benchmark can't fully anticipate — real usage will inevitably surface scenarios, edge cases, and user intents the benchmark's original designers didn't think to include. An architect needs **both**: the static benchmark for controlled, comparable pre-deployment validation and regression testing, and live evaluation for catching the real-world gaps a static benchmark inherently can't cover, ideally with a feedback loop where **interesting/problematic real-world cases discovered in production get incorporated back into the static benchmark** over time, continuously improving its real-world representativeness.

---

### Q: What is "task success rate" as an agent evaluation metric, and what important nuance is lost if this is the only metric tracked, without also considering cost, latency, and the rate of unsafe/undesirable actions taken along the way?

**Answer:**
Task success rate measures the fraction of test/real tasks the agent ultimately completes correctly — an intuitive, important top-line metric, but insufficient in isolation because it says nothing about the **cost efficiency** of achieving that success (an agent that succeeds 95% of the time but takes 20 expensive tool calls and 2 minutes per task might be far less viable in production than one succeeding 90% of the time with 3 calls and 10 seconds), the **latency experience** for the end user, or — critically for safety — whether the agent took any **genuinely risky or undesirable actions along the way even in cases where it ultimately arrived at a correct final answer** (e.g., an agent that succeeds at a task but did so by taking an action outside its intended scope, or by ignoring an explicit user constraint at some intermediate step that happened not to affect the final outcome). A comprehensive agent evaluation dashboard needs to track **success rate alongside cost, latency, and safety/compliance metrics together**, since optimizing purely for success rate in isolation can inadvertently produce or mask exactly the kind of inefficient or unsafe behavior that matters most to catch before production deployment.

---

### Q: What is "human-in-the-loop evaluation" for agentic systems, and how would you design an efficient process for incorporating human review into agent evaluation without requiring a human to manually review every single test case or production interaction?

**Answer:**
Human-in-the-loop evaluation incorporates genuine human judgment (which remains more reliable than automated proxies like LLM-as-judge for many nuanced quality dimensions) into the evaluation process, but doing this for *every* interaction is generally infeasible at any meaningful scale. Efficient designs: **stratified sampling** (having humans review a representative sample of interactions rather than all of them, potentially oversampling specific categories of interest — e.g., interactions the automated evaluation flagged as low-confidence or unusual), **using automated evaluation as a first-pass filter** (routing only the cases automated methods flag as potentially problematic, low-confidence, or genuinely novel/edge-case for human review, rather than having humans review the presumably larger volume of clearly successful, unremarkable interactions), and **calibration checks** (periodically having humans review a random sample of interactions the automated evaluation scored as "good," specifically to verify the automated evaluation's own reliability/calibration hasn't drifted, rather than only ever reviewing cases the automation already flagged as concerning, which could miss cases where the automated evaluation itself has a systematic blind spot).

---

### Q: How would you design an evaluation strategy specifically for testing an agent's behavior under a scenario where a tool it depends on becomes unavailable or starts returning degraded/incorrect results, given that this is a realistic production scenario but hard to reliably trigger through normal testing?

**Answer:**
This requires **deliberate fault injection** as part of the test suite — rather than only testing against tools behaving correctly, build test scenarios that **explicitly mock/simulate tool failures, timeouts, and malformed/incorrect results**, verifying the agent's behavior in each specific failure mode: does it retry sensibly (with appropriate limits, not infinitely), does it fall back to an alternative approach if one's available, does it clearly communicate the limitation to the end user rather than either silently failing or confidently proceeding as if the failed/degraded tool had actually succeeded, and does it avoid making increasingly erratic or nonsensical decisions as it accumulates failed observations. This kind of fault-injection testing mirrors the "chaos engineering" concept discussed in the AI Ops repo's reliability topic, applied specifically at the agent-behavior level rather than the infrastructure level — testing genuine resilience to realistic degraded conditions rather than only validating behavior under ideal, everything-works-correctly conditions that may not reliably reflect real production reality.

---
