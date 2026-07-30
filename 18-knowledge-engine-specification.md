# OCIF AI Platform --- Phase 1.5

## Complete Software Architecture Specification --- Knowledge Engine

**Status:** Architecture / permanent knowledge specification only. No
implementation code (no Python, FastAPI, or SQL). Builds on
`01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`,
`04-api-specification.md`, `05-frontend-ux-spec.md`, the OCIF Layer
Specifications (1-8), and `17-project-context-engine-specification.md`.

------------------------------------------------------------------------

## 1. Overview

The **Knowledge Engine** is the enterprise reference repository for the
OCIF AI Platform. Distinct from user-uploaded session projects, it acts
as the authoritative manager for curated, admin-uploaded reference
material---such as technical standards, SOPs, and industrial manuals. It
ensures that the OCIF AI Platform is grounded in approved organizational
and industrial knowledge.

## 2. Purpose

To durably ingest, validate, classify, index, version, and manage
authoritative enterprise knowledge. The engine guarantees that
downstream systems (like the RAG Engine) have access to structured,
high-quality, industry-tagged reference materials, thereby preventing AI
hallucinations in specialized industrial domains.

## 3. Objectives

-   Provide a dedicated, admin-gated pipeline for ingesting curated
    reference documents.
-   Tag and classify knowledge by industry, domain, and document class.
-   Version all knowledge artifacts to maintain auditability and
    backward compatibility.
-   Prepare and index knowledge for semantic retrieval by downstream
    engines.
-   Enforce rigorous knowledge governance, quality checks, and lifecycle
    management.

## 4. Business Need

Enterprise engineering relies heavily on standardized frameworks,
compliance manuals, and internal best practices. Without a centralized
Knowledge Engine, an AI platform operates purely on user uploads or base
LLM weights, leading to non-compliant, unsafe, or technically inaccurate
architectural recommendations. The business requires a governed system
to inject proprietary and standard engineering truths into the AI's
reasoning context.

## 5. Problem Statement

**Given** the critical need for generated architectures to adhere to
strict industrial standards, **the Knowledge Engine must** securely
ingest, structure, and lifecycle-manage approved reference materials,
**without** executing retrievals or generative tasks itself, and
**without** allowing unverified user session data to contaminate the
curated global knowledge base.

## 6. Responsibilities

  -----------------------------------------------------------------------
  Responsibility                      Excluded Responsibilities (Handled
                                      Elsewhere)
  ----------------------------------- -----------------------------------
  Ingesting admin-curated reference   Performing RAG vector retrieval
  materials.                          (RAG Engine).

  Managing `KnowledgeDocument`        Generating documentation or
  schemas and metadata.               diagrams (Doc/Diag Engines).

  Versioning, activating, and         Performing reasoning or
  archiving knowledge.                recommendations
                                      (Cognition/Prescription).

  Indexing knowledge for downstream   Parsing user-uploaded project files
  searchability.                      (Capture Layer).
  -----------------------------------------------------------------------

## 7. Engine Architecture

The Knowledge Engine follows the Clean Architecture pattern, residing
within the Application and Domain layers. It operates independently of
the real-time OCIF user pipeline, exposing admin-gated REST interfaces
for ingestion, and internal service ports for metadata and index access
by downstream engines (specifically the RAG Engine and Synthesis Layer).

## 8. Internal Modules

-   **`IngestionPipelineManager`**: Orchestrates the intake of new
    reference files.
-   **`KnowledgeCatalogService`**: Manages categories, tags, and
    document classes.
-   **`VersionControlService`**: Handles version increments, active
    flags, and deprecation.
-   **`GovernanceValidator`**: Enforces quality standards and metadata
    completeness.
-   **`IndexPreparationService`**: Formats and triggers vector indexing
    workflows.

## 9. Knowledge Ingestion Pipeline

Admin uploads trigger a specialized ingestion path:

1.  Admin submits a document via `POST /api/v1/admin/knowledge`.
2.  The engine extracts basic text and metadata using shared
    infrastructure parsers.
3.  The text is passed to the `IndexPreparationService` for semantic
    chunking.
4.  The embedded chunks are written to the `knowledge` schema.
5.  The document remains in a `Processing` state until validation
    passes.

## 10. Knowledge Validation Pipeline

Before a document state becomes `Ready` (`is_active=true`), the
`GovernanceValidator` ensures:

-   The `doc_class` mapping is valid.
-   Required `industry_tags` are present.
-   Vector embeddings were successfully generated for all extracted
    chunks.
-   The upload passes duplication checks against existing active
    versions.

## 11. Knowledge Classification

Documents are strictly classified using the `doc_class` enum: `manual`,
`standard`, `sop`, `reference`, `regulatory`, or `company`. This
taxonomy allows downstream retrieval engines to filter by exact
knowledge intent.

