# PROJECT_MANIFEST.md

> **Structured inventory of every architecture artifact produced for the OCIF AI Platform.**
> This is the machine-checkable companion to `PROJECT_HANDOFF.md` (narrative/onboarding) and `PROJECT_CHANGELOG.md` (chronological history). If a file exists in this project, it is listed here with its status, dependencies, and the `OCIFLayerRepository`/schema rows it feeds.

---

## 1. Manifest Metadata

| Field | Value |
|---|---|
| **Project** | OCIF AI Platform (Octagonal Cognitive Intelligence Framework) |
| **Manifest Version** | 1.0 |
| **Corresponds to Project Version** | 0.1.4.3-architecture |
| **Last Updated** | Layer 3 (Normalization) approval |
| **Total Approved Documents** | 9 (incl. this manifest and the changelog) |

---

## 2. Document Inventory

| ID | Filename | Title | Phase | Status | Depends On |
|---|---|---|---|---|---|
| 00 | `PROJECT_HANDOFF.md` | Project Handoff (narrative continuity doc) | — | Living document | All approved documents |
| 00a | `PROJECT_MANIFEST.md` | Project Manifest (this file) | — | Living document | All approved documents |
| 00b | `PROJECT_CHANGELOG.md` | Project Changelog | — | Living document | All approved documents |
| 01 | `01-architecture.md` | Architecture | Phase 1 | ✅ Approved | None (foundational) |
| 02 | `02-master-blueprint.md` | Master Blueprint | Phase 1 (revised) | ✅ Approved | 01 |
| 03 | `03-database-design.md` | Database Design | Phase 1.1 | ✅ Approved | 01, 02 |
| 04 | `04-api-specification.md` | API Specification | Phase 1.2 | ✅ Approved | 01, 02, 03 |
| 05 | `05-frontend-ux-spec.md` | Frontend UX Specification | Phase 1.3 | ✅ Approved | 02, 04 |
| 06 | `06-ocif-layer1-perception-specification.md` | OCIF Layer 1 — Perception | Phase 1.4.1 | ✅ Approved | 01–05 |
| 07 | `07-ocif-layer2-capture-specification.md` | OCIF Layer 2 — Capture | Phase 1.4.2 | ✅ Approved | 01–06 |
| 08 | `08-ocif-layer3-normalization-specification.md` | OCIF Layer 3 — Normalization | Phase 1.4.3 | ✅ Approved | 01–07 |
| 09 | `09-ocif-layer4-enrichment-specification.md` | OCIF Layer 4 — Enrichment | Phase 1.4.4 | ⬜ In progress | 01–08 |
| 10 | `10-ocif-layer5-synthesis-specification.md` | OCIF Layer 5 — Synthesis | Phase 1.4.5 | ⬜ Not started | 01–09 |
| 11 | `11-ocif-layer6-cognition-specification.md` | OCIF Layer 6 — Cognition | Phase 1.4.6 | ⬜ Not started | 01–10 |
| 12 | `12-ocif-layer7-prescription-specification.md` | OCIF Layer 7 — Prescription | Phase 1.4.7 | ⬜ Not started | 01–11 |
| 13 | `13-ocif-layer8-experience-specification.md` | OCIF Layer 8 — Experience | Phase 1.4.8 | ⬜ Not started | 01–12 |
| 14 | *(unnamed)* | Security Architecture Specification | Phase 1.5 (advisory) | ⬜ Not started | 01–13 |

Numbering convention: two-digit prefix reflects creation order, not folder location. All documents live logically under `docs/architecture/` per `01-architecture.md` §6 / `PROJECT_HANDOFF.md`'s current folder structure.

---

## 3. OCIF 8-Layer Repository Manifest

Tracks the `OCIFLayerRepository` row each specification is designed to populate, per `03-database-design.md` §6.1.

| `layer_number` | Layer Name | Spec Document | Spec Status | `documentation_template_id` target |
|---|---|---|---|---|
| 1 | Perception | `06-ocif-layer1-perception-specification.md` | ✅ Approved | `layer1_perception.md.j2` |
| 2 | Capture | `07-ocif-layer2-capture-specification.md` | ✅ Approved | `layer2_capture.md.j2` |
| 3 | Normalization | `08-ocif-layer3-normalization-specification.md` | ✅ Approved | `layer3_normalization_v2.md.j2` |
| 4 | Enrichment | `09-ocif-layer4-enrichment-specification.md` | ⬜ In progress | `layer4_enrichment.md.j2` |
| 5 | Synthesis | *(not yet created)* | ⬜ Not started | `layer5_synthesis.md.j2` |
| 6 | Cognition | *(not yet created)* | ⬜ Not started | `layer6_cognition.md.j2` |
| 7 | Prescription | *(not yet created)* | ⬜ Not started | `layer7_prescription.md.j2` |
| 8 | Experience | *(not yet created)* | ⬜ Not started | `layer8_experience.md.j2` |

