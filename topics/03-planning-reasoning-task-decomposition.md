# 🗺️ Planning, Reasoning & Task Decomposition

[← Back to main README](../README.md)

---

### Q: What is task decomposition in agent architecture, and why does breaking a complex goal into smaller sub-tasks generally improve an agent's success rate compared to attempting the entire goal in one pass?

**Answer:**
Task decomposition breaks a complex, high-level goal into a sequence (or hierarchy) of smaller, more concrete sub-tasks that are individually more tractable for the model to reason about and execute correctly. This generally improves success rate because it **reduces the reasoning burden at each individual step** (a model reasoning about "what's the very next concrete action to take" is generally more reliable than reasoning about "solve this entire complex, multi-faceted problem in one shot"), makes **intermediate progress and errors more visible/correctable** (a wrong sub-task result can be caught and corrected before it propagates through the rest of the plan, rather than being buried within one large, monolithic reasoning attempt), and often allows **sub-tasks to be delegated to more specialized components** (a dedicated tool, a specialized sub-agent, or a more targeted prompt) rather than requiring one generalist reasoning process to handle every aspect of the problem equally well.

---

### Q: What is the difference between "chain-of-thought" prompting and genuine multi-step agentic planning, and why is chain-of-thought alone insufficient for tasks requiring real-world tool use or verification?

**Answer:**
**Chain-of-thought (CoT)** prompting encourages a model to reason step-by-step *within a single response*, improving reasoning quality for problems solvable through better internal deliberation alone (e.g., a multi-step math problem). **Agentic planning** involves the model explicitly determining a sequence of **real actions** (tool calls, sub-tasks) to take across **multiple separate reasoning-action cycles**, with real-world feedback incorporated between steps. CoT alone is insufficient for tasks requiring tool use or verification because it's fundamentally still just **the model reasoning in isolation, without any grounding in real, current information or the ability to verify intermediate conclusions against reality** — a model can produce a perfectly coherent, well-reasoned chain of thought that's nonetheless factually wrong because it had no way to check its assumptions against real data; agentic planning's iterative act-observe cycle is specifically what provides that missing grounding and error-correction capability that pure internal reasoning cannot.

---

### Q: What is hierarchical planning in a multi-agent or complex single-agent system, and give an example of when a flat, single-level plan would be insufficient?

**Answer:**
Hierarchical planning organizes a task into **multiple levels of abstraction** — a high-level plan consisting of broad sub-goals, each of which is further decomposed into more concrete steps only when the agent actually reaches that point in execution (rather than fully planning out every low-level detail upfront). A flat, single-level plan becomes insufficient for **complex, long-horizon tasks where the details of later steps genuinely depend on the outcomes of earlier steps** — e.g., planning a multi-city research trip itinerary: the high-level plan ("research city A, then city B, then synthesize a comparison") is stable and sensible to commit to upfront, but the *specific* low-level steps needed to research city A (which sources to check, what specific questions to ask) can't be usefully or accurately planned in full detail before actually starting that sub-task, since what's needed depends on what's discovered along the way — hierarchical planning lets the agent commit to a stable high-level structure while deferring low-level planning detail until it's actually actionable and informed by real progress.

---

### Q: What is "replanning," and what specific triggers should cause an agent architecture to abandon or revise its current plan rather than rigidly continuing to execute it?

**Answer:**
Replanning is the process of revising an agent's plan mid-execution in response to new information that invalidates or changes the value of the original plan. Triggers that should prompt replanning: a **tool call fails or returns an unexpected/contradictory result** that the original plan didn't account for, **new information is discovered during execution that changes what's actually needed** (e.g., a research step reveals the original approach was based on an incorrect assumption), or **an intermediate step reveals the overall goal itself needs clarification/adjustment** (e.g., discovering ambiguity in the original task that wasn't apparent until partway through). An architecture that **can't replan at all** (rigidly executing a fixed plan regardless of what happens during execution) will fail ungracefully whenever real-world execution doesn't perfectly match the plan's assumptions — which, for any sufficiently complex or open-ended task, is a common and expected occurrence, not a rare edge case.

---

### Q: What is the risk of "over-planning" or excessive upfront deliberation in an agent architecture, and how would you design a system to balance planning thoroughness against responsiveness/cost?

**Answer:**
Excessive upfront planning — e.g., an agent spending many reasoning steps and tool calls deliberating over an elaborate plan before taking any real action — increases **latency and cost** for the user, and can also produce a plan based on **incomplete information that would be better resolved by simply starting execution and gathering real feedback sooner**, rather than trying to anticipate everything in advance through pure reasoning. Balancing this architecturally: use a **"just enough" planning depth** appropriate to task complexity (a simple, well-understood task might skip explicit planning almost entirely and go straight to a reactive execution loop, while only genuinely complex, multi-step tasks warrant a more deliberate upfront planning phase), and design for **planning and execution to interleave** rather than treating them as strictly sequential phases — allowing the agent to start acting on the parts of the plan it's confident about while still refining the less-certain parts, rather than blocking all action until an exhaustive, fully-detailed plan is finalized.

---

### Q: What is self-reflection/self-critique in agent architectures (e.g., an agent reviewing its own output or plan before finalizing it), and what's the architectural cost/benefit tradeoff of adding this as an explicit step?

