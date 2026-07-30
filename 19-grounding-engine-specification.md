# OCIF AI Platform --- Phase 1.5

## Complete Software Architecture Specification --- Grounding Engine

**Status:** Architecture / permanent knowledge specification only. No
implementation code (no Python, FastAPI, React, or SQL). Builds on
`01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`,
`04-api-specification.md`, `05-frontend-ux-spec.md`, the OCIF Layer
Specifications (1-8), `14`--`16` (Generative Engines),
`17-project-context-engine-specification.md`, and
`18-knowledge-engine-specification.md`.

------------------------------------------------------------------------

## 1. Overview

The **Grounding Engine** is the platform's absolute authority on truth
and traceability. Acting as the strict gatekeeper between raw data
retrieval and LLM context ingestion, it ensures that every prompt sent
to the generative engines (Documentation, Diagram, Image, Cognition) is
populated *only* with validated, ranked, and conflict-resolved
enterprise facts.

## 2. Purpose

To prevent AI hallucinations by enforcing a deterministic, policy-driven
assembly of context. The engine structures multiple data streams into a
cohesive, token-optimized payload, forcing the downstream LLM to reason
exclusively from verified reality rather than its internal parametric
memory.

## 3. Objectives

-   Assemble raw retrieved chunks from the RAG Engine, Project Context
    Engine, and Knowledge Engine into a unified context payload.
-   Prioritize evidence strictly by origin tier to resolve cross-source
    contradictions.
-   Optimize token budgets to prevent LLM context-window overflow and
    degradation.
-   Inject cryptographic or UUID-based citation markers to guarantee
    origin traceability.
-   Reject ungrounded queries or force "insufficient evidence" fallback
    behaviors.

## 4. Business Need

Enterprise AI cannot afford probabilistic guessing. In critical
industrial, financial, or architectural applications, an AI
hallucinating a non-existent API endpoint or a false safety standard is
a catastrophic failure. The business requires an irrefutable mechanism
to ensure every generated artifact is strictly traceable to an approved
document or explicit user upload.

## 5. Problem Statement

**Given** that generative LLMs naturally hallucinate when faced with
sparse, contradictory, or overly broad context windows, **the Grounding
Engine must** filter, rank, and explicitly map retrieved evidence into a
highly structured payload, **without** performing the semantic search
itself, and **without** bleeding generic internet knowledge into
specialized enterprise solutions.

## 6. Responsibilities

  -----------------------------------------------------------------------
  Responsibility                      Excluded Responsibilities (Handled
                                      Elsewhere)
  ----------------------------------- -----------------------------------
  Assembling the final Grounded       Executing vector similarity search
  Context Payload.                    (RAG Engine).

  Enforcing strict Priority Tiers for Generating final answers
  conflict resolution.                (Cognition/Synthesis Layers).

  Token budgeting and context window  Managing project state (Project
  truncation.                         Context Engine).

  Injecting citation markers          Managing enterprise standard files
  `[src-uuid]` for traceability.      (Knowledge Engine).
  -----------------------------------------------------------------------

## 7. Engine Architecture

The Grounding Engine is a pure Application Layer orchestrator. It
receives a `GroundingRequest` from the Synthesis/Cognition layers,
interfaces via service ports with the retrieval engines to collect data,
applies its business logic (ranking, trimming, mapping), and returns a
validated `GroundedContextBundle`.

## 8. Internal Modules

-   **`ContextAssembler`**: Merges structured metadata and unstructured
    semantic chunks.
-   **`EvidenceRanker`**: Scores combined results based on priority
    tier, relevance, and recency.
-   **`ConflictResolver`**: Deterministically prunes lower-tier facts
    that contradict higher-tier facts.
-   **`TokenOptimizer`**: Enforces strict LLM context limits using token
    estimation heuristics (e.g., `tiktoken`).
-   **`CitationMapper`**: Appends invisible or visible source references
    to each assembled chunk.

## 9. Grounding Pipeline

1.  **Request:** Synthesis layer requests context for a specific intent.
2.  **Collection:** Engine queries RAG, Project Context, and Knowledge
    engines.
3.  **Ranking & Resolution:** Applies tier logic; handles conflicts.
4.  **Optimization:** Fits the highest-priority data into the token
    budget.
5.  **Formatting:** Wraps data in strict XML tags (e.g.,
    `<verified_facts>`).
6.  **Delivery:** Returns the `GroundedContextBundle` to the caller.

## 10. Context Assembly Pipeline

The engine creates a structured hierarchical payload. Instead of dumping
raw text into the LLM, it formats the context as:

1.  `[Primary Directives]`
2.  `[Tier 1: Uploaded Project Files]`
3.  `[Tier 2: Structured Project Context]`
4.  `[Tier 3: Enterprise Knowledge Standards]`
5.  `[Tier 4: General RAG Chunks]`

