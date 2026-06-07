# Auto-RAG Architecture Blueprints

This folder holds the evolving design of the Enterprise Auto-RAG pipeline. Each version is a snapshot of architectural decisions — read them in order to see what changed and why.

**Current target: [V5](Auto_RAG_Pipeline_V5.puml)** — hexagonal architecture with a rich domain core, ports/adapters, pipes & filters, ACL, domain events, and index versioning.

---

## V2 — Master blueprint

[`Auto_RAG_Pipeline_V2.puml`](Auto_RAG_Pipeline_V2.puml)

The starting point: a monolithic `AppConfig` god-object, protocol-based components (Scraper, Chunker, Embedder, VectorStore, Retriever, Evaluator, Reporter), and a single `AutoRAGPipeline` orchestrator. Retriever is segregated from VectorStore. Metadata on chunks and retrieval results is still an untyped `dict`. Good for proving the pipeline shape; weak on config boundaries, error handling, and provenance.

## V3 — Scoped configs and operational boundaries

[`Auto_RAG_Pipeline_V3.puml`](Auto_RAG_Pipeline_V3.puml)

Replaces `AppConfig` with per-component configs (`ScraperConfig`, `EmbedderConfig`, `VectorStoreConfig`, etc.) so each interface receives only what it needs. Adds async across all pipeline methods, separates `EvaluationPolicy` (pass/fail thresholds) from `EvaluatorConfig` (measurement), introduces `PipelineError` and `ErrorHandler` with circuit breaker, and splits VectorStore vs Retriever mocks in tests. Still uses raw `dict` metadata and batch-oriented scraping.

## V4 — Production-grade ingestion

[`Auto_RAG_Pipeline_V4.puml`](Auto_RAG_Pipeline_V4.puml)

Focuses on making ingestion enterprise-ready: `DocumentMetadata` replaces raw dicts end-to-end, streaming via `stream_documents()` and `Result[T, E]` per file, a full scraper subsystem (file extractors, preprocessor pipeline, extractor registry, state store for dedup/stale cleanup), `ProgressReporter` observability, and bounded concurrency (`asyncio.Semaphore`, encoding fallbacks, size limits). Orchestrator integrates stale-vector deletion before each run. Still a layered “pipeline + protocols” design rather than explicit domain modeling.

## V5 — Hexagonal architecture and rich domain

[`Auto_RAG_Pipeline_V5.puml`](Auto_RAG_Pipeline_V5.puml)

Structural refactor: **Domain Core** (value objects, aggregates, domain events, no infra imports) sits at the center; **Ports** define boundaries; **Application Layer** use cases orchestrate via pipes & filters; **Adapters** implement Pinecone, OpenAI, local filesystem, SQLite, etc. Introduces `SourceDocument` aggregate, `IndexVersion` for re-indexing when embedding/chunking config changes, anti-corruption layer (ACL) between domain and external APIs, and domain events (`DocumentIngested`, `DocumentMarkedStale`, …) for observability and side effects. This is the blueprint we implement against.

---

## Viewing the diagrams

Open any `.puml` file in an IDE with PlantUML support, or render via [PlantUML online](https://www.plumtext.com/plantuml/uml/).
