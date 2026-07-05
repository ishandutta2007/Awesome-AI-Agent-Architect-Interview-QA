# 🧩 Agent Fundamentals & Architecture Patterns

[← Back to main README](../README.md)

---

### Q: What makes an LLM-based system an "agent" rather than just a chatbot or a simple prompt-response application?

**Answer:**
An agent is distinguished by combining an LLM with a **loop of perception, reasoning, and action** — it can decide, based on the current state and its goal, to take **actions** (calling tools, querying data, taking steps in the world), observe the results, and iterate — rather than producing a single, direct response to a single input. A simple chatbot or prompt-response application maps one input to one output with no autonomous decision-making about *what to do next*. The key architectural elements that make something agentic are typically: some form of **goal or task specification**, **tool/action access**, a **loop that persists across multiple reasoning-action steps** (not just one LLM call), and often (though not always) some form of **memory** persisting across those steps or across sessions.

---

### Q: What is the ReAct (Reasoning + Acting) pattern, and why was interleaving explicit reasoning with actions an improvement over having the model just directly output actions?

**Answer:**
ReAct prompts the model to alternate between explicit **reasoning traces** ("Thought: I need to find X, so I should search for Y") and **actions** (calling a tool), with the results of each action fed back in as an **observation** before the next reasoning step. This interleaving improves over directly outputting actions because it gives the model an explicit space to **plan, self-correct, and reason about intermediate results** before committing to the next action — in practice, this significantly improves the model's ability to recover from unexpected tool outputs (e.g., a search returning irrelevant results, prompting a "Thought" that reformulates the query) rather than blindly proceeding with a rigid action sequence decided upfront, and it also produces a more interpretable trace of *why* the agent took each action, which matters a lot for debugging and trust.

---

### Q: What is the difference between a single-agent architecture and a multi-agent architecture, and what's a concrete scenario where splitting a task across multiple specialized agents outperforms a single, more capable "do everything" agent?

**Answer:**
A **single-agent architecture** has one LLM-driven loop handling the entire task end-to-end, using whatever tools/context it needs. A **multi-agent architecture** decomposes the work across multiple agents, each often with a narrower role/expertise, coordinating (directly or via an orchestrator) toward the overall goal. A concrete scenario favoring multi-agent: a **research-and-report-writing task** where one agent specializes in web research/fact-gathering (with search tool access and a prompt tuned for thorough, citation-aware retrieval) and a separate agent specializes in synthesizing/writing (with a prompt tuned for clear, well-structured prose) — splitting these roles often produces better results than one agent context-switching between "research mode" and "writing mode" within a single long, increasingly cluttered context, and it also makes each agent's behavior easier to test, prompt-tune, and debug independently.

---

### Q: What is the difference between a stateless and a stateful agent architecture, and why does even a "simple" single-turn tool-using agent typically need at least some state management within a single task execution?

**Answer:**
A **stateless** agent treats every request independently with no retained context between calls. A **stateful** agent maintains context — either short-term (within a single multi-step task) or long-term (across sessions, via persistent memory). Even a "simple" single-turn tool-using agent needs **within-task state management** because a multi-step task (e.g., "search for X, then use the results to calculate Y, then format the answer") requires the agent to accumulate and track the results of each intermediate step to inform subsequent ones — this working state (the accumulating trace of thoughts/actions/observations, as in ReAct) is a form of state that exists even without any long-term memory across separate sessions. The distinction between "state within a task execution" and "state persisted across sessions" (the domain of the memory systems discussed later) is an important one to draw clearly in an architecture discussion.

---

### Q: What is the "agent loop," and what are its core components at a high level, regardless of specific framework implementation?

**Answer:**
The agent loop is the core execution cycle: (1) **Perceive/observe** — gather the current state (user input, tool results, environment state). (2) **Reason/plan** — the LLM decides what to do next given the current state and goal (this may include explicit chain-of-thought reasoning). (3) **Act** — execute a chosen action (call a tool, respond to the user, delegate to another agent). (4) **Observe the result** — capture the outcome of that action. (5) **Repeat or terminate** — loop back to reasoning with the new information, or stop if the goal is achieved or a stopping condition (max iterations, explicit "done" signal) is met. Nearly every agent framework (LangGraph, AutoGen, custom implementations) implements some variant of this loop — understanding it at this framework-agnostic level is what lets an architect evaluate and compare different frameworks' specific implementations meaningfully.

---

### Q: What is the difference between a "planner-executor" agent architecture and a "reactive" (step-by-step ReAct-style) agent architecture, and what's the key tradeoff between them?

**Answer:**
A **reactive/ReAct-style** agent decides its next single action at each step, without committing to a full plan upfront — highly adaptive to unexpected observations, since each step can freely change direction based on what was just learned, but can be less efficient/predictable for complex tasks (the agent might backtrack or wander before converging on an effective approach). A **planner-executor** architecture first generates an explicit, multi-step **plan** upfront (often via a dedicated "planner" LLM call), then an "executor" component carries out each planned step (possibly with some ability to replan if a step fails or reveals new information). The key tradeoff: planner-executor architectures tend to produce more **structured, predictable, and efficient** execution for well-understood task types, while reactive architectures tend to be more **robust to genuinely novel or highly uncertain situations** where committing to a rigid upfront plan would be premature — many production systems use a hybrid, generating a rough initial plan while retaining reactive replanning capability when execution reveals the plan needs adjustment.

---

### Q: What is the difference between "agentic" and merely "tool-augmented" LLM applications, and why does this distinction matter when scoping an architecture design?