## 11. Evidence Collection

The Grounding Engine acts as the client to internal retrieval systems.
It dispatches parallel requests:

-   To the **RAG Engine**: "Give me the top 20 semantic chunks for this
    query."
-   To the **Project Context Engine**: "Give me the JSONB architecture
    mapping for the active project."
-   To the **Knowledge Engine**: "Give me the active SOPs matching this
    industry tag."

## 12. Evidence Validation

Before assembly, the engine validates that every piece of evidence
possesses a valid `source_uuid`, a non-expired `is_active` flag, and
matches the strict `org_id` / `session_id` of the requesting user. Any
orphaned or cross-tenant chunk is instantly discarded.

## 13. Source Prioritization

The Engine enforces the OCIF Grounding Priority Chain:

-   **Priority 1 (Absolute Truth):** Explicit user-uploaded source
    code/files in the current session.
-   **Priority 2 (Derived Truth):** The structured classifications in
    the Project Context Engine.
-   **Priority 3 (Enterprise Truth):** Approved standards from the
    Knowledge Engine.
-   **Priority 4 (Historical Truth):** Past conversational RAG chunks.

## 14. Conflict Resolution

If the user uploads a file specifying `PostgreSQL` (Priority 1), but the
enterprise Knowledge Base (Priority 3) mandates `Oracle`, the
`ConflictResolver` ensures Priority 1 overrides Priority 3 for the
context of the user's specific project generation, while explicitly
flagging the deviation for the Cognition layer to note.

## 15. Context Ranking

Within a single tier, evidence is ranked by the semantic
`relevance_score` provided by the RAG Engine, multiplied by a time-decay
factor (newer chunks are weighted slightly higher than older chunks from
the same session).

## 16. Context Window Optimization

Generative models degrade in adherence when the context window is fully
saturated (the "Lost in the Middle" phenomenon). The Grounding Engine
targets a maximum fill rate (e.g., 80% of the model's total context
limit) and strategically places the highest-priority evidence at the
very beginning and very end of the context prompt.

## 17. Token Budget Optimization

The `TokenOptimizer` pre-calculates the payload size. If the collected
evidence exceeds the budget (e.g., 60,000 tokens), it aggressively
truncates from the bottom up (dropping Priority 4 entirely, then
trimming the lowest-relevance chunks of Priority 3) until the budget is
met. Priority 1 data is preserved at all costs.

## 18. Confidence Scoring

The engine calculates an aggregate `grounding_confidence_score` (0.0 to
1.0) based on the density and relevance of the retrieved evidence. If
this score falls below the platform's strict threshold (e.g., 0.4), it
flags the payload with `insufficient_evidence = true`, triggering Layer
8 to output a safe refusal rather than hallucinate.

## 19. Traceability

Every token of assembled context is tracked. The `GroundedContextBundle`
maintains a programmatic dictionary mapping every text block to its
originating `KnowledgeDocument` or `ProjectSourceFile`.

## 20. Citation Management

The `CitationMapper` dynamically rewrites the text chunks being fed to
the LLM to include inline markers (e.g., `[src-1234]`). The system
prompt instructs the LLM to append these markers whenever it uses a
specific fact, allowing the Frontend UI to render clickable reference
links.

## 21. Hallucination Prevention

