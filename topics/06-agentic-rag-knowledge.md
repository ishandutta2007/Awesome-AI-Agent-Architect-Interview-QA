# 📚 Agentic RAG & Knowledge Integration

[← Back to main README](../README.md)

---

### Q: What is "agentic RAG," and how does it differ architecturally from standard, single-pass RAG (retrieve once, generate once)?

**Answer:**
Standard RAG performs a **single retrieval step** based on the initial query, then generates a response grounded in whatever was retrieved. **Agentic RAG** treats retrieval as **one of several tools an agent can invoke iteratively and adaptively** — the agent can decide to search, evaluate whether the results are actually sufficient/relevant, reformulate and search again if not, search multiple different sources sequentially based on what it's learned so far, or combine retrieval with other tool use entirely (e.g., retrieving a document, then using a calculation tool on data found within it). This is architecturally more powerful for complex, multi-hop questions (where the final answer genuinely requires synthesizing information discovered across multiple, sequentially-informed retrieval steps) but is also **slower and more expensive** than a single-pass retrieval, so the added complexity should be reserved for use cases that genuinely benefit from iterative, adaptive retrieval rather than applied by default to every query.

---

### Q: What is "query decomposition" in agentic RAG, and why might an agent need to break a single complex user question into multiple separate retrieval queries rather than searching with the original question directly?

**Answer:**
Query decomposition breaks a complex question into **multiple simpler sub-questions**, each independently searched, with results combined to answer the original complex question. This matters because a single complex question often has **multiple distinct informational needs embedded within it** that a single retrieval query (especially via semantic/vector search, which works best matching a query against similarly-focused content) may not retrieve well all at once — e.g., "compare the safety records of Company A and Company B's most recent products" genuinely requires retrieving distinct information about two different companies' distinct products, which is better served by **two separate, more targeted retrieval queries** (one per company/product) than a single combined query that a retrieval system might not match well against any single document covering both companies together.

---

### Q: What is "self-correcting retrieval" or "corrective RAG," and how does an agent architecturally determine that its initial retrieval results were insufficient and a different retrieval strategy is needed?

**Answer:**
Self-correcting/corrective RAG adds an explicit **evaluation step** after retrieval, where the agent (often via a dedicated grading/relevance-check prompt, or a lighter-weight classifier) assesses whether the retrieved documents actually appear sufficient and relevant to answer the query, **before** proceeding to generate a final response based on them. If the evaluation determines the results are insufficient, the architecture can trigger a **corrective action** — reformulating the query and retrying, expanding the search to additional sources, or falling back to a different retrieval strategy (e.g., switching from pure semantic search to a broader keyword search) — rather than proceeding to generate a response grounded in retrieval results that were actually inadequate, which would risk producing an unsupported or hallucinated answer despite technically having "done RAG."

---

### Q: What is the difference between an agent choosing which knowledge source to query from among multiple available sources (e.g., an internal wiki, a database, and a web search tool) versus always querying all available sources every time?

**Answer:**
**Always querying all available sources** guarantees the agent doesn't miss potentially relevant information from a source it might have otherwise skipped, but incurs **unnecessary latency and cost** for sources that clearly aren't relevant to a given specific query, and can also introduce **noise** (irrelevant results from an inapplicable source diluting the genuinely relevant retrieved context). An architecture where the **agent selects the appropriate source(s) based on the query's nature** (e.g., recognizing a question about internal company policy should query the internal wiki, not the general web search tool) is generally more efficient and precise, but requires the agent to **correctly reason about which source is likely to be relevant** — a decision that itself can be wrong, potentially skipping a source that actually would have had the needed information. The right design often depends on the **cost/latency profile of the available sources** (cheap, fast sources might reasonably be queried more liberally/by default, while an expensive or slow source is more worth being selective about) and how reliably distinguishable the sources' appropriate use cases actually are.

---

### Q: What is "tool-augmented fact verification" in an agentic RAG system, and why might an architecture include a dedicated verification step that re-checks a generated claim against retrieved sources before finalizing the response?

