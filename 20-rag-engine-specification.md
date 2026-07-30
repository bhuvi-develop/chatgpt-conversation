# OCIF AI Platform --- Phase 1.5

## Complete Software Architecture Specification --- RAG Engine

**Status:** Architecture / permanent knowledge specification only. No
implementation code (no Python, FastAPI, React, or SQL). Builds on
`01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`,
`04-api-specification.md`, `05-frontend-ux-spec.md`, the OCIF Layer
Specifications (1-8), and Engines 14--19
(`19-grounding-engine-specification.md`).

------------------------------------------------------------------------

## 1. Overview

The **RAG (Retrieval-Augmented Generation) Engine** is the platform's
dedicated semantic search and retrieval subsystem. It executes
high-performance vector and hybrid searches across the platform's
embedded knowledge stores (`ProjectContextChunk` and `KnowledgeChunk`).
It isolates the mathematical complexity of embedding generation, query
expansion, and vector similarity from the rest of the application.

## 2. Purpose

To retrieve the most semantically relevant text chunks based on a given
query, transforming raw natural language into optimized vectors,
executing similarity algorithms against PostgreSQL (`pgvector`), and
returning mathematically ranked evidence to the Grounding Engine.

## 3. Objectives

-   Decouple vector search logic from the Grounding and Synthesis
    layers.
-   Execute query expansion and rewriting to overcome vocabulary
    mismatches between user queries and technical documents.
-   Perform hybrid search (Dense Vector + Sparse Keyword) to maximize
    recall.
-   Apply cross-encoder re-ranking to maximize precision of the Top-K
    results.
-   Enforce strict metadata pre-filtering (by tenant, session, and
    active context) to minimize the search space and guarantee data
    isolation.

## 4. Business Need

Users frequently ask questions using terminology that differs from the
underlying uploaded source code or enterprise standards (e.g., asking
about "login" when the code uses "OAuth2 token exchange"). Standard
keyword search fails here. The business requires a highly accurate
semantic retrieval engine that can bridge this vocabulary gap and pull
the exact technical specifications needed to ground the AI, without
retrieving irrelevant or cross-tenant data.

## 5. Problem Statement

