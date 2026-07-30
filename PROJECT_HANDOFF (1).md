# PROJECT_HANDOFF.md

> **This document is the single source of truth for project continuity.**
> Any AI assistant (Claude, ChatGPT, Copilot, Cursor, Gemini, or otherwise) picking up this project must read this file in full before taking any action.

---

# Project Information

| Field | Value |
|---|---|
| **Project Name** | OCIF AI Platform (Octagonal Cognitive Intelligence Framework) |
| **Version** | 0.1.4.3-architecture (pre-implementation) |
| **Current Development Phase** | Phase 1.4 in progress — OCIF Layer Knowledge Repository specifications. Layers 1–3 (Perception, Capture, Normalization) approved. Layer 4 (Enrichment) in progress. |
| **Current Status** | **Architecture-only.** No backend code, no frontend code, no database has been created. All work to date is design documentation. |

---

# Product Vision

### Purpose of the Platform
An Enterprise AI Documentation Platform that understands any uploaded software/industrial project and automatically generates professional engineering documentation, diagrams, architecture explanations, reports, and images — behaving like ChatGPT/Claude but specialized for engineering, software architecture, and industrial systems.

### Business Goals
- Eliminate manual, slow, inconsistent engineering documentation work.
- Produce patent-ready, presentation-ready, enterprise-grade technical documentation automatically.
- Support multi-industry use (software systems, industrial/IoT systems) via automatic project/domain detection.
- Serve as a reusable knowledge and documentation engine across many uploaded projects, not a single-purpose tool.
- Provide grounded, non-hallucinated output suitable for enterprise/engineering trust requirements.

### Core Features
1. Enterprise AI Chat
2. Project Context Engine
3. OCIF Engine (8-layer cognitive pipeline)
4. Documentation Engine
5. Diagram Engine
6. Image Generation Engine
7. Knowledge Engine
8. Grounded RAG Engine
9. Enterprise Dashboard
10. Multi-language Engine

---

# Architecture Status

| Document | Status |
|---|---|
| ✅ Product Vision | Defined (this document + original Project Constitution) |
| ✅ Architecture (Phase 1) | Approved |
| ✅ Master Blueprint (Phase 1 revised) | Approved |
| ✅ Database Design (Phase 1.1) | Approved |
| ✅ API Specification (Phase 1.2) | Approved |
| ✅ Frontend UX Specification (Phase 1.3) | Approved |
| ✅ OCIF Layer 1 — Perception Specification (Phase 1.4) | Approved |
| ✅ OCIF Layer 2 — Capture Specification (Phase 1.4) | Approved |
| ✅ OCIF Layer 3 — Normalization Specification (Phase 1.4) | Approved |
| ⬜ OCIF Layer 4 — Enrichment Specification (Phase 1.4) | In progress |
| ⬜ OCIF Layer 5 — Synthesis Specification (Phase 1.4) | Not started |
| ⬜ OCIF Layer 6 — Cognition Specification (Phase 1.4) | Not started |
| ⬜ OCIF Layer 7 — Prescription Specification (Phase 1.4) | Not started |
| ⬜ OCIF Layer 8 — Experience Specification (Phase 1.4) | Not started |
| ⬜ Security Architecture | Not started |
| ⬜ Backend Implementation | Not started |
| ⬜ Frontend Implementation | Not started |
| ⬜ Testing Strategy | Not started |
| ⬜ Deployment Architecture (cloud IaC) | Not started (local-first path defined only) |

### Per-Document Detail

**1. Architecture (`01-architecture.md`)**
- **Purpose:** Establishes Clean Architecture layering, tech stack (Python/FastAPI, Anthropic Claude API, PostgreSQL+pgvector, Docker), OCIF 8-layer-to-module mapping, folder skeleton, deployment path.
- **Status:** Approved.
- **Dependencies:** None (foundational document).

**2. Master Blueprint (`02-master-blueprint.md`)**
- **Purpose:** Expands Phase 1 with Documentation Engine, Prompt Library, Template Library, Image Generation Engine (multi-provider), Diagram Engine (Mermaid→SVG→PNG/PDF), Language Engine, Context Switching Engine, Knowledge Base Engine, OCIF Layer Repository, and the master Enterprise AI Flow sequence diagram.
- **Status:** Approved.
- **Dependencies:** `01-architecture.md`.