**Answer:**
Even when a response is generated with retrieved context available, the underlying LLM generation step can still **introduce claims not actually well-supported by that retrieved context** (a form of hallucination that occurs despite having relevant grounding material available, simply because generation isn't perfectly constrained to only use what was retrieved). A dedicated verification step **re-checks the generated response's specific claims against the retrieved source material** (potentially via a separate, focused prompt asking "is this specific claim actually supported by this specific retrieved passage?") before finalizing the output — catching cases where the generation step drifted from or embellished beyond what the retrieved evidence actually supports. This adds latency/cost but is architecturally valuable for **higher-stakes applications** where factual accuracy grounded specifically in verifiable sources matters enough to justify the additional verification step, similar in spirit to the "critic/verifier" pattern discussed in the planning section, applied specifically to fact-grounding.

---

### Q: What is the architectural challenge of combining agentic RAG with an agent's own long-term memory (discussed in the memory systems topic) — how does the agent decide whether to answer from retrieved documents, from its own remembered context about the user/situation, or some combination?

**Answer:**
This requires the architecture to treat **retrieval and memory as related but distinct knowledge sources**, each with different characteristics: retrieved documents typically represent **general, shared, potentially authoritative knowledge** (a company's documentation, a knowledge base), while long-term memory represents **specific, personalized, potentially more subjective context** about this particular user/situation. A well-designed architecture generally treats these as **complementary rather than competing sources** — e.g., using memory to inform *how* to interpret or apply retrieved general knowledge to this specific user's situation (e.g., retrieved general product information combined with remembered knowledge of this specific user's prior stated preferences/constraints), rather than the agent needing to arbitrarily choose one source over the other — the prompt/context construction should generally make clear to the model which pieces of context come from which source (general reference material vs. specific remembered user context) so the model can appropriately weigh and combine them rather than conflating a general fact with a personal, memory-derived one.

---

### Q: What is the tradeoff between giving an agent broad, unrestricted access to search the open web versus restricting it to a curated, internal knowledge base, from both a quality and a control/safety architecture standpoint?

**Answer:**
**Open web search access** provides much broader coverage and up-to-date information, but introduces significant **quality control challenges** — the agent has much less control over the reliability, bias, or accuracy of whatever it retrieves from the open web, and it introduces potential **security/safety risks** (as discussed in later topics, open web content could contain prompt injection attempts, or the agent could be steered toward unreliable/manipulated sources). A **curated internal knowledge base** provides much tighter control over source quality/reliability and eliminates most external content-based security risks, but is inherently limited to whatever's actually been curated/indexed, potentially missing relevant information that exists but wasn't included. Many production architectures use a **tiered approach** — preferring the curated, trusted internal source when it has sufficient coverage for a given query, falling back to broader web search only when necessary, and applying additional scrutiny/validation specifically to information retrieved from the less-controlled open web source compared to the trusted internal one.

---

### Q: What is "knowledge base freshness" as an architectural concern specific to agentic RAG systems used for time-sensitive domains, and how would you design the update/ingestion pipeline to minimize the risk of an agent confidently answering with outdated information?

**Answer:**
For domains where underlying information changes relatively frequently (product pricing, policy details, current events), a RAG knowledge base that isn't kept sufficiently current risks the agent **confidently retrieving and presenting outdated information as if it were current** — arguably worse than the agent having no information at all, since retrieved-and-cited information often carries an implicit air of authority/verification that can mislead users into trusting outdated content more than they otherwise would. Architectural mitigations: designing the **ingestion pipeline with an appropriately frequent update cadence** matched to how quickly the specific domain's information actually changes (rather than a generic, one-size-fits-all refresh schedule), **explicitly timestamping indexed content and surfacing that recency information** to the agent (and ideally to the end user, e.g., "according to information last updated on...") so staleness is at least visible rather than silently hidden, and, for especially time-sensitive queries, potentially **routing to a live, real-time source (e.g., a direct API call) rather than a periodically-refreshed static knowledge base** when the specific query type is known to be highly time-sensitive.

---

### Q: What is the difference between "extractive" and "generative" use of retrieved context in an agentic RAG system, and why might an architecture prefer extractive responses (directly quoting/citing retrieved text) for certain high-stakes query types?

**Answer:**
**Generative** use has the model synthesize/paraphrase retrieved information into a new, freely-composed response — natural-sounding and able to combine information from multiple sources coherently, but introduces the risk of the generation step subtly altering, embellishing, or misrepresenting the original source content in the process. **Extractive** use has the system directly surface/quote the relevant retrieved passage(s) with minimal or no paraphrasing, trading some naturalness of response for **higher fidelity guarantee** that what's presented is exactly what the source actually said. For **high-stakes query types** (legal, medical, financial, or compliance-related information, where precise wording can genuinely matter and even subtle paraphrasing drift could change meaning or introduce liability), an architecture might deliberately favor extractive presentation (or at minimum, generative synthesis with very prominent, precise citations directly linking back to exact source passages) over free-form generative summarization, explicitly trading some response fluency for verifiable accuracy.

---

### Q: How would you architect an agentic RAG system to gracefully handle a query where the available knowledge base genuinely doesn't contain relevant information, rather than having the agent generate a plausible-sounding but unsupported answer anyway?

**Answer:**
This requires the architecture to explicitly support and correctly trigger an **"insufficient information" outcome** as a first-class, expected response type, rather than treating "generate some kind of answer" as the only valid outcome for every query. Concretely: the retrieval-evaluation step (discussed in the corrective RAG question) should be able to conclude "the retrieved results are not sufficiently relevant/complete to answer this query" and this conclusion needs to **actually change the downstream generation behavior** — the generation prompt should be explicitly instructed (and ideally verified via evaluation testing, covered in a later topic) to **honestly state that it doesn't have sufficient information to answer**, rather than defaulting to generating a plausible-sounding response anyway when explicitly told the retrieved context is inadequate. This is a genuinely important design point because the default behavior of many LLMs, absent explicit architecture and prompting to prevent it, tends toward confidently answering even when the actual grounding evidence is weak or absent — this needs to be a deliberately engineered behavior, not an assumed default.

---