**Given** massive volumes of chunked technical data spread across
project uploads and enterprise knowledge bases, **the RAG Engine must**
efficiently locate and rank the most relevant information for any given
prompt, **without** assembling the final LLM context payload, and
**without** applying business-logic priority tiers (which is the
Grounding Engine's job).

## 6. Responsibilities

  -----------------------------------------------------------------------
  Responsibility                      Excluded Responsibilities (Handled
                                      Elsewhere)
  ----------------------------------- -----------------------------------
  Query expansion and rewriting.      Assembling the final context
                                      payload (Grounding Engine).

  Generating query embeddings.        Resolving cross-tier data conflicts
                                      (Grounding Engine).

  Executing pgvector hybrid searches. Chunking the original documents
                                      (Normalization Layer).

  Cross-encoder re-ranking of         Determining the active project
  retrieved chunks.                   (Project Context Engine).
  -----------------------------------------------------------------------

## 7. Engine Architecture

The RAG Engine follows the Clean Architecture pattern. It acts as an
Infrastructure/Application boundary layer. It receives a
`RetrievalRequest` from the Grounding Engine, processes the query,
interfaces with an external `EmbeddingProvider`, executes the query
against the database `VectorSearchPort`, and returns a
`ScoredChunkList`.

## 8. Internal Modules

-   **`QueryProcessor`**: Handles query rewriting, expansion, and
    keyword extraction.
-   **`EmbeddingService`**: Adapter for external embedding APIs (e.g.,
    OpenAI, Cohere).
-   **`VectorSearchClient`**: Constructs and executes the `pgvector` SQL
    queries with metadata filters.
-   **`HybridSearchMerger`**: Combines vector (cosine similarity) and
    keyword (BM25/pg_trgm) results using Reciprocal Rank Fusion (RRF).
-   **`ReRanker`**: Applies a cross-encoder model to re-score the merged
    results for high precision.

## 9. Retrieval Pipeline

1.  **Request:** Receives query string and metadata boundaries (e.g.,
    `project_context_id`, `industry_tags`).
2.  **Expansion:** `QueryProcessor` rewrites the query into multiple
    semantic variants.
3.  **Embedding:** `EmbeddingService` converts the rewritten queries
    into vector arrays.
4.  **Search:** `VectorSearchClient` executes HNSW/IVFFlat
    nearest-neighbor searches in Postgres, filtered by the metadata
    boundaries.
5.  **Merge:** `HybridSearchMerger` blends vector hits with keyword
    hits.
6.  **Re-rank:** `ReRanker` orders the final candidate list.
7.  **Return:** Returns the Top-K `ScoredChunk` objects to the Grounding
    Engine.

## 10. Query Processing

Raw user intent (from Layer 1) or internal engine prompts (from Layer 5)
are often too brief for optimal vector search. The `QueryProcessor`
normalizes the text, removes stop words for the keyword track, and
prepares the input for the expansion phase.

## 11. Query Expansion

Uses a lightweight LLM call to generate synonyms and related concepts.
For example, if the query is "database scalability", the expansion might
append "sharding, read replicas, connection pooling, horizontal
scaling".

## 12. Query Rewriting

Transforms conversational queries ("How do I fix the pump?") into
declarative retrieval targets ("Troubleshooting steps for industrial
centrifugal pump failure"). This significantly improves cosine
similarity matches against technical manuals.

## 13. Embedding Generation

The `EmbeddingService` converts the rewritten string into a
high-dimensional dense vector (e.g., 1536 dimensions for standard
models). This service caches identical queries (using Redis) to avoid
redundant API calls and reduce latency.

## 14. Embedding Providers

The engine uses the Strategy Pattern to support multiple embedding
models:

-   `OpenAIEmbeddingAdapter` (`text-embedding-3-large`)
-   `CohereEmbeddingAdapter` (`embed-english-v3.0`)
-   `LocalEmbeddingAdapter` (HuggingFace `BGE-m3` for air-gapped/local
    deployments).

## 15. Vector Storage

All vectors reside in PostgreSQL using the `pgvector` extension. The RAG
Engine does not manage the schema (Database Design phase handled this),
but it heavily relies on the HNSW (Hierarchical Navigable Small World)
indexes placed on the vector columns for sub-millisecond retrieval.

## 16. Chunking Strategy

*Boundary Note:* The RAG Engine **does not chunk documents**. Layer 3
(Normalization) and the Knowledge Engine handle chunking. The RAG Engine
assumes chunks are perfectly sized (e.g., 500 tokens with 50-token
overlap) and pre-populated in the database.

## 17. Chunk Metadata

Every retrieved chunk includes its `source_uuid`, `page_number`,
`file_name`, and `chunk_index`. The RAG Engine passes this metadata
blindly back to the Grounding Engine, which uses it for citation
mapping.

## 18. Semantic Search

Executes the primary vector search using cosine distance (`<=>` in
pgvector). The search is strictly bounded by the `WHERE` clauses mapping
to the tenant and session.

## 19. Hybrid Search

Pure vector search sometimes misses exact part numbers or specific
variable names. The RAG Engine executes a parallel keyword search (using
Postgres Full Text Search or `pg_trgm`) and merges the results with the
vector search using Reciprocal Rank Fusion (RRF).

## 20. Metadata Filtering

**Critical Security Layer:** The Vector Search must dynamically
construct queries that append `WHERE session_id = ? AND org_id = ?`. If
searching the Knowledge Base, it appends
`WHERE industry_tags ?| array[...]`.

## 21. Similarity Ranking

Initial results from pgvector are returned with a raw cosine distance
score. The `HybridSearchMerger` normalizes these scores (0.0 to 1.0)
before passing them to the Re-ranker.

## 22. Re-ranking

Because vector dot-products are approximate representations of
relevance, the engine takes the Top-N (e.g., 50) chunks and passes them
through a Cross-Encoder (e.g., Cohere Rerank API or local
`bge-reranker`). The Cross-Encoder scores the exact query against the
exact chunk text, providing a highly precise final score.

## 23. Top-K Selection

After re-ranking, the engine truncates the list to the `Top-K` requested
by the Grounding Engine (e.g., K=15) to preserve token budget, returning
only the most mathematically relevant facts.

## 24. Retrieval Optimization

-   `HNSW` index configuration for scale.
-   Redis caching for identical embedding requests.
-   Asynchronous parallel execution of the Dense (Vector) and Sparse
    (Keyword) database queries.

## 25. Project Context Engine Integration

The RAG Engine requires the Project Context Engine to provide the
`active_project_context_id`. This ID is used as a hard `WHERE` clause to
ensure the vector search does not pull chunks from the user's other
uploaded projects.

## 26. Knowledge Engine Integration

The RAG Engine queries the `KnowledgeChunk` table. It relies on the
Knowledge Engine to have properly tagged and indexed these vectors,
using the active project's industry tags to filter the vector space.

## 27. Grounding Engine Integration

The RAG Engine is a direct dependency of the Grounding Engine. The RAG
Engine provides the raw, scored, relevant chunks. The Grounding Engine
applies the business rules (Tier 1 vs Tier 3) and cuts off chunks that
exceed the LLM token budget.

## 28. Documentation Engine Integration

Indirect. The Documentation Engine asks the Grounding Engine for facts;
the Grounding Engine delegates the semantic search to the RAG Engine.

## 29. Diagram Engine Integration

Indirect. The RAG Engine retrieves structural definitions (e.g., "The
API talks to Postgres via SQLAlchemy") which ultimately inform the
Diagram Engine's node generation.

## 30. Image Engine Integration

Indirect. The RAG Engine may retrieve descriptive styling guidelines
from the Knowledge Base, which the Grounding Engine passes to the Image
Engine.

## 31. Language Engine Integration

The `QueryProcessor` detects if the query is in a regional language
(e.g., Tamil). It translates the query to English *before* embedding,
ensuring it accurately matches the predominantly English vector space of
technical standards.

## 32. Validation Rules

-   Requests must include valid authentication and context boundary IDs.
-   Top-K must not exceed a platform-defined hard limit (e.g., max 50)
    to prevent memory bloat.

## 33. Error Handling

-   `400 Bad Request`: Missing mandatory search filters (e.g.,
    attempting a global search without an `org_id`).
-   `422 Unprocessable Entity`: The embedding provider rejected the
    string (e.g., string exceeds embedding token limit).
-   `502 Bad Gateway`: Timeout from the external Re-ranker or Embedding
    API.

## 34. Retry Strategy

Embedding and Re-ranking API calls are wrapped in a resilient circuit
breaker. Transients trigger 3 immediate retries with exponential
backoff. If the Re-ranker fails entirely, the engine falls back to the
raw `pgvector` cosine similarity scores to guarantee a response.

## 35. Logging

Logs `query_intent`, `embedding_latency_ms`, `db_search_latency_ms`,
`rerank_latency_ms`, and the `chunk_ids` returned. Never logs the raw
text of the user's query or the retrieved chunks to protect proprietary
IP.

## 36. Performance

-   Embedding generation: \< 150ms.
-   Vector + Keyword hybrid DB search (pgvector): \< 50ms.
-   Cross-encoder Re-ranking (Top 50): \< 250ms.
-   Total retrieval SLA: \< 500ms.

## 37. Security

The engine is completely blind to data it shouldn't see. The Repository
layer injects Tenant (`org_id`) and Session (`session_id`) constraints
into the SQLAlchemy/SQL layer before `pgvector` executes the distance
function, guaranteeing cryptographically safe data isolation.

## 38. Database Mapping

-   **Reads from:** `project_ctx.ProjectContextChunk`,
    `knowledge.KnowledgeChunk`.
-   **Utilizes:** `pgvector` (`vector` type columns), `pg_trgm` (for
    keyword fallback).

## 39. REST API Mapping

-   `POST /api/v1/internal/rag/search` (Internal microservice port used
    by Grounding/Synthesis).

## 40. Folder Structure

``` text
backend/app/engines/rag/
├── __init__.py
├── query_processor.py
├── embedding_service.py
├── vector_search_client.py
├── hybrid_merger.py
├── reranker.py
└── exceptions.py
```

## 41. Mermaid Architecture Diagram

``` mermaid
flowchart TB
    subgraph GE["Grounding Engine"]
        REQ[Request Context]
    end

    subgraph RAG["RAG Engine"]
        QP[QueryProcessor]
        ES[EmbeddingService]
        VSC[VectorSearchClient]
        HM[HybridSearchMerger]
        RR[ReRanker]
    end

    subgraph External["External APIs"]
        EMB[Embedding Model]
        XR[Cross-Encoder]
    end

    subgraph DB["Database"]
        PGV[(pgvector / Postgres)]
    end

    REQ --> QP
    QP --> ES
    ES <--> EMB
    ES --> VSC
    VSC <--> PGV
    VSC --> HM
    HM --> RR
    RR <--> XR
    RR -->|Top-K Scored Chunks| REQ
```

## 42. Mermaid Sequence Diagram

``` mermaid
sequenceDiagram
    participant GE as Grounding Engine
    participant QP as QueryProcessor
    participant ES as EmbeddingService
    participant DB as pgvector
    participant RR as ReRanker
    
    GE->>QP: search("auth flow", project_id=12)
    QP->>QP: Rewrite: "OAuth2 authentication JWT flow"
    QP->>ES: get_embedding(rewritten_query)
    ES-->>QP: [0.014, -0.055, ...] (Vector)
    QP->>DB: Cosine search (<=>) + Keyword search WHERE project_id=12
    DB-->>QP: Top 50 mixed chunks
    QP->>RR: re_rank(query, 50 chunks)
    RR-->>QP: Top 15 re-ordered chunks
    QP-->>GE: ScoredChunkList
```

## 43. Mermaid Component Diagram

``` mermaid
componentDiagram
    component "RAG Engine" {
        [QueryProcessor]
        [EmbeddingService]
        [VectorSearchClient]
        [HybridSearchMerger]
        [ReRanker]
    }
    
    [Grounding Engine] --> [QueryProcessor]
    [QueryProcessor] --> [EmbeddingService]
    [QueryProcessor] --> [VectorSearchClient]
    [VectorSearchClient] --> [HybridSearchMerger]
    [HybridSearchMerger] --> [ReRanker]
```

## 44. Mermaid Deployment Diagram

``` mermaid
flowchart LR
    subgraph AppServer["Application Container (FastAPI)"]
        RAG[RAG Engine]
    end

    subgraph CacheTier["Redis"]
        EMB_CACHE[Embedding Cache]
    end

    subgraph DatabaseTier["PostgreSQL"]
        VEC_IDX[HNSW Vector Index]
    end

    RAG <--> EMB_CACHE
    RAG <--> VEC_IDX
```

## 45. Industrial Examples

The RAG Engine bridges the gap between human questions and dense
industrial telemetry or code.

## 46. Water Pump Example

**Scenario:** User asks, "Why is the pump vibrating?" **RAG Action:**
The `QueryProcessor` expands "vibrating" to "cavitation, resonance,
bearing wear". The `VectorSearchClient` finds a chunk in the uploaded
maintenance PDF discussing "cavitation due to low NPSH". The `ReRanker`
scores this as 0.98 relevance and returns it to the Grounding Engine.

## 47. Smart Building Example

**Scenario:** User asks, "Find the lighting controller script." **RAG
Action:** A pure vector search might struggle with finding exact code
filenames. The `HybridSearchMerger` combines the vector search for
"lighting automation" with a sparse keyword search for
`lighting_controller.py`, successfully retrieving the exact chunk of
uploaded source code.

## 48. Attendance System Example

**Scenario:** User asks, "How does biometric data get stored?" **RAG
Action:** The engine applies the active project ID filter. It searches
`ProjectContextChunk` for the user's database schema, and simultaneously
searches `KnowledgeChunk` (filtered by the "hr-tech" industry tag) for
GDPR compliance SOPs regarding biometric hashing, returning both sets of
vectors.

## 49. Future Extensions

-   **GraphRAG:** Integrating a Knowledge Graph to traverse
    relationships (e.g., retrieving Chunk B simply because it is
    structurally linked to Chunk A, even if Chunk B's cosine similarity
    to the query is low).
-   **Multi-Modal Retrieval:** Upgrading the Embedding Service to
    support CLIP/Vision models, allowing the RAG Engine to retrieve
    image chunks or diagram snippets alongside text.

## 50. Best Practices

-   **Always Filter First:** Apply relational metadata filters
    (`WHERE session_id = ?`) *before* executing the vector distance
    function to radically reduce query latency and guarantee security.
-   **Hybrid is Mandatory:** Never rely solely on dense vectors for
    technical architecture. Code bases require sparse/keyword fallback
    to match exact class names and UUIDs.
-   **Cache Embeddings:** Identical internal queries generated by the
    pipeline should instantly hit a Redis cache rather than repeatedly
    calling the OpenAI/Cohere API.

## 51. Common Mistakes

-   **Violating Boundaries:** Letting the RAG Engine decide which chunks
    are "allowed" into the prompt. The RAG Engine scores purely on
    semantic math; the Grounding Engine applies the enterprise rules.
-   **Skipping Re-ranking:** Returning the raw pgvector Top-K. Vector
    math often surfaces chunks that are tangentially related. A
    Cross-Encoder is required to prune false positives.
-   **Over-fetching:** Requesting a K value of 100+, which bottlenecks
    the Re-ranker API and spikes latency beyond acceptable interactive
    limits.

## 52. Interview Questions

1.  **Why do we use Reciprocal Rank Fusion (RRF)?** *Answer:* Because
    vector similarity scores (dense) and keyword BM25 scores (sparse)
    are on completely different mathematical scales. RRF provides a
    stable algorithm to combine their rankings without needing to
    arbitrarily weight the raw scores.
2.  **How does the RAG Engine ensure cross-tenant data doesn't leak
    during a vector search?** *Answer:* Data isolation is enforced via
    hard `WHERE org_id = ?` clauses appended to every SQL query executed
    by the `VectorSearchClient`. `pgvector` respects standard Postgres
    row-level security and indexing constraints.

## 53. Summary

The RAG Engine provides the high-performance mathematical muscle of the
OCIF AI Platform's retrieval capabilities. By executing expanded queries
across a hybrid vector-and-keyword search space, strictly bounded by
tenant metadata, and refined through cross-encoder re-ranking, it
guarantees that the Grounding Engine receives the most precise,
semantically relevant artifacts available to inform the AI's generation
tasks.
