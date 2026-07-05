# 🎭 Scenario-based & Behavioral

[← Back to main README](../README.md)

> Scenario questions test architectural and product judgment under realistic ambiguity — there's often no single "correct" answer, but strong answers show a clear, structured design process and awareness of tradeoffs. Behavioral questions use the **STAR method** (Situation, Task, Action, Result) — the frameworks below are for structuring your *own* real experience, not for fabricating one.

---

### Q: "Design an agentic system for automating customer support ticket triage and initial response for a mid-sized SaaS company. Walk me through your architecture."

**Answer (framework):**
Structure your answer around key decisions: (1) **Scope/autonomy level** — start by clarifying what the agent should fully automate (categorization, drafting a response) versus what should always route to a human (refund approvals, anything the agent has low confidence about) — reflecting the HITL/escalation design principles discussed throughout this repo. (2) **Architecture** — likely a **single specialized agent or a small hierarchical set** (a triage/classification step, a retrieval step against a knowledge base for common issues, a response drafting step), rather than an overly elaborate multi-agent system for what's fundamentally a well-scoped, bounded task. (3) **Tools** — a knowledge base retrieval tool, a ticket categorization/tagging tool, potentially a "check customer account status" read-only tool; explicitly note which tools are read-only versus which have write/action capability, and apply extra scrutiny to the latter. (4) **Guardrails** — explicit escalation criteria (low confidence, sensitive keywords like "legal" or "cancel," any refund/credit above a threshold). (5) **Evaluation** — a golden test set of representative historical tickets, tracking both response quality and appropriate escalation behavior. Close by acknowledging what you'd want to validate with real usage data before expanding the agent's autonomy further.

---

### Q: "A stakeholder wants an agent to have full autonomous authority to resolve customer refund requests up to any amount, arguing this maximizes efficiency. How do you respond, and how would you architect this differently?"

