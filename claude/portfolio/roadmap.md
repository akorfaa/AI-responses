# Tier 1 — Junior (prove: can build & ship a working thing)
1. CRUD API with auth — FastAPI + PostgreSQL service (e.g. a claims-intake or policy-quote API), JWT auth, input validation, basic tests. Deployed somewhere free (Render/Railway).
2. Data cleaning + dashboard — take a messy public insurance/finance dataset, clean it, build a small analysis + dashboard (Streamlit). Proves the data-science-to-product bridge.
3. Simple LLM-powered tool — a single-purpose app calling an LLM API (e.g. document summarizer or FAQ bot) with a real UI, not just a script. Proves you can integrate AI into a product, not just call an API in a notebook.

# Tier 2 — Mid (prove: can design, not just implement)
4. RAG system over a real document set — e.g. insurance policy documents or regulatory text. Chunking strategy, embeddings, retrieval evaluation (not just “it works” — show you measured it).
5. Multi-step agent with tool use — a Bedrock/LangChain agent that plans and calls 2–3 tools (e.g. a claims-triage assistant that looks up policy data, calculates risk score, drafts a response). Include guardrails and error handling.
6. Backend service with real architecture — extend Project 1 into a multi-service system (e.g. API + background worker + queue), with tests, logging, and a basic CI pipeline (GitHub Actions).
7. ML model in production shape — take a model you’d build for actuarial/underwriting-style prediction (e.g. loan default, claim severity), wrap it behind an API, add monitoring for drift/performance, containerize with Docker.

# Tier 3 — Senior / Expert (prove: can own a system end-to-end and make tradeoffs)
8. Multi-agent orchestration system — several specialized agents coordinating (e.g. an underwriting assistant: intake agent → risk-analysis agent → policy-recommendation agent), with a supervisor pattern, observability/tracing, and documented failure-mode handling.
9. Full insurtech-flavored capstone — an end-to-end product: FastAPI backend, Postgres, an agentic AI core, a real frontend, deployed to the cloud (AWS, tying into your Bedrock/agent-nanodegree work), with cost-and-latency tradeoff notes and a security review.
10. Evaluation & reliability project — build an eval harness/test suite for one of your earlier agentic systems (hallucination checks, regression tests, red-teaming for prompt injection). This is the project that most clearly signals “senior” — most junior/mid candidates never do this.
11. System design write-up + reference implementation — pick a hard, realistic scenario (“design an AI claims-processing platform for 1M policies”) — produce an architecture doc (tradeoffs, scaling, cost) plus a scoped-down working implementation of the core piece. This is your interview-ready, portfolio-centerpiece project.
12. Open-source or teaching artifact — contribute a meaningful PR to an agentic-AI OSS project, or publish a well-documented technical guide/library from your own work. Proves communication and community credibility, which senior roles weight heavily.