By wrapping the assembled data in strict XML boundaries
(`<grounding_material> ... </grounding_material>`) and appending hard
LLM instructions ("Do not use external knowledge. If the answer is not
in the XML, state 'Data Unavailable'"), the engine mechanically
neutralizes the LLM's propensity to invent facts.

## 22. Grounding Policies

Policies define how strict the engine must be.

-   **Strict Mode:** Requires \>= 1 Priority 1 or 2 hit. (Fails fast if
    zero project data exists).
-   **Advisory Mode:** Allows generation from Priority 3 (Knowledge
    Base) alone if the user is asking general enterprise questions.

## 23. Project Context Engine Integration

Queries the active context to retrieve the current architectural state,
injecting it as unassailable Priority 2 facts.

## 24. Knowledge Engine Integration

Fetches relevant standard operating procedures, mapping them as Priority
3 constraints that the generated output must comply with.

## 25. RAG Engine Integration

Relies on the RAG Engine for raw vector similarity search. The RAG
Engine does the math; the Grounding Engine applies the business rules to
the results.

## 26. Documentation Engine Integration

Provides the Documentation Engine with highly structured,
section-specific grounded bundles, ensuring that (for example) the
Security section is generated using *only* security-related evidence.

## 27. Diagram Engine Integration

Filters out narrative fluff and provides the Diagram Engine solely with
explicitly detected entities, modules, and API relationships to
guarantee accurate node generation.

## 28. Image Engine Integration

Supplies the explicit industrial context (e.g., "Industry: Maritime
Shipping") so the Image Engine does not hallucinate generic corporate
backgrounds for a shipyard architecture.

## 29. Language Engine Integration

Provides the accepted enterprise glossary (Priority 3) to ensure
technical terms are correctly transliterated during Tanglish or regional
language rendering.

## 30. Validation Rules

-   Must never mix data from different `org_id` values.
-   Token count must not exceed the configured LLM provider's context
    maximum minus the output reserve.
-   Returned context bundles must always include the traceability map.

## 31. Error Handling

-   `413 Payload Too Large`: Thrown if Priority 1 data alone exceeds the
    maximum token window.
-   `422 Unprocessable Entity`: Thrown if the required data structures
    from the RAG engine are malformed.
-   `404 Not Found`: Thrown if a requested explicit source UUID has been
    soft-deleted.

## 32. Retry Strategy

Internal timeouts when calling the RAG or Knowledge Engines trigger an
immediate fast-retry (up to 3 attempts with 200ms backoff). If
unresolved, the engine degrades gracefully by returning whatever valid
context has been successfully assembled, flagging the missing tier.

## 33. Logging

Logs the `request_id`, assembled token count, tier distribution
percentages (e.g., 50% Tier 1, 30% Tier 3), and dropped chunk counts for
observability. Never logs the raw proprietary text itself.

## 34. Performance

-   Context assembly, ranking, and token optimization must execute in \<
    150ms.
-   Token counting (`tiktoken` heuristic) must be heavily optimized or
    cached to avoid CPU bottlenecks.

## 35. Security

Acts as the final data-loss prevention (DLP) safeguard before data
leaves the internal network to hit the Claude API. It enforces a strict,
last-pass validation of tenant isolation on every chunk.

## 36. Database Mapping

The engine is primarily stateless, processing data dynamically. However,
it writes to `generation.GroundingAuditLog` to durably record which
source IDs were used to assemble the context for a specific
`GenerationSession`, guaranteeing historical auditability.

## 37. REST API Mapping

-   `POST /api/v1/internal/grounding/assemble` (Internal microservice
    port)
-   `GET /api/v1/grounding/audit/{session_id}` (For Frontend citation
    resolution)

## 38. Folder Structure

``` text
backend/app/engines/grounding/
├── __init__.py
├── context_assembler.py
├── evidence_ranker.py
├── conflict_resolver.py
├── token_optimizer.py
├── citation_mapper.py
└── exceptions.py
```

## 39. Mermaid Architecture Diagram

``` mermaid
flowchart TB
    subgraph Downstream["Consuming Engines"]
        SYN[Layer 5: Synthesis]
        COG[Layer 6: Cognition]
    end

    subgraph GE["Grounding Engine"]
        CA[ContextAssembler]
        ER[EvidenceRanker]
        CR[ConflictResolver]
        TO[TokenOptimizer]
        CM[CitationMapper]
    end

    subgraph DataSources["Truth Sources"]
        PCE[Project Context Engine]
        KE[Knowledge Engine]
        RAG[RAG Engine]
    end

    SYN -->|Request Context| GE
    CA -->|Fetch| PCE & KE & RAG
    PCE & KE & RAG --> CA
    CA --> ER --> CR --> TO --> CM
    CM -->|GroundedContextBundle| SYN
```

## 40. Mermaid Sequence Diagram

``` mermaid
sequenceDiagram
    participant Cog as Cognition Layer
    participant GE as Grounding Engine
    participant RAG as RAG Engine
    participant PCE as Project Context Engine
    participant Opt as TokenOptimizer

    Cog->>GE: request_grounded_context(intent)
    par Collect Evidence
        GE->>RAG: fetch_vector_chunks(query)
        GE->>PCE: fetch_active_project_state()
    end
    RAG-->>GE: 50 chunks
    PCE-->>GE: JSON Context
    GE->>GE: Rank by Priority (1->4)
    GE->>GE: Resolve cross-tier conflicts
    GE->>Opt: truncate_to_budget(chunks, 80000)
    Opt-->>GE: Validated, trimmed list
    GE->>GE: Inject Citation Tags [src-id]
    GE-->>Cog: GroundedContextBundle (XML Formatted)
```

## 41. Mermaid Component Diagram

``` mermaid
componentDiagram
    component "Grounding Engine" {
        [ContextAssembler]
        [EvidenceRanker]
        [TokenOptimizer]
        [CitationMapper]
    }
    
    [Synthesis Layer] --> [ContextAssembler]
    [ContextAssembler] --> [EvidenceRanker] : Raw Evidence
    [EvidenceRanker] --> [TokenOptimizer] : Ranked Tiers
    [TokenOptimizer] --> [CitationMapper] : Budgeted List
    [CitationMapper] ..> [GroundingAuditLog] : Persist Trace
```

## 42. Mermaid Deployment Diagram

``` mermaid
flowchart LR
    subgraph AppServer["Application Container (FastAPI)"]
        GE[Grounding Engine]
    end

    subgraph InternalServices["In-Memory / Module Calls"]
        RAG[RAG Engine]
        PCE[Project Context Engine]
    end

    subgraph DatabaseTier["PostgreSQL"]
        AUDIT[GroundingAuditLog]
    end

    GE <--> RAG
    GE <--> PCE
    GE --> AUDIT
```

## 43. Industrial Examples

The Grounding Engine ensures that AI responses are strictly bound to
industrial reality, refusing to interpolate outside the assembled
evidence.

## 44. Water Pump Example

**Scenario:** A user asks, "What is the maximum pressure for the intake
valve?" **Grounding Action:** The engine retrieves a chunk from the
user's uploaded spec sheet (Priority 1) stating 150 PSI, and a general
RAG chunk stating 200 PSI. The `ConflictResolver` prunes the 200 PSI
chunk. The context sent to the LLM explicitly forces the answer to 150
PSI, citing `[src-upload-01]`.

## 45. Smart Building Example

**Scenario:** Generating the Layer 3 Database schema for an HVAC system.
**Grounding Action:** The `ContextAssembler` fetches the user's IoT
device list (Priority 1) and the Enterprise Data Retention Policy
(Priority 3). The `TokenOptimizer` ensures both fit. The LLM generates a
database schema that perfectly matches the user's hardware while
enforcing the enterprise's 90-day cold-storage rule.

## 46. Attendance System Example

**Scenario:** A user asks a generic question: "How should I build an
attendance app?" **Grounding Action:** No Priority 1 or 2 data exists
(no project uploaded). The `EvidenceRanker` sees only Priority 4 general
data. The `ConfidenceScoring` module flags this as ungrounded. Layer 8
renders a response: "Please upload your project specifications first so
I can provide a grounded architecture."

## 47. Future Extensions

-   **Graph-Based Grounding:** Integrating with a Knowledge Graph to
    dynamically pull adjacent architectural nodes (e.g., if a chunk
    mentions "Kafka", automatically retrieve the enterprise "Kafka
    Security Standard" chunk).
-   **Dynamic Token Scaling:** Auto-adjusting the token budget based on
    the specific LLM model selected for the generation task (e.g.,
    expanding for Claude 3 Opus, shrinking for a faster, smaller model).

## 48. Best Practices

-   **Strict Tiering:** Never allow general RAG semantic similarity to
    outrank explicit user project metadata.
-   **Middle-Out Truncation:** When hitting token limits, preserve the
    absolute beginning (system prompt/Tier 1) and absolute end (specific
    query instructions) of the context window.
-   **Fail Safe:** If traceability fails (a source ID cannot be
    resolved), drop the evidence entirely rather than passing un-citable
    text to the LLM.

## 49. Common Mistakes

-   **Leaking Responsibilities:** Allowing the Grounding Engine to
    execute embedding distance calculations (this breaks the Clean
    Architecture boundary with the RAG Engine).
-   **Blind Concatenation:** Merely pasting all retrieved chunks
    together without XML boundaries, which causes the LLM to confuse
    user code with enterprise standards.
-   **Ignoring Token Limits:** Sending massive context payloads that
    cause the LLM API to throw 400 Bad Request errors or suffer from
    massive latency spikes.

## 50. Interview Questions

1.  **Why do we need a Grounding Engine if the RAG Engine already finds
    relevant text?** *Answer:* The RAG Engine only understands
    mathematical similarity, not business authority. The Grounding
    Engine enforces policies, resolves cross-tier conflicts (e.g., User
    Project vs. Company Standard), manages strict token budgets, and
    ensures traceability.
2.  **How does the engine handle a situation where the context budget is
    exceeded by Priority 1 (Uploaded) data alone?** *Answer:* It raises
    a `413 Payload Too Large` error, signaling to the application layer
    that the user's uploaded project is too massive for a single context
    window, triggering a request for the user to narrow their focus or
    split the project.

## 51. Summary

The Grounding Engine is the ultimate guarantor of truth within the OCIF
AI Platform. By rigorously collecting, ranking, conflict-resolving, and
token-optimizing data from across the system's engines, it mechanically
prevents LLM hallucinations. It transforms probabilistic AI generation
into a deterministic, enterprise-safe process where every architectural
decision is fully auditable and strictly anchored in verified reality.