**Answer:**
A **tool-augmented** application lets an LLM call a fixed, small set of tools within a single, bounded interaction (e.g., a chatbot that can look up the weather when asked) — the tool use is typically a single step or a short, predictable sequence directly serving one user turn. A genuinely **agentic** application involves the LLM **autonomously deciding a sequence of steps/tools to pursue a broader goal**, potentially across many iterations, with meaningful decision-making about *what to do next* rather than a largely pre-determined tool-call pattern. This distinction matters for architecture scoping because tool-augmented applications can often be built with much simpler, more constrained/predictable architectures (less need for complex planning, extensive guardrails, or sophisticated failure recovery), while genuinely agentic systems need to invest much more heavily in the concerns covered throughout this repo (planning, memory, safety/guardrails, evaluation) proportional to the degree of autonomy and open-endedness actually required.

---

### Q: What is the "orchestrator" pattern in agent architecture, and what specific responsibilities does an orchestrator component typically own that individual agents/tools don't?

**Answer:**
An orchestrator is a component (which may itself be LLM-driven or more traditionally rule-based/deterministic) responsible for **coordinating the overall flow** of a multi-step or multi-agent task, without necessarily doing the domain-specific work itself. Typical orchestrator responsibilities: **routing** (deciding which agent/tool should handle the current step), **state management** (tracking overall task progress and passing relevant context between steps/agents), **error handling and retries** at the workflow level (distinct from an individual agent's own internal error handling), and **termination logic** (deciding when the overall task is complete, has failed, or needs escalation to a human). Separating orchestration from individual agent/tool logic is an important architectural principle because it keeps each individual agent focused and simpler to reason about/test, while centralizing the more complex, cross-cutting coordination logic in one well-defined place rather than scattering it implicitly across many agents' individual behaviors.

---

### Q: What is the difference between designing an agent to be "goal-directed" versus "instruction-following," and why does this distinction significantly affect how much autonomy/guardrails an architecture needs?

**Answer:**
An **instruction-following** agent executes a relatively specific, well-defined set of steps/instructions provided by the user or a calling system, with limited latitude to deviate from what was explicitly asked. A **goal-directed** agent is given a higher-level objective and has significant latitude to **determine its own steps/approach** to achieve that goal, potentially including sub-goals the user never explicitly specified. This distinction significantly affects architecture because goal-directed agents, by design, can take actions the designer didn't explicitly anticipate — this requires **substantially more investment in guardrails, action-scoping (what tools/permissions the agent actually has access to), and evaluation for open-ended/unexpected behavior**, compared to a narrower instruction-following agent whose action space is inherently more constrained and predictable by construction, simply because it's not expected to autonomously determine novel approaches on its own.

---

### Q: What is "grounding" in the context of an agent's actions, and why is an agent that can only reason in text (without grounding to real tools/data/actions) fundamentally limited for most practical agentic use cases?

**Answer:**
Grounding refers to connecting an agent's reasoning to **actual, verifiable external reality** — real data retrieval, real tool execution, real environment feedback — rather than the model purely reasoning based on its internal, potentially outdated or hallucination-prone training knowledge. An agent that can only reason in text without grounding is fundamentally limited because it has **no way to verify its assumptions against current, accurate information**, no way to actually **effect change in the world** (which is usually the entire point of an agentic system, as opposed to a purely conversational one), and no **feedback loop to catch and correct its own errors** — grounding via tool use, data retrieval, and observing real action outcomes is what transforms an LLM from a text generator into something capable of genuinely useful, verifiable, corrective agentic behavior.

---

### Q: What is the difference between designing an agent for a "closed" task domain (well-defined, bounded set of possible actions/outcomes) versus an "open" task domain (broad, less predictable scope), and how does this affect architecture choices around tool design?

**Answer:**
A **closed** task domain (e.g., "process this specific type of customer support ticket using these five specific tools") has a well-understood, bounded action space — allowing more **specific, purpose-built tool design**, more thorough upfront testing/evaluation coverage (since the space of possible scenarios is enumerable), and tighter guardrails (since expected behavior is well-defined). An **open** task domain (e.g., "help the user accomplish whatever research task they bring") requires more **general-purpose tools** and reasoning capability, since the specific needs can't be fully anticipated in advance — this generally requires more robust error handling/recovery (since novel, unanticipated situations will occur more often), more emphasis on the agent's own judgment/reasoning quality (since specific procedures can't be hard-coded for every scenario), and typically a higher tolerance for occasional imperfect behavior, balanced against correspondingly more investment in monitoring and human-in-the-loop escalation for cases outside expected bounds.

---

### Q: How would you decide, as an architect, whether a given business problem actually warrants an agentic solution versus a simpler, more traditional deterministic workflow or a single-call LLM application?

**Answer:**
Key questions to ask: **Does the task genuinely require multi-step, adaptive decision-making** where the right sequence of actions can't be fully predetermined (favoring an agentic approach), or is it actually a **well-defined, repeatable sequence** that could be implemented as a deterministic pipeline with LLM calls only at specific points needing natural language understanding/generation (often simpler, more predictable, and cheaper than a full agentic loop)? **Is the cost of occasional agent errors/unpredictability acceptable** for this use case, given that agentic systems inherently trade some predictability for flexibility — a high-stakes, low-error-tolerance task might be better served by a more constrained, deterministic system with LLM calls only in well-tested, bounded roles. A common architectural anti-pattern is reaching for a full agentic architecture by default when a much simpler, more reliable, and more debuggable deterministic pipeline (potentially with one or two well-scoped LLM calls) would actually serve the business need just as well with substantially less complexity and risk.

---