Each row, once its spec is approved, is loaded in full into `OCIFLayerRepository` (fields: `objective`, `rules`, `examples`, plus join-table rows into `OCIFLayerPromptMap`, `OCIFLayerDiagramTemplateMap`, `OCIFLayerImagePromptMap` — per `03-database-design.md` §6).

---

## 4. Engine Specification Manifest

Per `PROJECT_HANDOFF.md`'s Backend Architecture (Engine Summary). Engine *architecture* is specified across `01`–`05`; engine *content* (concrete prompts/templates) is populated per-layer as each OCIF layer spec is approved.

| Engine | Architecture Specified In | Per-Layer Content Populated By |
|---|---|---|
| OCIF Engine | `01`, `02` | Each layer spec (`06`–`13`), §21–24 of each |
| Project Context Engine | `01`, `03` | Layers 2–4 specs (Capture creates, Normalization chunks, Enrichment classifies) |
| Documentation Engine | `02` §3 | Each layer spec's §22 (Documentation Template Design) |
| Diagram Engine | `02` §5 | Each layer spec's §23 (Diagram Template Design) |
| Image Engine | `02` §4 | Each layer spec's §24 (Image Prompt Template) |
| Knowledge Engine | `02` §8 | Not layer-specific (separate admin ingestion path) |
| Grounding Engine (RAG) | `01` §5, `02` §8 | Layers 3 (chunks/embeddings) and 5 (Synthesis — retrieval merge) |
| Language Engine | `02` §6 | Cross-cutting; not a single layer's spec |
| Context Switching Engine | `02` §7 | Not layer-specific (pointer table, deferred updates) |

---

## 5. Database Schema Coverage Manifest

Every table in `03-database-design.md` and which document(s) currently define its producers/consumers:

| Schema | Tables | Producer/Consumer Defined In |
|---|---|---|
| `identity` | `Organization`, `User`, `UserSession` | `03` (schema only; no layer writes these) |
| `project_ctx` | `ProjectContext`, `ProjectSourceFile`, `ProjectContextChunk`, `ActiveContextPointer` | `03` (schema); `06`–`08` (producers so far: Layer 2 creates `ProjectContext`/`ProjectSourceFile`, Layer 3 creates `ProjectContextChunk`); Layer 4 (pending) will update `ProjectContext`'s classification fields and `ActiveContextPointer` |
| `knowledge` | `KnowledgeDocument`, `KnowledgeChunk` | `03` (schema); `02` §8.4 (admin ingestion path, outside OCIF pipeline) |
| `prompt_lib` | `PromptTemplate` | `03` (schema); read by every layer spec's §21 |
| `template_lib` | `TemplateRegistry`, `OCIFLayerRepository` (+3 join tables) | `03` (schema); read/written per layer spec's §19, §22–24 |
| `diagram` | `DiagramArtifact` | `03` (schema); written per layer spec's §23 output |
| `image` | `ImageArtifact`, `ImageProviderConfig` | `03` (schema); written per layer spec's §24 output |
| `generation` | `GenerationSession` | `03` (schema); `01` §8 (Continuation System) |
| `conversation` | `ConversationMessage` | `03` (schema); Layer 1/Layer 8 (not yet detailed beyond `06`) |

---

## 6. API Endpoint Coverage Manifest

Endpoint groups from `04-api-specification.md` §1–20, cross-referenced against which layer specs currently document their consumption:

| Endpoint Group | Consumed/Documented By |
|---|---|
| Auth, Authorization/RBAC, Users | `04` only (no layer spec references these directly yet) |
| Project Upload, Project Context | `06` (entry point), `07` §20, `08` §20 |
| OCIF Layers | `06`–`08` §20 (per-layer mapping); `09`+ pending |
| Documentation, Diagram, Image | `04` (contract); each layer spec's §22–24 (content design) |
| Knowledge Base, Prompt Library, Template Library | `04` (contract); `02` §8 (architecture) |
| Context Switching, Language | `04` (contract); `02` §6–7 (architecture) |
| Chat, Generation Sessions, Export, Admin, Health | `04` only (no layer spec references these directly yet) |

---

## 7. Cross-Reference Integrity Notes

- All Mermaid diagram IDs, section numbers, and rule references in `06`, `07`, `08` have been verified against `01`–`05` and against each other; no renumbering or content contradiction was introduced when `08` was added.
- `04-api-specification.md` §23's Upload Flow Diagram (`CAP->>NORM`, `NORM->>DB: ProjectContextChunk rows + embeddings`) is treated as binding by `08` — this manifest records that dependency explicitly so future layer specs (5 onward, which read from `ProjectContextChunk`) remain consistent with it.
- The `page_ref`/`section_ref` future-schema-addition flagged in both `07` §36 and `08` §36 is **not yet adopted** — this manifest tracks it as a single open extension point shared by two documents, to avoid it being proposed twice independently in later layers.

---

## 8. Status

This manifest is updated whenever a new document is created or an existing document's status changes. It does not restate content — only inventory, status, and cross-document dependency tracking. See `PROJECT_CHANGELOG.md` for the chronological record of what changed and when.