**3. Database Design (`03-database-design.md`)**
- **Purpose:** Complete schema — every table, field, PK/FK, index, relationship — across identity, project_ctx, knowledge, prompt_lib, template_lib, diagram, image, generation, conversation schemas. Full ER diagram and 4 data-flow diagrams.
- **Status:** Approved.
- **Dependencies:** `01-architecture.md`, `02-master-blueprint.md`.

**4. API Specification (`04-api-specification.md`)**
- **Purpose:** Complete REST API contract — 20 sections, ~45 endpoints, OpenAPI-ready — covering auth, users, projects, OCIF layers, documentation/diagram/image generation, knowledge/prompt/template libraries, context switching, language, chat, generation sessions, export, admin, health. Plus 8 flow diagrams.
- **Status:** Approved.
- **Dependencies:** `01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md` (endpoints map directly to the schema).

**5. Frontend UX Specification (`05-frontend-ux-spec.md`)**
- **Purpose:** Complete UX/UI design document — 50 sections — design philosophy, systems (color/typography/tokens/spacing/grid/components), every major page/screen with full interaction detail, and platform-wide patterns (accessibility, states, animation, responsive rules). No implementation code.
- **Status:** Approved.
- **Dependencies:** `04-api-specification.md` (screens map to API capabilities), `02-master-blueprint.md` (engines surfaced in UI).

**6. OCIF Layer 1 — Perception Specification (`06-ocif-layer1-perception-specification.md`)**
- **Purpose:** Permanent, canonical knowledge source for OCIF Layer 1 (intent/language/request-type classification), loaded into `OCIFLayerRepository` (`layer_number = 1`). 37-section spec: overview through future extension points, worked examples, 5 canonical Mermaid diagrams, Prompt/Documentation/Diagram/Image template designs.
- **Status:** Approved.
- **Dependencies:** `01-architecture.md` through `05-frontend-ux-spec.md`.

**7. OCIF Layer 2 — Capture Specification (`07-ocif-layer2-capture-specification.md`)**
- **Purpose:** Permanent knowledge source for OCIF Layer 2 (file ingestion: PDF/DOCX/MD/ZIP/code parsing, ZIP-bomb guards, `ProjectContext`/`ProjectSourceFile` creation), loaded into `OCIFLayerRepository` (`layer_number = 2`). Same 37-section format as Layer 1.
- **Status:** Approved.
- **Dependencies:** `01-architecture.md` through `06-ocif-layer1-perception-specification.md`.

**8. OCIF Layer 3 — Normalization Specification (`08-ocif-layer3-normalization-specification.md`)**
- **Purpose:** Permanent knowledge source for OCIF Layer 3 (text cleaning, semantic chunking, embedding generation, `ProjectContextChunk` persistence), loaded into `OCIFLayerRepository` (`layer_number = 3`). Same 37-section format as Layers 1–2.
- **Status:** Approved.
- **Dependencies:** `01-architecture.md` through `07-ocif-layer2-capture-specification.md`.

---

# Current Folder Structure