**Answer:**
Self-reflection adds an explicit step where the agent (often via a separate prompt or even a separate model call) **reviews and critiques its own prior output or proposed plan** before committing to it or presenting it as final — e.g., "here's my draft answer; now critically review it for errors or gaps before finalizing." This can meaningfully **improve output quality/correctness** by catching errors the original generation missed, similar in spirit to a human proofreading their own work before submitting it. The cost is **additional latency and token/compute cost** for the extra reasoning step(s), and self-reflection isn't a guaranteed fix — a model reviewing its own work can still miss the same blind spots that caused the original error, particularly for errors stemming from a genuine knowledge gap rather than a careless mistake. Architecturally, self-reflection is generally most valuable for **higher-stakes outputs where the added latency/cost is justified by the quality improvement**, and is often applied selectively (e.g., only for final outputs or particularly consequential decisions) rather than after every single intermediate reasoning step.

---

### Q: What is the difference between explicit, structured planning output (e.g., a numbered list of steps the agent commits to) versus implicit planning embedded within free-form reasoning traces, and why might an architect prefer explicit structured plans for certain use cases?

**Answer:**
**Implicit planning** happens within the model's free-form reasoning (as in a ReAct-style "Thought" step) without ever producing a distinct, separately-inspectable plan artifact. **Explicit structured planning** has the model produce a formal, distinctly-formatted plan (e.g., a JSON list of discrete steps with dependencies) as its own artifact, separate from the ongoing execution trace. An architect might prefer explicit structured plans when: the plan needs to be **shown to a human for review/approval before execution** (much easier to review a clear, structured list of intended steps than to parse an implicit plan buried within free-form reasoning text), the system needs to **programmatically track progress against the plan** (checking off completed steps, detecting deviation), or **multiple agents need to coordinate around a shared plan** (a structured artifact is much easier for a separate orchestrator or other agents to consume and reason about than unstructured text) — the tradeoff is that forcing planning into a rigid structure can sometimes be less naturally expressive than free-form reasoning for genuinely fluid, hard-to-formalize planning situations.

---

### Q: What is the "context degradation" problem in long-running agentic tasks with many reasoning/action steps, and what architectural strategies mitigate it?

**Answer:**
As an agent executes many steps, its accumulated context (all prior thoughts, actions, and observations) grows, eventually **consuming a large portion of the context window** and, in some cases, degrading reasoning quality as genuinely relevant recent information gets diluted among a long history — similar to the "lost in the middle" effect discussed in LLMOps contexts. Architectural mitigations: **periodic summarization/compression** of the accumulated trace (condensing older steps into a brief summary while retaining full detail only for recent, directly relevant steps), **explicit working memory management** (deliberately deciding what information genuinely needs to persist in context versus what can be dropped or offloaded to external storage, discussed further in the memory systems section), and **task decomposition itself** (breaking a very long task into genuinely separate sub-tasks, each with a fresh, more focused context, rather than accumulating one ever-growing context for the entire end-to-end task) — this is a genuine, recurring architectural challenge for any agent expected to operate over long task horizons, not something that can be ignored until it becomes a problem in production.

---

### Q: What is a "critic" or "verifier" pattern in agent architecture, where a separate component evaluates a proposed action/plan before it's executed, and how does this differ from self-reflection performed by the same agent?

**Answer:**
A critic/verifier pattern uses a **separate, independent component** (potentially a different model, a different prompt with a distinctly different role/perspective, or even a non-LLM rule-based check) to evaluate a proposed plan or action **before** it's committed to or executed, rather than relying on the same agent that generated the proposal to also judge its own quality. This differs from self-reflection (where the same agent reviews its own work) in an important way: a genuinely independent critic is **less susceptible to the same blind spots or biases** that might have led to the original flawed proposal in the first place, since it approaches the evaluation with a different, dedicated framing rather than potentially anchoring on the same reasoning that produced the original (possibly flawed) output. This pattern is particularly valuable for **higher-stakes decisions** where the cost of an extra verification step (latency, compute) is justified by meaningfully reducing the risk of executing a flawed or unsafe action.

---

### Q: How would you design an agent's planning approach differently for a task with a well-defined, verifiable success criterion (e.g., "write code that passes these specific tests") versus a task with a fuzzy, subjective success criterion (e.g., "write a compelling marketing email")?

**Answer:**
For a **well-defined, verifiable** task, the architecture can lean heavily on an **iterative generate-test-refine loop** — the agent proposes a solution, actually verifies it against the concrete success criterion (running the tests), and uses failures as direct, unambiguous, actionable feedback to refine its approach, potentially iterating many times with high confidence about when it's actually succeeded. For a **fuzzy, subjective** task, there's no equivalent objective verification signal available — the architecture instead needs to rely more heavily on **well-designed prompting/guidelines capturing the subjective quality criteria as explicitly as possible**, potentially a **human-in-the-loop review step** (since no automated check can reliably substitute for human judgment on genuinely subjective quality), or an **LLM-as-judge evaluation** (as discussed in LLMOps) as an imperfect but scalable proxy for human judgment — critically, the architect needs to recognize that these two task types warrant genuinely different planning/verification architectures, rather than applying the same "keep iterating until it looks done" approach uniformly regardless of whether a truly objective success signal actually exists.

---