**Answer (framework):**
Show the ability to push back constructively with concrete reasoning rather than either blindly agreeing or being dismissive of the efficiency goal. Structure: acknowledge the genuine efficiency benefit motivating the request, then explain the **specific, concrete risk** of unlimited autonomous authority (an agent's occasional reasoning errors or susceptibility to manipulation, applied to unlimited-value financial transactions, creates unbounded downside risk, unlike a bounded-value scenario), and propose a **concrete alternative architecture** reflecting the staged-autonomy and reversibility principles discussed in the safety topic — e.g., full autonomy for refunds below a validated, data-informed threshold, with human approval required above it, and a plan to **revisit and potentially raise that threshold over time** based on accumulated evidence of the agent's reliability at the current threshold. This demonstrates you can translate an architectural safety principle into a concrete, actionable compromise that still meaningfully serves the stakeholder's efficiency goal, rather than either rejecting the request outright or accepting an ill-advised design as requested.

---

### Q: "How would you design an agent architecture to handle a task where you genuinely don't yet know all the tools/data sources it might need, because the space of user requests is very broad and evolving?"

**Answer (framework):**
This tests architectural thinking about **extensibility and graceful handling of gaps** rather than assuming perfect upfront specification is possible. Good structure: design for **explicit "I don't have a tool for this" acknowledgment** rather than the agent attempting a task it has no appropriate capability for and producing an unreliable, unsupported result — an honest capability gap is architecturally preferable to a confident wrong answer. Discuss designing the **tool catalog to be extensible** (a clean, standardized way to add new tools over time, potentially via a protocol like MCP discussed in the frameworks topic, rather than a tightly-coupled, hard-to-extend tool integration approach) and instrumenting **production usage/gap analysis** (tracking cases where the agent had to decline or escalate due to a missing capability, using this data to prioritize which new tools/data sources to actually build next, driven by real observed demand rather than speculative upfront guessing about every possible future need).

---

### Q: "Tell me about a time you had to decide against building a fully autonomous agentic solution, in favor of a simpler, more constrained/deterministic system, even though 'agent' was the more exciting or trendy technical approach."

**Answer (framework):**
This tests genuine engineering judgment and resistance to over-engineering/hype-driven design, echoing the fundamentals topic's discussion of when agentic architecture is and isn't actually warranted. Structure: describe the specific problem and why an agentic approach was initially considered/proposed, the **specific reasoning that led you to a simpler alternative** (e.g., the task was actually a well-defined, repeatable sequence better served by a deterministic pipeline with LLM calls only at specific points, or the acceptable error tolerance for the use case was too low to justify an agent's inherent unpredictability), and the outcome — ideally showing the simpler solution actually served the business need effectively, reinforcing that architectural decisions should be driven by genuine requirements fit, not by defaulting to the most sophisticated or currently-fashionable technical approach available.

---

### Q: "Describe a time you had to debug a genuinely confusing or unexpected agent behavior in production. What was your process, and what did you ultimately find?"

**Answer (framework):**
Structure: describe the specific symptom (be concrete — "the agent kept calling the same tool repeatedly with slightly different arguments" is much stronger than "the agent was acting weird"), your **systematic investigation process** (checking the full trace, examining the exact prompts sent at each step, considering the layered-attribution approach discussed in the observability topic — was this a reasoning flaw, a tool bug, or an information gap), and the actual root cause you found. Strong answers here often illustrate exactly the kind of non-obvious failure modes discussed throughout this repo — a subtle prompt template bug, an unexpected tool output format the agent wasn't prepared for, or genuine reasoning limitations under a specific type of ambiguous input. Close with what you changed as a result (added observability, a new guardrail, a regression test case) to prevent recurrence or at least detect it faster next time.

---

### Q: "How would you evaluate whether a multi-agent architecture is actually outperforming a well-designed single-agent alternative for a given task, rather than just assuming more agents means better results?"

**Answer (framework):**
This tests rigor against the "more sophisticated architecture must be better" assumption, tying back to the fundamentals and evaluation topics. Structure your answer around: designing a **genuine head-to-head comparison** (the same evaluation suite, discussed in the evaluation topic, run against both the single-agent and multi-agent implementations), measuring **not just task success rate but cost, latency, and reliability/consistency across repeated runs** (since multi-agent systems often trade some of these for potential quality gains, and that tradeoff needs to be explicitly measured, not assumed), and being genuinely willing to conclude that the **simpler single-agent approach is actually sufficient or even superior** for the specific task if the data supports that conclusion, rather than being anchored to a predetermined belief that the more architecturally sophisticated multi-agent approach must be the better choice.

---

### Q: "A user reports that an agent you built took an action they didn't expect or want, though it was technically within the agent's granted permissions. How do you investigate and respond, both immediately and architecturally?"

**Answer (framework):**
Immediate response: investigate the **specific trace of that interaction** to understand exactly what led to the unexpected action (was it a reasonable interpretation of an ambiguous request, a genuine reasoning error, or a sign of a gap in the guardrail/confirmation design) and address the immediate user impact (can the action be reversed/corrected, and how do you communicate transparently with the affected user). Architectural follow-up: does this reveal a **need for additional confirmation/HITL friction** for this specific action category (tying to the action-reversibility and confirmation design discussed in the safety topic), or a **gap in how the agent handles ambiguous instructions** (should it have asked a clarifying question rather than proceeding on an assumption) — and critically, **does this specific case get added to your evaluation/regression test suite** so this exact scenario (or the general pattern it represents) is explicitly covered going forward, converting a one-off incident into a durable improvement to the system's evaluation coverage and guardrails.

---

### Q: "How do you approach explaining the inherent unpredictability/non-determinism of agentic systems to a stakeholder or leadership team used to traditional software's more deterministic behavior guarantees?"

**Answer (framework):**
This tests communication skill translating a genuinely important technical property into terms a less technical audience can act on. Good structure: use an accessible framing (e.g., comparing traditional software to "a very precise but rigid set of instructions" versus an agent to "a capable but occasionally imperfect employee who can handle novel situations flexibly but won't be right 100% of the time") rather than a purely technical explanation, be **honest and specific about the actual expected reliability** (grounded in real evaluation data, not vague reassurance) rather than either overselling the system's reliability or being needlessly alarmist, and pivot the conversation toward **what safeguards are in place to manage that inherent unpredictability** (the guardrails, HITL escalation, and monitoring discussed throughout this repo) — stakeholders are often more reassured by a clear-eyed explanation of "here's the realistic reliability level and here's exactly how we manage the gap" than by an implicit promise of the same deterministic reliability guarantee traditional software provides, which agentic systems genuinely cannot offer in the same way.

---

### Q: "Tell me about a time you had to balance a product/business team's desire for more agent autonomy (to improve user experience/efficiency) against your own assessment of the safety/reliability risk of granting that autonomy."

**Answer (framework):**
Structure: describe the specific autonomy being requested and the genuine business motivation behind it, your **specific risk assessment** (grounded in the reversibility, confidence, and evaluation-evidence framework discussed in the safety topic, not just a vague, generic sense of caution), and how you **navigated the disagreement constructively** — did you propose a staged/data-driven path to eventually reach the desired autonomy level rather than a flat refusal, did you find additional safeguards that made the requested autonomy acceptable, or did the business team's perspective change your own view once you understood their reasoning more fully. Close with the actual outcome and, ideally, what you learned about calibrating this specific kind of tradeoff that you've since applied to later decisions.

---

### Q: "Where do you see agentic AI systems architecture heading over the next few years, and how are you positioning your own skills for that?"

**Answer (framework):**
Interviewers want genuine, specific perspective, not generic AI-hype enthusiasm. A strong answer discusses **specific, concrete trends** you've actually observed or thought carefully about — e.g., the growing standardization around protocols like MCP reducing integration fragmentation, increasing sophistication in multi-agent coordination patterns as the field matures past early experimental approaches, growing emphasis on rigorous evaluation methodology as agentic systems move from demos into genuinely high-stakes production use, or evolving safety/governance expectations as regulatory attention increases — and connects this to **your own concrete skill development plan** (e.g., "I've been focused on deepening my evaluation/testing methodology specifically because I think that's currently the most significant maturity gap between impressive demos and genuinely reliable production agentic systems"). Showing you're actively tracking and critically thinking about a genuinely fast-evolving field — rather than treating your current knowledge as a finished, static destination — is the differentiator interviewers are looking for.

---

### Q: "Describe your experience (or how you'd approach) working cross-functionally with product managers or business stakeholders who may have unrealistic expectations about what current agentic AI systems can reliably do."

**Answer (framework):**
This tests the ability to be a constructive, educating technical partner rather than either simply saying "no" repeatedly or over-promising to avoid friction. Good structure: describe how you'd **ground the conversation in concrete demonstrations/evaluation data** rather than abstract capability claims (showing, not just telling, what the system can and can't currently do reliably), how you'd **reframe an unrealistic ask into an achievable, staged path** (similar to the autonomy-calibration framing discussed earlier — "we can't do X fully autonomously today, but here's a version with appropriate human oversight we could ship now, with a credible path to more autonomy as we validate reliability") rather than a flat rejection, and how you'd **maintain the relationship/trust over time** by being consistently honest about capabilities and limitations, since credibility built through accurate expectation-setting pays off significantly in a stakeholder's willingness to trust your judgment on future, harder architectural tradeoffs.

---
