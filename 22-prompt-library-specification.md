# OCIF AI Platform --- Phase 1.5

## Complete Software Architecture Specification --- Prompt Library

**Status:** Architecture / permanent knowledge specification only. No
implementation code (no Python, FastAPI, React, or SQL). Builds on
`01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`,
`04-api-specification.md`, `05-frontend-ux-spec.md`, the OCIF Layer
Specifications (1-8), and Engines 14--21.

------------------------------------------------------------------------

## 1. Overview

The **Prompt Library** is the centralized control plane for all prompt
engineering assets within the OCIF AI Platform. It acts as the
definitive system of record for storing, versioning, templating, and
governing the exact textual instructions sent to the underlying LLMs
(Claude).

## 2. Purpose

To decouple hardcoded string manipulation from application business
logic. The library provides a version-controlled, Jinja2-based
repository that allows prompt engineers and platform admins to refine AI
behavior dynamically without requiring backend code deployments.

## 3. Objectives

-   Centralize all System, User, and Layer-specific prompt templates.
-   Enforce strict versioning to ensure backward compatibility and
    historical reproducibility of AI generations.
-   Validate template syntax (Jinja2) to prevent runtime formatting
    errors.
-   Provide a governance workflow (Draft -\> Test -\> Approved -\>
    Active).
-   Bind model execution parameters (temperature, token limits) directly
    to their corresponding prompts.

## 4. Business Need

As AI platforms scale, prompts become complex, proprietary intellectual
property. Hardcoding them in Python files creates a bottleneck where
only developers can tune AI behavior, and changes cannot be easily
rolled back or audited. The enterprise requires a governed,
UI-accessible (or API-driven) repository where prompt engineers can
optimize instructions independently of the software release cycle.

## 5. Problem Statement

**Given** that the OCIF platform executes hundreds of distinct AI calls
across its layers and generative engines, **the Prompt Library must**
serve the exact, validated, and version-controlled prompt template for
any given task, **without** executing the prompt itself, and **without**
allowing malformed templates to crash the generative engines at runtime.

## 6. Responsibilities

  -----------------------------------------------------------------------
  Responsibility                      Excluded Responsibilities (Handled
                                      Elsewhere)
  ----------------------------------- -----------------------------------
  Storing and serving                 Executing the Claude API calls
  `PromptTemplate` records.           (Application Layer).

  Validating Jinja2 variable          Assembling grounded facts
  requirements.                       (Grounding Engine).

  Versioning and deprecating old      Generating documentation/diagrams
  prompts.                            (Doc/Diagram Engines).

  Managing model parameters           Managing UI text or localization
  (temperature, max_tokens).          (Language Engine).
  -----------------------------------------------------------------------

## 7. Engine Architecture

The Prompt Library follows the Clean Architecture pattern, residing in
the Domain and Application layers. It exposes internal service ports
(`PromptRetrievalPort`) to all other engines in the platform, and
admin-gated REST endpoints to the Frontend for governance and
optimization workflows.

## 8. Internal Modules

-   **`PromptRepository`**: Interfaces with the database and filesystem
    to retrieve templates.
-   **`VersionManager`**: Handles immutable version increments and
    active-flag toggling.
-   **`TemplateValidator`**: Parses Jinja2 AST to extract and validate
    required variables.
-   **`GovernanceService`**: Manages the approval workflow for new
    prompt versions.
-   **`ParameterVault`**: Serves the execution parameters bound to
    specific prompts.

## 9. Prompt Repository Architecture

Per `02-master-blueprint.md`, templates exist physically as `.j2` files
in the backend repository for developer ergonomics, but are mirrored and
synchronized into the PostgreSQL `prompt_lib` schema. At runtime,
engines fetch the active version exclusively from the database to enable
live updates.

## 10. Prompt Categories

Prompts are strictly categorized to match the platform's execution
contexts:

-   `system`: Core identity and behavior guardrails.
-   `layer_{1-8}`: specific instructions for OCIF pipeline processing.
-   `documentation`: Prose generation instructions.
-   `diagram`: Mermaid syntax instructions.
-   `image`: Midjourney/DALL-E prompt generation instructions.
-   `utility`: Translation, query expansion, and routing.

## 11. Prompt Variables