## 12. Knowledge Categorization

Categorization is driven by `industry_tags` and `domain_tags` (stored as
JSONB). These tags align with the classifications produced by Layer 4
(Enrichment), allowing seamless cross-referencing between user projects
and global knowledge.

## 13. Knowledge Versioning

Knowledge is immutable once active. When a standard is updated, the
previous `KnowledgeDocument` is soft-deleted (`is_active=false`), and a
new version is created. This preserves the integrity of historical chat
sessions that relied on previous standards.

## 14. Knowledge Metadata

The engine tracks robust metadata including `uploaded_by` (Admin ID),
`version_number`, `created_at`, `org_id` (for tenant isolation),
`industry_tags`, `domain_tags`, and `compliance_framework`.

## 15. Knowledge Lifecycle

1.  **Draft:** Initial upload, awaiting metadata.
2.  **Processing:** Extracting, chunking, and embedding.
3.  **Active:** Approved, indexed, and available for downstream
    consumption.
4.  **Deprecated:** Soft-deleted, replaced by a newer version,
    accessible only for historical audit.

## 16. Knowledge Repository

The core database abstraction, serving as the ultimate source of truth
for all enterprise guidelines. It separates global knowledge
(`org_id = null`) from tenant-specific proprietary knowledge
(`org_id = tenant_uuid`).

## 17. Knowledge Storage

Raw physical files (PDF, DOCX) are written to secure Blob Storage (e.g.,
AWS S3). Parsed, embedded representations and relational metadata are
stored in PostgreSQL within the dedicated `knowledge` logical schema.

## 18. Knowledge Indexing

The engine coordinates the storage of vector embeddings using `pgvector`
(HNSW indexing) in the `KnowledgeChunk` table. Relational lookups
utilize B-tree indexes on `doc_class` and GIN indexes on `industry_tags`
for rapid pre-filtering.

## 19. Knowledge Search

*Boundary Note:* The Knowledge Engine **does not perform semantic
similarity search**. It exposes internal query interfaces (ports) and
indexed database views that the **RAG Engine** utilizes to perform the
actual retrieval during Layer 5 (Synthesis).

## 20. Knowledge Quality Management

Ensures that no empty, corrupt, or poorly chunked document enters the
active pool. The engine utilizes background workers to evaluate chunk
token density and metadata completeness, flagging low-quality ingestions
for manual admin review.

## 21. Knowledge Governance

Enforces strict Role-Based Access Control (RBAC). Only users with the
`PlatformAdmin` or `KnowledgeAdmin` roles can mutate the repository. The
engine maintains an immutable audit log of all creations, updates, and
deprecations.

## 22. Industry Knowledge Management

Manages broad, globally applicable materials (e.g., IEEE, ISO
specifications). These are tagged appropriately and made available
across all platform tenants.

## 23. Engineering Standards Repository

Manages rigorous technical constraints, safety codes, and architectural
patterns (e.g., IEC 62443 for cybersecurity), ensuring generated output
complies with explicitly tagged standards.

## 24. Best Practice Repository

Holds internal SOPs and company-specific design patterns. These are
scoped strictly to the uploading `org_id`, ensuring proprietary
practices never leak across tenants.

## 25. Prompt Knowledge Integration

The engine aligns logically with the Prompt Library. Downstream engines
can configure prompts to demand strict adherence to specific
`doc_class = standard` knowledge provided by this engine.

## 26. Template Knowledge Integration

Integrates with the Template Library by providing curated engineering
checklists and compliance requirements that are injected into
documentation template variables.

## 27. Project Context Engine Integration

The Knowledge Engine utilizes the active project's `industry` (resolved
by the Project Context Engine) to filter and expose only the
`KnowledgeDocument` records relevant to the current user session.

## 28. Documentation Engine Integration

Prepares and serves high-fidelity reference texts to the Documentation
Engine, allowing Layer 6 (Cognition) to cite specific enterprise SOPs
when drafting sections.

## 29. Diagram Engine Integration

Provides structured architectural standard patterns (e.g., required
security layers from a compliance manual) that guide the generation of
Mermaid diagram nodes.

## 30. Image Engine Integration

Supplies reference iconography conventions and industrial visual
standards to inform how the Image Engine themes its prompts.

## 31. Grounding Engine Integration

Acts as the authoritative third tier in the platform's grounding chain
(Priority 1: User Uploads -\> Priority 2: Project Context -\> Priority
3: **Knowledge Base**).

## 32. Language Engine Integration

Stores and manages domain-specific glossaries and enterprise-approved
translations, which the Language Engine references to ensure technical
terminology is rendered accurately in regional languages (e.g., Tamil,
Hindi).

