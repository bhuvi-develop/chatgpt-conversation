# 17-project-context-engine-specification.md

# OCIF AI Platform --- Phase 1.5

## Complete Software Architecture Specification --- Project Context Engine

**Status:** Architecture / Permanent Knowledge Specification Only

> **Note:** This document defines the architecture of the Project
> Context Engine. It is **not** an implementation document. No FastAPI,
> ORM, React, or database migration code is included.

------------------------------------------------------------------------

## 1. Overview

The **Project Context Engine** is the central state and knowledge hub of
the OCIF AI Platform. It manages the complete lifecycle of every
uploaded project and ensures all downstream engines operate against the
correct active project context.

## 2. Purpose

To provide a reliable, versioned, secure and context-aware foundation
for the entire OCIF platform by managing project identity, active
context resolution and context persistence.

## 3. Objectives

-   Maintain authoritative ProjectContext records.
-   Resolve active project context.
-   Support context switching.
-   Preserve historical project versions.
-   Enable downstream AI grounding.

## 4. Business Need

Enterprise users work with multiple projects simultaneously. The Project
Context Engine guarantees complete isolation between projects while
enabling seamless switching.

## 5. Problem Statement

Without centralized context management, AI systems risk mixing unrelated
project knowledge, leading to incorrect documentation, diagrams and
recommendations.

## 6. Responsibilities

  Responsibility              Out of Scope
  --------------------------- --------------------------
  Context lifecycle           Parsing documents
  Context switching           Classification
  Active pointer management   Embedding generation
  Metadata serving            Documentation generation

## 7. Engine Architecture

The engine follows Clean Architecture and serves as the authoritative
source of project state.

## 8. Internal Modules

-   ContextLifecycleManager
-   ActivePointerService
-   ContextSwitcher
-   ContextValidator
-   ContextRepository
-   ContextCacheManager

## 9. Context Creation Pipeline

Layer 2 creates a new ProjectContext in Processing state.

## 10. Context Update Pipeline

Layer 4 enriches metadata and transitions the context to Ready.

## 11. Context Versioning

Ready contexts are immutable. Re-analysis creates a new version.

## 12. Active Context Management

Exactly one active context exists per session.

## 13. Context Switching

Supports explicit and fuzzy project switching.

## 14. Context Retrieval

Returns only contexts owned by the requesting tenant.

## 15. Context Validation

-   Exists
-   Ready
-   Authorized
-   Not deleted

## 16. Context Metadata

Stores project intelligence including stack, industry, modules and
architecture.

## 17. Context Relationships

-   UserSession
-   ProjectContext
-   ActiveContextPointer
-   ProjectSourceFile
-   ProjectContextChunk

## 18. Context Lifecycle

Draft → Processing → Enriching → Ready → Archived

## 19. Context Persistence

Stored within PostgreSQL project_ctx schema.

## 20. Context Indexing

-   B-tree
-   GIN
-   pg_trgm

## 21. Context Caching

Redis / Local cache for ActiveContextPointer.

## 22. Context History

Maintains complete historical lineage.

## 23. Context Synchronization

Uses PostgreSQL transactions and distributed cache invalidation.

## 24. Documentation Engine Integration

Provides grounded metadata.

## 25. Diagram Engine Integration

Supplies modules and relationships.

## 26. Image Engine Integration

Supplies industry, project type and business goals.

## 27. Knowledge Engine Integration

Filters domain knowledge.

## 28. Grounding Engine Integration

Provides immutable project facts.

## 29. Language Engine Integration

Maintains language continuity.

## 30. RAG Engine Integration

Restricts retrieval to active project context.

## 31. Validation Rules

-   Ready contexts only
-   Tenant isolation
-   Immutable approved contexts

## 32. Error Handling

-   403
-   404
-   409
-   422

## 33. Retry Strategy

Exponential backoff.

## 34. Logging

Context creation, switching, retrieval and validation.

## 35. Performance

-   \<5ms cache
-   \<20ms DB lookup

## 36. Security

JWT, RBAC, tenant isolation.

## 37. Database Mapping

-   ProjectContext
-   ActiveContextPointer
-   UserSession

## 38. REST API Mapping

-   GET /api/v1/projects
-   GET /api/v1/projects/{id}
-   GET /api/v1/projects/active
-   POST /api/v1/projects/switch
-   DELETE /api/v1/projects/{id}

## 39. Folder Structure

``` text
backend/app/engines/project_context/
├── context_manager.py
├── pointer_service.py
├── switcher_service.py
├── validator.py
└── exceptions.py
```

## 40. Mermaid Architecture Diagram

``` mermaid
flowchart TB
UI --> API
API --> ProjectContextEngine
ProjectContextEngine --> ProjectContextDB
ProjectContextEngine --> ActiveContextPointer
ProjectContextEngine --> DocumentationEngine
ProjectContextEngine --> DiagramEngine
ProjectContextEngine --> ImageEngine
ProjectContextEngine --> RAGEngine
```

## 41. Mermaid Sequence Diagram

``` mermaid
sequenceDiagram
User->>API: Switch Project
API->>ProjectContextEngine: Resolve
ProjectContextEngine->>Database: Lookup
Database-->>ProjectContextEngine: Context
ProjectContextEngine-->>User: Active Context Updated
```

## 42. Mermaid Component Diagram

``` mermaid
flowchart LR
Lifecycle --> Pointer
Pointer --> Validator
Validator --> Repository
Repository --> Database
```

## 43. Mermaid Deployment Diagram

``` mermaid
flowchart LR
Frontend --> FastAPI
FastAPI --> ProjectContextEngine
ProjectContextEngine --> Redis
ProjectContextEngine --> PostgreSQL
```

## 44. Industrial Examples

-   Smart Factory
-   Water Pump Monitoring
-   Smart Building
-   Energy Management

## 45. Future Extensions

-   Context Branching
-   Multi-context Comparison
-   Cold Storage

## 46. Best Practices

Always resolve context dynamically.

## 47. Common Mistakes

Never overwrite approved contexts.

## 48. Interview Questions

1.  Why separate ActiveContextPointer?
2.  Why immutable contexts?
3.  Why tenant isolation?

## 49. Summary

The Project Context Engine is the authoritative state management layer
of the OCIF AI Platform, ensuring every downstream engine operates on
the correct project while maintaining security, traceability,
scalability and enterprise-grade context isolation.