Every template utilizes Jinja2 syntax (e.g.,
`{{ active_project_context }}`). The library statically analyzes
templates upon upload to map all required variables. Downstream engines
*must* provide a dictionary fulfilling these variables when requesting a
template compilation.

## 12. Prompt Parameters

A prompt is not just text; it is an execution profile. The library
stores bound parameters for each template:

-   `target_model` (e.g., `claude-3-opus-20240229`)
-   `temperature` (e.g., `0.0` for extraction, `0.7` for brainstorming)
-   `max_tokens` (e.g., `4096`)
-   `top_p`

## 13. Prompt Templates

Templates isolate structure from data. A typical template includes a
`[SYSTEM BEHAVIOR]` block, a `[GROUNDED FACTS]` block (fed by the
Grounding Engine), and a `[USER INTENT]` block.

## 14. Prompt Versioning

Prompts are immutable. When an admin updates a prompt to improve its
instructions, the system creates `v2`, sets it to `is_active = true`,
and soft-deletes `v1`. This guarantees that historical
`GenerationSession` logs can always fetch the exact text that produced a
past result.

## 15. Prompt Metadata

Metadata includes `author_id`, `created_at`, `category`,
`required_variables` (JSON array), and `performance_notes` (used by
prompt engineers to document why a specific phrasing was chosen).

## 16. Prompt Lifecycle

1.  **Draft:** Created by a prompt engineer, unverified.
2.  **Testing:** Accessible only via explicit test endpoints.
3.  **Approved:** Passed peer review.
4.  **Active:** Currently serving production requests.
5.  **Deprecated:** Replaced by a newer version, retained for audit.

## 17. Prompt Validation

The `TemplateValidator` prevents bad prompts from breaking the system.
Before a draft reaches `Testing` state, it verifies that the Jinja2
syntax is valid, all variables are declared, and the total token length
of the raw template does not exceed platform thresholds.

## 18. Prompt Testing

Exposes internal endpoints allowing prompt engineers to perform A/B
testing on Draft prompts by injecting synthetic payloads and evaluating
the LLM output without affecting production users.

## 19. Prompt Optimization

Acts as the data source for future ML-ops pipelines. By linking prompt
versions to user feedback (thumbs up/down on generated architectures),
the enterprise can track which prompt versions yield the highest
accuracy.

## 20. Prompt Governance

Strict RBAC enforced at the API layer. Only users with the `PromptAdmin`
role can create or modify templates. Standard users and active project
sessions have read-only execution access via the internal service ports.

## 21. Prompt Approval Workflow

A two-person rule for critical system prompts. A draft must be submitted
for review, and a secondary `PromptAdmin` must explicitly toggle the
state to `Approved` before it can be marked `Active`.

## 22. Prompt Security

Defends against internal misconfiguration. It sanitizes templates to
ensure no dynamic execution tags (like Python `eval` blocks inside
Jinja) are present, restricting templates purely to string substitution.

## 23. Prompt Audit Trail

Every state change, version bump, and parameter adjustment is written to
`prompt_lib.PromptAuditLog`, ensuring total traceability of AI behavior
modifications.

## 24. Prompt Search