## 33. RAG Engine Integration

The Knowledge Engine maintains the structural integrity of the
`KnowledgeChunk` vector space, providing the indexed, metadata-filtered
boundary within which the RAG Engine executes its queries.

## 34. Validation Rules

-   Uploads must include an explicit `doc_class` and at least one
    `industry_tag`.
-   Updates to an existing standard must increment the version number
    and deprecate the previous ID.
-   File payloads must conform to approved enterprise formats (PDF,
    DOCX, MD).

## 35. Error Handling

-   `400 Bad Request`: Missing mandatory metadata (e.g., missing
    industry tags).
-   `403 Forbidden`: Insufficient admin privileges to mutate knowledge.
-   `409 Conflict`: Attempting to upload a duplicate document hash.
-   `422 Unprocessable Entity`: File parsing or vector embedding failure
    during processing.

## 36. Retry Strategy

Asynchronous ingestion tasks (parsing and embedding) utilize Celery
background workers with exponential backoff for transient external API
failures (e.g., rate limits on the embedding model).

## 37. Logging

Logs all administrative mutations including the `admin_user_id`,
`document_id`, and action (`UPLOAD`, `DEPRECATE`). Logs vector
processing durations to monitor ingestion pipeline health. Does not log
raw proprietary document text.

## 38. Performance

-   Vector ingestion of standard manuals (\~100 pages) must complete
    asynchronously within 90 seconds.
-   Metadata filter queries (preparing the index for RAG) must resolve
    in \< 15ms.

## 39. Security

-   Rigorous tenant isolation enforced via `org_id` at the database
    repository layer.
-   Raw files stored in S3 are encrypted at rest using KMS.
-   Hard deletion is prohibited; soft-deletion ensures historical audit
    trails are preserved.

## 40. Database Mapping

-   `knowledge.KnowledgeDocument` (Relational metadata)
-   `knowledge.KnowledgeChunk` (Vector embeddings via pgvector)
-   `knowledge.KnowledgeAuditLog` (Governance tracking)

## 41. REST API Mapping

-   `POST /api/v1/admin/knowledge` (Upload)
-   `GET /api/v1/admin/knowledge` (List/Search metadata)
-   `GET /api/v1/admin/knowledge/{id}` (Retrieve details)
-   `PUT /api/v1/admin/knowledge/{id}/deprecate` (Soft delete)

## 42. Folder Structure

``` text
backend/app/engines/knowledge/
├── __init__.py
├── ingestion_pipeline.py
├── catalog_service.py
├── version_control.py
├── governance_validator.py
├── index_prep_service.py
└── exceptions.py
```

## 43. Mermaid Architecture Diagram

``` mermaid
flowchart TB
    subgraph Admins["Admin Interface"]
        UI[Upload Console]
    end

    subgraph API["FastAPI"]
        R1[/api/v1/admin/knowledge/]
    end

    subgraph KE["Knowledge Engine"]
        ING[IngestionPipelineManager]
        CAT[KnowledgeCatalogService]
        GOV[GovernanceValidator]
        VC[VersionControlService]
    end

    subgraph ConsumingEngines["Downstream Systems"]
        RAG[RAG Engine]
    end

    subgraph DB["Database"]
        KD[(KnowledgeDocument)]
        KC[(KnowledgeChunk - pgvector)]
    end

    UI --> R1
    R1 --> ING
    ING --> GOV
    GOV --> CAT & VC
    CAT --> KD
    ING --> KC
    RAG -->|Queries Indexed Data| KC
```

## 44. Mermaid Sequence Diagram

``` mermaid
sequenceDiagram
    participant Admin
    participant API as /admin/knowledge
    participant ING as IngestionPipeline
    participant GOV as GovernanceValidator
    participant DB as DB (knowledge schema)

    Admin->>API: POST file + metadata (tags)
    API->>ING: trigger async ingestion
    ING->>GOV: Validate metadata
    GOV-->>ING: Validation Passed
    ING->>ING: Parse & Chunk document
    ING->>ING: Request Embeddings
    ING->>DB: Store KnowledgeChunk vectors
    ING->>DB: Insert KnowledgeDocument (is_active=true)
    ING->>API: Ingestion Complete
    API-->>Admin: 201 Created
```

## 45. Mermaid Component Diagram

``` mermaid
componentDiagram
    component "Knowledge Engine" {
        [IngestionPipelineManager]
        [KnowledgeCatalogService]
        [VersionControlService]
        [GovernanceValidator]
        [IndexPreparationService]
    }
    
    [Admin Upload API] --> [IngestionPipelineManager]
    [IngestionPipelineManager] --> [GovernanceValidator]
    [GovernanceValidator] --> [VersionControlService]
    [IngestionPipelineManager] --> [IndexPreparationService]
    [IndexPreparationService] ..> [PostgreSQL/pgvector]
```