```
ocif-platform/
├── backend/
│   ├── app/
│   │   ├── domain/                      # entities, value objects, pure logic
│   │   │   ├── entities/
│   │   │   └── ocif_layers/             # interfaces only
│   │   ├── ocif/                        # 8 concrete OCIF layer implementations
│   │   ├── engines/
│   │   │   ├── project_context/
│   │   │   ├── documentation/
│   │   │   ├── diagram/
│   │   │   │   └── render/              # mmdc/svg/png/pdf pipeline
│   │   │   ├── image/
│   │   │   │   └── providers/           # gpt_image, dalle, stable_diffusion, midjourney, claude_refiner
│   │   │   ├── knowledge/               # admin ingestion + retrieval
│   │   │   ├── rag/
│   │   │   ├── language/
│   │   │   └── context_switching/
│   │   ├── repository/                  # Prompt Library + Template Library (files, not hardcoded)
│   │   │   ├── templates/{documentation,diagram,image}/
│   │   │   └── prompts/{system,enrichment,synthesis,prescription,language}/
│   │   ├── api/
│   │   │   ├── routers/                 # chat, upload, documentation, diagram, image, context, admin, health
│   │   │   └── schemas/                 # Pydantic request/response models
│   │   ├── infrastructure/
│   │   │   ├── llm/                     # Anthropic client wrapper
│   │   │   ├── vectorstore/             # pgvector adapter
│   │   │   ├── db/                      # SQLAlchemy models, Alembic migrations
│   │   │   ├── parsers/                 # pdf, docx, md, zip, code parsers
│   │   │   ├── mermaid_render/          # headless mmdc subprocess/service
│   │   │   └── storage/                 # local FS now, S3-compatible later
│   │   ├── core/                        # config, DI container, logging
│   │   └── main.py
│   ├── tests/
│   └── requirements.txt
├── frontend/                            # Not yet started (Phase 3)
├── docs/
│   ├── architecture/                    # 01–05 documents + this handoff file
│   └── continuation/                    # per-generation continuation files
└── infra/
    ├── docker-compose.yml               # local: postgres+pgvector, backend, frontend
    ├── mermaid-render/                  # Docker image for mmdc + Chromium
    └── cloud/                           # Phase 7: Terraform/IaC, not started
```

---

# Backend Architecture (Engine Summary)