Admins can search the library by category, variable name, or text
content using standard PostgreSQL indexing to quickly locate and update
specific instructions (e.g., "Find all prompts mentioning 'JSON
formatting'").

## 25. Prompt Repository

The physical database abstraction mapping to the `PromptTemplate` table,
isolating prompt state from user project state.

## 26. Project Context Engine Integration

Supplies the extraction and classification prompts used by the Context
Engine to deduce architectural states from raw uploads.

## 27. Knowledge Engine Integration

Supplies the formatting prompts used when indexing and summarizing heavy
enterprise standards for the RAG chunking pipeline.

## 28. Grounding Engine Integration

Provides the critical system framing prompts (e.g., "You are an
enterprise AI. You MUST base your answer strictly on the XML context
below...") into which the Grounding Engine injects its assembled facts.

## 29. RAG Engine Integration

Provides the query expansion and query rewriting prompts used to
optimize semantic search strings.

## 30. Language Engine Integration

Supplies the translation, transliteration, and tone-adjustment prompts,
ensuring strict adherence to the enterprise terminology glossary.

## 31. Documentation Engine Integration

Serves the highly complex, multi-section generation templates (e.g.,
`doc_layer3_database.md.j2`) used to author Markdown deliverables.

## 32. Diagram Engine Integration

Serves the Mermaid syntax constraint prompts, ensuring the LLM does not
hallucinate unsupported node shapes or edge types.

## 33. Image Engine Integration

Serves the composition prompts used to instruct Claude on how to
generate the final Midjourney/DALL-E strings.

## 34. OCIF Layer Integration

Serves the distinct intent, capture, normalization, enrichment,
synthesis, cognition, prescription, and experience prompts that drive
the core 8-layer pipeline.

## 35. Validation Rules

-   A prompt cannot be set `Active` if its required variables list is
    empty but the template contains `{{ ... }}` tags.
-   There can be only one `Active` prompt per unique `prompt_key` at any
    given time.

## 36. Error Handling

-   `404 Not Found`: Requested `prompt_key` does not exist or has no
    active version.
-   `400 Bad Request`: Validation failure on Jinja syntax.
-   `422 Unprocessable Entity`: The consuming engine failed to provide
    all variables required by the template.

## 37. Retry Strategy

As a highly available, read-heavy internal service, the library relies
on database connection pooling. If the DB is momentarily unavailable,
the internal port falls back to the local `.j2` file system cache as a
resilient degraded mode.

## 38. Logging

Logs all administrative CRUD operations. At runtime, the consuming
engines log the `prompt_version_id` used for generation, not the Prompt
Library itself, to avoid duplicating massive text logs.

## 39. Performance

-   Prompt retrieval by `prompt_key`: \< 5ms (via Redis caching).
-   All internal service port reads must bypass HTTP overhead where
    possible.

## 40. Security

Strict separation of data. The Prompt Library stores *instructions*; it
never stores *user data*. It is immune to prompt injection because it
only provides the static template; the injection defense happens
downstream during injection and execution.

## 41. Database Mapping

-   `prompt_lib.PromptTemplate` (Main template store)
-   `prompt_lib.PromptAuditLog` (Governance tracking)

## 42. REST API Mapping

-   `GET /api/v1/admin/prompts` (List versions)
-   `POST /api/v1/admin/prompts` (Create draft)
-   `PUT /api/v1/admin/prompts/{id}/activate` (Trigger
    approval/activation)
-   Internal Microservice Port: `get_active_prompt(prompt_key)`

## 43. Folder Structure

``` text
backend/app/engines/prompt_library/
├── __init__.py
├── repository.py
├── version_manager.py
├── template_validator.py
├── governance.py
└── parameter_vault.py
```

## 44. Mermaid Architecture Diagram

``` mermaid
flowchart TB
    subgraph Admins["Prompt Engineers"]
        UI[Admin Console]
    end

    subgraph API["FastAPI"]
        R1[/api/v1/admin/prompts/]
    end

    subgraph PL["Prompt Library Engine"]
        VM[VersionManager]
        TV[TemplateValidator]
        GOV[GovernanceService]
        PV[ParameterVault]
    end

    subgraph ConsumingEngines["All OCIF Engines"]
        DOC[Documentation Engine]
        COG[Cognition Layer]
    end

    subgraph DB["Database"]
        PT[(PromptTemplate)]
    end

    UI --> R1
    R1 --> VM & GOV
    VM --> TV
    VM --> PT
    DOC & COG -->|get_prompt(key)| PV
    PV --> PT
```

## 45. Mermaid Sequence Diagram

``` mermaid
sequenceDiagram
    participant Doc as Documentation Engine
    participant PL as Prompt Library
    participant DB as Postgres
    
    Doc->>PL: get_active_prompt("doc_layer3_schema")
    PL->>DB: SELECT * FROM PromptTemplate WHERE key='doc_layer3_schema' AND is_active=true
    DB-->>PL: Template v4 + Parameters (temp=0.2)
    PL->>PL: Extract required vars: ['project_context', 'grounded_facts']
    PL-->>Doc: PromptTemplateObject
    Note over Doc: Doc Engine injects variables and calls Claude API
```

## 46. Mermaid Component Diagram

``` mermaid
componentDiagram
    component "Prompt Library Engine" {
        [PromptRepository]
        [VersionManager]
        [TemplateValidator]
        [ParameterVault]
    }
    
    [Admin API] --> [VersionManager]
    [VersionManager] --> [TemplateValidator]
    [Internal Service Port] --> [PromptRepository]
    [PromptRepository] --> [ParameterVault]
```

## 47. Mermaid Deployment Diagram

``` mermaid
flowchart LR
    subgraph AppServer["Application Container (FastAPI)"]
        PL[Prompt Library]
    end

    subgraph CacheTier["Redis"]
        PROMPT_CACHE[Active Template Cache]
    end

    subgraph DatabaseTier["PostgreSQL"]
        PROMPT_SCHEMA[prompt_lib schema]
    end

    PL <--> PROMPT_CACHE
    PL <--> PROMPT_SCHEMA
```

## 48. Industrial Examples

The Prompt Library ensures specific industrial tasks always use
mathematically optimal model parameters.

## 49. Water Pump Example

**Scenario:** Extracting pump tolerances from a datasheet. **Prompt
Action:** The `layer2_extraction` prompt is fetched. The
`ParameterVault` ensures this prompt is executed with
`temperature = 0.0` and `top_p = 0.1` to prevent the LLM from rounding
or approximating exact PSI measurements.

## 50. Smart Building Example

**Scenario:** Generating an HVAC architecture diagram. **Prompt
Action:** The `diagram_hvac_flow` prompt requires variables
`{{ sensor_list }}` and `{{ controller_type }}`. The `TemplateValidator`
ensures that if a developer accidentally updates the template to expect
`{{ hvac_type }}` without updating the Diagram Engine, the prompt is
rejected in the Draft phase, preventing a production crash.

## 51. Attendance System Example

**Scenario:** A company updates its strict privacy rules regarding
biometric data. **Prompt Action:** The prompt engineer writes v2 of
`layer7_prescription_rules`, instructing the LLM to aggressively flag
PII storage. They activate v2. Immediately, all new chats globally use
the new privacy constraints. Old `GenerationSessions` still point to v1
in the database, preserving historical auditability.

## 52. Future Extensions

-   **Multi-Provider Syntax Translation:** Automatically converting
    Claude-optimized prompt templates into OpenAI or Google
    Gemini-optimized formats to support hot-swapping foundational models
    without manual rewrites.
-   **Cost Analytics:** Binding token-cost tracking directly to the
    `prompt_key` to analyze which specific architectural templates
    consume the most enterprise budget.

## 53. Best Practices

-   **Treat Prompts as Code:** They must go through peer review,
    validation, and testing before hitting production.
-   **Parameter Binding:** Never let the consuming engine guess the
    temperature or top-p; these belong natively to the prompt's intent
    and must be stored together in the `ParameterVault`.
-   **Never Delete:** Use `is_active = false`. Hard-deleting prompts
    destroys the ability to debug past AI hallucinations.

## 54. Common Mistakes

-   **Leaking Logic:** Placing complex conditional logic (e.g., massive
    if/else chains) inside the Jinja template instead of handling it in
    the OCIF Application layer.
-   **Variable Mismatch:** Activating a prompt that expects
    `{{ user_name }}` when the consuming engine only knows how to
    provide `{{ session_context }}`.
-   **Execution Bleed:** Allowing the Prompt Library to actually call
    the Anthropics API. It is a library, not a runtime agent.

## 55. Interview Questions

1.  **Why do we store prompt templates in the database instead of just
    keeping them as `.py` string constants?** *Answer:* To enable live
    governance and updates without requiring an application
    redeployment. It treats AI instructions as configuration data rather
    than compiled code, enabling prompt engineers to iterate rapidly and
    track version history.
2.  **How does the system ensure a malformed Jinja template doesn't
    crash the generation engine?** *Answer:* The `TemplateValidator`
    intercepts all changes at the governance layer. It performs AST
    (Abstract Syntax Tree) parsing on the template text before it can be
    marked `Approved`, guaranteeing that syntax errors are caught before
    runtime.

## 56. Summary

The Prompt Library is the central nervous system of OCIF's AI
instructions. By providing a secure, version-controlled, and strictly
validated repository for all Jinja2 templates and execution parameters,
it decouples prompt engineering from software engineering. This ensures
that every AI response across the platform is generated using audited,
reproducible, and enterprise-approved instructions.