## 46. Mermaid Deployment Diagram

``` mermaid
flowchart LR
    subgraph AppServer["Application Container (FastAPI)"]
        KE[Knowledge Engine]
    end

    subgraph Workers["Celery Worker Tier"]
        ING_JOB[Ingestion Task]
        IDX_JOB[Indexing Task]
    end

    subgraph DatabaseTier["PostgreSQL"]
        KNOW_SCHEMA[knowledge logical schema]
    end

    KE --> ING_JOB
    ING_JOB --> IDX_JOB
    IDX_JOB --> KNOW_SCHEMA
```

## 47. Industrial Examples

The Knowledge Engine manages domain-specific rules required to prevent
generic, unsafe LLM hallucinations in highly specialized fields.

## 48. Water Pump Example

**Scenario:** An engineering firm designs industrial centrifugal pumps.
**Action:** An admin uploads the *ISO 13709 (API 610)* standard. It is
categorized with
`industry_tags = ["fluid-dynamics", "industrial-pumping"]` and
`doc_class = standard`. **Result:** When the RAG Engine searches on
behalf of a user's water pump project, the Knowledge Engine's indexed
space ensures the AI references specific vibration limits and seal
requirements defined by API 610, rather than generating generic advice.

## 49. Smart Building Example

**Scenario:** A company creates IoT building management systems.
**Action:** An admin uploads the *ASHRAE 90.1* energy standard. It is
categorized with `industry_tags = ["smart-building", "hvac"]` and
`doc_class = regulatory`. **Result:** When generating a Layer 5
Synthesis document, the RAG Engine retrieves these chunks. The AI is
forced to ground its HVAC controller architecture recommendations in
ASHRAE compliance, ensuring energy-efficient design patterns are
strictly followed.

## 50. Attendance System Example

**Scenario:** A corporate IT team builds biometric attendance software.
**Action:** An admin uploads the *Internal GDPR Data Processing SOP*. It
is categorized with `industry_tags = ["hr-tech", "identity"]`,
`doc_class = sop`, and linked to the specific `org_id`. **Result:**
Because it is org-isolated, only employees of that enterprise will have
this SOP indexed. The AI will cross-reference biometric data retention
policies specific to that company's legal requirements when mapping out
the Layer 3 database design.

## 51. Future Extensions

-   **Knowledge Graphs:** Expanding the engine to automatically extract
    and persist entity-relationship graphs from standards to map
    explicit dependencies (e.g., ISO standard A requires compliance with
    ISO standard B).
-   **Automated Expiry / TTL:** Setting Time-To-Live limits on specific
    regulatory documents to force administrative review of stale SOPs.

## 52. Best Practices

-   **Decouple Ingestion from Search:** Keep heavy PDF parsing and
    embedding out of the critical path of user chat requests.
-   **Tag Broadly, Filter Narrowly:** Ensure documents receive
    comprehensive `industry_tags` during ingestion so they are not
    orphaned from relevant projects.
-   **Immutable History:** Always use versioning and soft-deletes; never
    overwrite a document that previous user sessions relied upon for
    generation.

## 53. Common Mistakes

-   **Violating Boundaries:** Allowing the Knowledge Engine to execute
    RAG queries. It must only *prepare and manage* the data; the RAG
    Engine executes the retrieval.
-   **Contaminating the Base:** Uploading draft user project files into
    the Knowledge Engine. It must remain strictly for curated,
    authoritative reference materials.
-   **Ignoring Org Isolation:** Failing to apply `org_id` checks, which
    could leak one tenant's proprietary SOPs to another enterprise.

## 54. Interview Questions

1.  **Why doesn't the Knowledge Engine execute the RAG similarity
    search?** *Answer:* To maintain the Single Responsibility Principle.
    The Knowledge Engine governs data lifecycle, indexing, and quality.
    The RAG Engine specializes in vector search, ranking, and retrieval
    across multiple contexts (Project vs. Knowledge Base).
2.  **How does the system ensure an old user project isn't broken if a
    standard is updated?** *Answer:* The engine uses strict versioning
    and soft-deletions (`is_active=false`). Old project generation
    sessions are linked to the specific version of the
    `KnowledgeDocument` that was active at the time of generation.

## 55. Summary

The Knowledge Engine forms the governed, authoritative foundation of
enterprise intelligence for the OCIF AI Platform. By securely ingesting,
validating, and organizing curated standards and SOPs---while strictly
avoiding downstream generation tasks---it guarantees that the platform's
outputs are grounded in approved, high-quality, and structurally indexed
organizational knowledge.
