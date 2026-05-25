# Praveen Kumar Bommali

AI Architect and Founder building MedhaLink AI.

I design production AI systems across Agentic AI, LLM applications, RAG, enterprise search, tool-use, MCP-style integrations, LLMOps, backend platforms, and cloud delivery. My focus is not just building demos, but turning AI into governed systems: grounded answers, reliable retrieval, reviewable actions, traceable agent runs, cost-aware inference, and services that can be operated in the real world.

## AI architecture focus

- **Agentic AI:** tool calling, multi-step reasoning flows, handoffs, memory, human-in-the-loop approvals, guardrails, audit trails, and agent tracing
- **LLM applications:** prompt orchestration, structured outputs, function calling, provider abstraction, model selection, latency/cost tradeoffs, and fallback paths
- **RAG and enterprise search:** ingestion pipelines, chunking, metadata, hybrid retrieval, reranking, source attribution, citations, permission-aware retrieval, and evaluation loops
- **MCP and tool ecosystems:** connecting models to tools, resources, prompts, internal APIs, databases, email, documents, and collaboration systems through clean integration boundaries
- **LLMOps and evaluation:** prompt/version control, trace analysis, retrieval quality checks, LLM-as-judge evaluation, regression datasets, observability, and production monitoring
- **LLM serving:** API-based LLMs plus open-source serving patterns around vLLM/TGI, batching, KV cache, quantization, LoRA adapters, throughput, latency, and GPU-aware deployment
- **Platform engineering:** FastAPI, Spring Boot, PostgreSQL, Redis, Celery, Qdrant/vector search, Docker, Kubernetes, Helm, GCP/Azure, CI/CD, health checks, metrics, and logs

## Building now: MedhaLink AI

**MedhaLink AI** is my founder work: an enterprise AI platform for governed knowledge search and AI-assisted workflows. The product connects company knowledge, retrieves evidence from the right sources, generates grounded answers, and turns repeatable tasks into controlled actions.

Architecture problems I care about:

- Agentic RAG that can search, reason, use tools, and still show the evidence behind an answer
- Connectors for email, documents, and collaboration tools that separate useful knowledge from noisy, automated, or unsafe content
- AI actions that create drafts, summaries, tasks, and workflow outputs with human review before external delivery
- Workflow runs where scheduling, skipped steps, provider draft creation, delivery, and failure states remain visible
- A lean API runtime with heavier ingestion, embeddings, vector writes, sync jobs, and inference work pushed into workers or dedicated serving layers

## Current AI skill map

**Agentic AI:** agents, tools, handoffs, planning loops, human review, guardrails, memory, state machines, LangGraph-style workflow orchestration  
**LLM engineering:** OpenAI/Anthropic-style APIs, structured outputs, prompt routing, fallback strategies, model/provider abstraction, cost and latency tuning  
**RAG systems:** embeddings, vector search, hybrid retrieval, reranking, GraphRAG concepts, citations, document/email ingestion, chunking strategy, evaluation  
**LLMOps:** tracing, observability, regression evals, prompt/version tracking, LLM-as-judge, failure analysis, safety checks, production monitoring  
**Serving and inference:** vLLM, TGI, batching, KV cache, quantization, LoRA adapters, GPU-aware deployment, API gateway patterns, streaming responses  
**Backend and cloud:** Python, FastAPI, Pydantic, SQLAlchemy, Java, Spring Boot, PostgreSQL, Redis, Celery, Qdrant, Docker, Kubernetes, Helm, GCP, Azure  
**Frontend when needed:** TypeScript, React, Vite, TanStack, Tailwind CSS

## Selected systems

| Project | AI / architecture signal | Stack |
| --- | --- | --- |
| [Brand Asset Studio API](https://github.com/praveenbommalibits/brand-studio-suite-ba) | AI workflow backend for brief parsing, tool routing, generation orchestration, QA, and package delivery | Python, FastAPI |
| [Brand Asset Studio](https://github.com/praveenbommalibits/brand-studio-suite) | Product UI for AI-assisted planning, approvals, generation review, and downloadable outputs | TypeScript, React |
| [LLM Connector Service](https://github.com/praveenbommalibits/llm-connector) | LLM query handling with RAG-style SAP field context retrieval | Java 21, Spring Boot |
| [Medha MoM](https://github.com/praveenbommalibits/medha-mom) | Local meeting intelligence with transcription, speaker identification, suggestions, and structured notes | Python, Whisper |
| [MedhaWhisper](https://github.com/praveenbommalibits/medha-whisper) | Local macOS voice-to-text utility with hotkey capture, Whisper transcription, and auto-typing | Python, macOS |
| [String Reply Service](https://github.com/praveenbommalibits/rest-service) | API engineering fundamentals: compatibility, validation, rule execution, and testability | Java, Spring Boot |

## What I am sharpening next

- Agentic AI systems that combine RAG, tool use, state, approvals, and observable execution
- MCP-style connector layers for enterprise data, tools, and governed AI actions
- Evaluation-first RAG with retrieval tests, trace review, LLM-as-judge checks, and regression datasets
- LLM serving strategy across hosted APIs and self-hosted inference with vLLM/TGI-style deployment patterns
- Production LLMOps: tracing, cost controls, latency budgets, safety review, monitoring, and rollout discipline

## Links

- Website: https://praveenkumarbommali.com
- LinkedIn: https://linkedin.com/in/praveenkumarbommali
- GitHub: https://github.com/praveenbommalibits