| Engine | Responsibility | Key Design Points |
|---|---|---|
| **OCIF Engine** | Runs every request through the 8 fixed layers: Perception → Capture → Normalization → Enrichment → Synthesis → Cognition → Prescription → Experience | Each layer is an isolated module implementing `OCIFLayer.process(context)`; each layer's prompts/templates/rules/examples live in `OCIFLayerRepository` (one row per layer, 8 total) — never scattered in code |
| **Project Context Engine** | Detects project/industry/domain/modules/APIs/database/business goal/architecture from any upload | Persists as `ProjectContext`; every future answer in a session queries this first (grounding priority #1/#2); never overwritten — new uploads create new rows |
| **Documentation Engine** | Generates the 31-section documentation for any requested layer | Two-stage fill: Markdown template defines structure, Prompt Library fills each section's content via Claude; supports "explain ONLY layer N" via direct module invocation |
| **Diagram Engine** | Generates Mermaid diagrams, exports to SVG/PNG/PDF | Diagram templates (`.mmd.j2`) filled with Project-Context-derived node/edge content; mermaid-cli renders SVG, cairosvg/resvg converts to PNG/PDF; cached by content hash |
| **Image Engine** | Multi-provider image generation | Strategy pattern (`ImageProvider` interface): GPT Image, DALL·E, Stable Diffusion, Midjourney; Claude is used only to refine/compose the image prompt, never as the image renderer itself |
| **Knowledge Engine** | Admin-managed reference library (manuals, standards, SOPs, company docs) | Separate ingestion path from user uploads; industry-tagged; ranks as grounding priority #3 |
| **Grounding Engine (RAG)** | Enforces the strict grounding priority chain | Order: Uploaded Project → Project Context → Knowledge Base → RAG retrieval → LLM; if none found, explicitly states no grounded info is available — never fabricates |
| **Language Engine** | Detects and generates responses in English/Tamil/Tanglish/Hindi/Malayalam/Kannada/Telugu/mixed | Fast lang-id first, Claude fallback for low-confidence/code-mixed text; generates directly in the target language rather than translating after the fact (preserves technical terminology correctly) |

Supporting systems: **Context Switching Engine** (`ActiveContextPointer` ensures the newest upload becomes authoritative immediately, with explicit named-switch support), **Prompt Library** and **Template Library** (no hardcoded prompts/templates anywhere — all versioned, file-based, DB-indexed), **Continuation System** (`GenerationSession` tracks completed/remaining sections so long generations survive context limits).

---

# Frontend Architecture (Module Summary)

*(Design only — no implementation yet.)*

| Module | Summary |
|---|---|
| Navigation shell | Sidebar (primary nav + active-project indicator) + Top Nav (breadcrumb, search, language, notifications, profile) |
| Dashboard | Stat cards, recent projects, recent generations, quick actions |
| AI Chat Workspace | Core conversational surface; flat message layout; inline artifact previews; grounding-source transparency strip on every AI answer |
| Project Upload / Explorer | Drag-drop ingestion with live pipeline-stage stepper; browse/manage all uploaded projects; explicit "Set as Active" |
| OCIF Layer Explorer | 8-node rail (the platform's signature visual motif); scoped generation of documentation/diagram/image per layer |
| Documentation / Diagram / Image Viewers | Dedicated full-fidelity viewers with TOC navigation, zoom/pan canvas, provider metadata, export controls |
| Knowledge / Prompt / Template Libraries | Admin-facing management surfaces for the three "no hardcoding" repositories |
| Generation History / Export Center | Operations-style dense tables; async job monitoring and PDF/DOCX bundling |
| Settings / Profile / Admin Console | Personal preferences, identity, and org-wide administration (users, image providers, OCIF repository editing, stats) |
| Cross-cutting systems | Theme Engine (dark-first, token-driven), Multi-language UX, Notifications, AI Assistant Panel (persistent contextual chat), full responsive rules (desktop/tablet/mobile), accessibility (WCAG AA), empty/loading/error state patterns, restrained enterprise animation |

Design language: dark-first, glassmorphism used sparingly (overlays only), single restrained accent color, diagrams/Mermaid treated as first-class content, three-pane layout (sidebar / main / contextual panel) as the platform's structural signature.

---

# Database Status

- **Engine:** PostgreSQL + pgvector, single physical database, logically separated by schema (`identity`, `project_ctx`, `knowledge`, `prompt_lib`, `template_lib`, `diagram`, `image`, `generation`, `conversation`).
- **Core tables designed:** `Organization`, `User`, `UserSession`, `ProjectContext`, `ProjectSourceFile`, `ProjectContextChunk`, `ActiveContextPointer`, `KnowledgeDocument`, `KnowledgeChunk`, `PromptTemplate`, `TemplateRegistry`, `OCIFLayerRepository` (+3 join tables), `DiagramArtifact`, `ImageArtifact`, `ImageProviderConfig`, `GenerationSession`, `ConversationMessage`.
- **Conventions:** UUID surrogate PKs everywhere; `created_at`/`updated_at` on all tables; soft-delete (`is_active`/`deleted_at`) instead of hard deletes for anything versioned or auditable; vector columns get ANN (HNSW/IVFFlat) indexes; jsonb fields get GIN indexes where containment queries are needed; partial unique indexes enforce "exactly one active version" patterns.
- **Not yet done:** actual SQL/migrations, ORM models, seed data.

---

# API Status

- **Contract fully specified**, not implemented. REST over HTTPS, `/api/v1/` base path, standard JSON error envelope, standard status-code usage, idempotency keys on creating POSTs, async (`202 Accepted` + polling) pattern for all generation-type operations.
- **20 endpoint groups specified:** Auth, Authorization/RBAC (viewer/engineer/admin), Users, Project Upload, Project Context, OCIF Layers, Documentation, Diagram, Image, Knowledge Base, Prompt Library, Template Library, Context Switching, Language, Chat, Generation Sessions, Export, Admin, Health.
- **Not yet done:** actual FastAPI route implementations, OpenAPI YAML generation, request/response Pydantic models, auth middleware code.

---

# UI Status

- **50-section UX/UI specification complete** — design philosophy, full design-token system (color/typography/iconography/spacing/grid), complete component inventory, every major page/screen fully specified (purpose, layout, components, interaction flow, user journey, UX notes, enterprise benchmarking), and platform-wide behavior (theme engine, multi-language UX, accessibility, states, animation, responsive rules).
- **Not yet done:** no React/HTML/CSS/Tailwind, no component code, no visual mockups/high-fidelity comps — this remains a specification only.

---

# Development Rules

**These rules are mandatory and must be followed by any assistant continuing this project:**

1. **Never redesign completed, approved architecture.** Extend it; do not replace it wholesale.
2. **Never overwrite approved documents.** Create new versioned documents (e.g. `01-architecture-v2.md`) if a revision is genuinely needed, and explain the change before making it.
3. **Always extend the current architecture** rather than introducing parallel/competing designs.
4. **Backend contains all business logic.** The frontend must remain lightweight — presentation and interaction only, no business rules embedded in UI code.
5. **Every request must pass through the OCIF 8-layer pipeline.** No shortcuts that bypass Perception→Capture→Normalization→Enrichment→Synthesis→Cognition→Prescription→Experience.
6. **Always use grounded responses**, following the strict priority chain: Uploaded Project → Project Context → Knowledge Base → RAG → LLM. If nothing is found, state that clearly — never fabricate.
7. **Always use the uploaded Project Context** for any project-specific answer; never generate generic, non-adapted explanations for a specific project.
8. **No hardcoded prompts or templates in code.** All prompts and documentation/diagram/image templates live in the Prompt Library / Template Library (versioned, file-based, DB-indexed).
9. **Never generate placeholder implementations, fake data, or random code** once implementation begins.
10. **Work in phases; stop after each phase** for explicit approval before proceeding, per the project's established workflow.
11. **The OCIF framework has exactly 8 layers.** Never remove or simplify it.

---

# Completed Work

- [x] Phase 1 — Architecture (approved)
- [x] Phase 1 (revised) — Master Blueprint (approved)
- [x] Phase 1.1 — Database Design Specification (approved)
- [x] Phase 1.2 — API Specification (approved)
- [x] Phase 1.3 — Frontend UX/UI Specification (approved)
- [x] Phase 1.4.1 — OCIF Layer 1 (Perception) Specification (approved)
- [x] Phase 1.4.2 — OCIF Layer 2 (Capture) Specification (approved)
- [x] Phase 1.4.3 — OCIF Layer 3 (Normalization) Specification (approved)
- [x] PROJECT_HANDOFF.md — this document (kept current after every phase approval)
- [x] PROJECT_MANIFEST.md — document/artifact inventory (introduced at Phase 1.4.3)
- [x] PROJECT_CHANGELOG.md — chronological change record (introduced at Phase 1.4.3)

---

# Pending Work

- [ ] Phase 1.4.4 — OCIF Layer 4 (Enrichment) Specification (in progress)
- [ ] Phase 1.4.5 — OCIF Layer 5 (Synthesis) Specification
- [ ] Phase 1.4.6 — OCIF Layer 6 (Cognition) Specification
- [ ] Phase 1.4.7 — OCIF Layer 7 (Prescription) Specification
- [ ] Phase 1.4.8 — OCIF Layer 8 (Experience) Specification
- [ ] Security Architecture (authN/authZ hardening, secrets management, data isolation audit, threat model)
- [ ] Phase 2 — Backend Implementation (FastAPI routes, SQLAlchemy models + Alembic migrations, Claude API client wrapper, parsers, OCIF layer modules, all engines)
- [ ] Phase 3 — Frontend Implementation (React/Next.js build of the UX spec)
- [ ] Phase 4 — AI (prompt authoring per layer/section, RAG pipeline wiring, few-shot examples population in `OCIFLayerRepository`)
- [ ] Phase 5 — Documentation Engine content population (real templates/prompts written and tested per layer)
- [ ] Phase 6 — Testing (unit, integration, regression via stored `examples`, load testing for async generation pipeline)
- [ ] Phase 7 — Deployment (Docker Compose → cloud IaC on AWS/Azure/GCP)

---

# Immediate Next Task

**Next document to be created:** **OCIF Layer 4 — Enrichment Specification** (`09-ocif-layer4-enrichment-specification.md`), covering domain/industry/project-type/programming-language/framework/database/API/module/architecture-pattern/sensor/device/business-goal detection, metadata & context enrichment, confidence scoring, and knowledge-tag generation — classification and metadata-attachment only, no reasoning/prediction/recommendation (that begins at Layer 5 — Synthesis / Layer 6 — Cognition).

Once Layers 4–8 are all specified and approved, the recommended follow-on document remains a **Security Architecture Specification** (authentication/authorization hardening, secrets management strategy for multi-provider API keys, tenant/session data isolation guarantees, and a threat model), before backend implementation begins — this recommendation is advisory and unchanged from the prior version of this document.

---

# Long-Term Roadmap

```
Phase 1     Architecture                              ✅ Approved
Phase 1.1   Database Design                           ✅ Approved
Phase 1.2   API Specification                         ✅ Approved
Phase 1.3   Frontend UX/UI Specification               ✅ Approved
Phase 1.4   OCIF Layer Knowledge Repository (8 layers)
            1.4.1  Layer 1 - Perception                ✅ Approved
            1.4.2  Layer 2 - Capture                   ✅ Approved
            1.4.3  Layer 3 - Normalization              ✅ Approved
            1.4.4  Layer 4 - Enrichment                ⬜ In progress
            1.4.5  Layer 5 - Synthesis                 ⬜ Not started
            1.4.6  Layer 6 - Cognition                 ⬜ Not started
            1.4.7  Layer 7 - Prescription               ⬜ Not started
            1.4.8  Layer 8 - Experience                 ⬜ Not started
Phase 1.5   Security Architecture (recommended)        ⬜ Not started
Phase 2     Backend Implementation                     ⬜ Not started
Phase 3     Frontend Implementation                     ⬜ Not started
Phase 4     AI (prompts, RAG wiring, few-shot examples)  ⬜ Not started
Phase 5     Documentation Engine content population      ⬜ Not started
Phase 6     Testing                                      ⬜ Not started
Phase 7     Deployment (local → cloud)                    ⬜ Not started
```

---

# Decisions Taken

| Decision | Rationale |
|---|---|
| Backend: Python + FastAPI | Async-native, Pydantic-first, strong OpenAPI auto-docs support |
| LLM provider: Anthropic Claude API | Chosen by project owner; used for Cognition, Enrichment, Knowledge/RAG synthesis, and image-prompt refinement (not image rendering itself) |
| Deployment: local-first, cloud later | Chosen by project owner; Docker Compose locally, AWS/Azure/GCP migration path preserved via infrastructure-layer abstraction only |
| Single Postgres instance + pgvector | Avoids a second database system for local-first simplicity; kept swappable behind a vector-store adapter interface |
| Clean Architecture (domain/application/infrastructure/interface) | Keeps OCIF pipeline and providers (LLM, image, vector store) swappable without touching business logic |
| Prompts and templates are data, not code | Enables non-deploy tuning, versioning, A/B testing, and full audit trails |
| Documentation is a two-stage fill (template structure + prompt-driven content) | Separates "what sections exist" from "how each section is worded," so either can change independently |
| Grounding priority is a strict, ordered chain with explicit no-data fallback | Enterprise/engineering trust requirement — never fabricate technical facts |
| Multi-language responses are generated directly in-language, not translated after the fact | Preserves correct handling of technical terms in code-mixed languages like Tanglish |
| Context switching uses a dedicated pointer table, never overwrites history | Enables both automatic (new upload) and explicit (named) project switching without losing audit history |
| Frontend: dark-first, restrained accent color, glassmorphism only on overlays | Matches enterprise/industrial register (Siemens/EcoStruxure/AWS) rather than generic consumer-AI-app aesthetics |
| Image generation is multi-provider via a Strategy pattern | Avoids vendor lock-in; Claude explicitly is not used as an image renderer |

---

# Risks

- **Multi-provider image generation** introduces external dependency risk (rate limits, cost, inconsistent output quality across GPT Image/DALL·E/Stable Diffusion/Midjourney) — needs a clear fallback/error UX (already specified) but real-world reliability is unproven until Phase 2/3.
- **Mermaid-to-PNG/PDF rendering pipeline** depends on a headless Chromium/mmdc component — this is an operational dependency (extra container, potential memory/CPU cost) not yet load-tested.
- **31-section documentation generation per layer** is a large number of sequential LLM calls per document — latency and cost at scale need validation; the Continuation System mitigates context-limit failures but not raw cost/time.
- **Grounding chain correctness** depends entirely on retrieval quality (embedding model choice, chunking strategy) — poor chunking could silently degrade "grounded" answers into effectively ungrounded ones without obvious failure signals.
- **Multi-tenancy isolation** (org_id scoping) is designed but not yet security-tested; this is the reason a Security Architecture phase is recommended before Phase 2.
- **No formal authentication/authorization implementation exists yet** — the API spec assumes JWT + RBAC, but token issuance, rotation, and revocation mechanics need hardening before production use.
- **Vector index tuning** (HNSW vs IVFFlat, dimension size matching the chosen embedding model) is deferred to implementation — wrong choices here can cause poor retrieval recall.

---

# Future Improvements

- Org-level custom branding in the Theme Engine (beyond dark/light/system).
- Fine-grained per-document/per-layer sharing and permissions (currently only org/session-level isolation is designed).
- Real-time collaborative viewing of generated documentation (multiple users on one document simultaneously).
- Automated regression testing of OCIF layer outputs using the `examples` field already reserved in `OCIFLayerRepository`.
- Support for additional image providers beyond the initial four, via the existing Strategy-pattern interface.
- Expanded language support beyond the initial seven (English, Tamil, Tanglish, Hindi, Malayalam, Kannada, Telugu, mixed).
- Offline/edge deployment mode for industrial/IoT use cases where cloud connectivity is restricted.

---

# Resume Instructions

**If you are an AI assistant picking up this project, follow these steps exactly, in order:**

1. **Read this entire document first**, before reading or acting on anything else the user says.
2. **Read `PROJECT_MANIFEST.md` and `PROJECT_CHANGELOG.md` next**, for the current document inventory and change history, then **read every approved document in full**, in this order, before writing any code or making any design suggestion:
   - `01-architecture.md`
   - `02-master-blueprint.md`
   - `03-database-design.md`
   - `04-api-specification.md`
   - `05-frontend-ux-spec.md`
   - `06-ocif-layer1-perception-specification.md`
   - `07-ocif-layer2-capture-specification.md`
   - `08-ocif-layer3-normalization-specification.md`
   - (and any further `0N-ocif-layerX-*-specification.md` files present, per `PROJECT_MANIFEST.md`)
3. **Do not treat any approved document as a draft.** They are final unless the project owner explicitly asks for a revision. If you believe something should change, propose the change and explain why — do not silently alter approved architecture.
4. **Confirm the current phase** against the "Pending Work" checklist above before starting any new work. Do not skip ahead (e.g. do not start Phase 2 backend code while Phase 1.4 Security Architecture is still pending) unless the project owner explicitly instructs you to.
5. **Follow the project's phased workflow discipline:** complete one phase, explain what was done, and wait for explicit approval before starting the next. This has been the working pattern for the entire project so far and must continue.
6. **Never hardcode prompts or templates.** If implementing anything in the Documentation, Diagram, Image, or Knowledge engines, route all prompt/template content through the Prompt Library / Template Library design (§ Backend Architecture) — never inline a prompt string in application code.
7. **Preserve the grounding chain exactly as specified**: Uploaded Project → Project Context → Knowledge Base → RAG → LLM, with an explicit "no grounded information available" fallback. Do not weaken this to improve apparent helpfulness.
8. **Preserve the OCIF 8-layer pipeline exactly as specified.** Do not merge, remove, reorder, or simplify layers.
9. **When implementation eventually begins**, follow the Clean Architecture dependency rule (infrastructure → application → domain, never reversed) and the exact folder structure in this document unless the project owner approves a change.
10. **Update this document** (`PROJECT_HANDOFF.md`) whenever a phase is completed and approved, so it always reflects true current status for the next assistant or session.
11. **If context is lost or a new session/tool is starting cold**, this file plus `PROJECT_MANIFEST.md`, `PROJECT_CHANGELOG.md`, and every approved document listed in step 2 above are sufficient to fully reconstruct project intent, decisions, and next steps — there should be no need to ask the user to re-explain the project from scratch.
12. **Update `PROJECT_MANIFEST.md` and `PROJECT_CHANGELOG.md`**, in addition to this file, whenever a phase/document is completed and approved.

---

*End of PROJECT_HANDOFF.md*
