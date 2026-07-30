# PROJECT_CHANGELOG.md

> Chronological record of changes to the OCIF AI Platform's architecture documentation.
> Format loosely follows *Keep a Changelog*, adapted for a documentation-only, pre-implementation project. Versions use the scheme `0.<phase>.<subphase>-architecture` established in `PROJECT_HANDOFF.md`.
> Entries for work completed before this changelog was introduced are marked **[retrospective]** — no original timestamps were recorded for these, since no changelog existed at the time.

---

## [0.1.4.5-architecture] — Layer 5 approved

### Approved
- **OCIF Layer 5 — Synthesis Specification** (`10-ocif-layer5-synthesis-specification.md`) — full 37-section spec: grounding context assembly across Project Content (`ProjectContextChunk` vector search), Project Context (direct `ProjectContext` read), and Knowledge Base (`KnowledgeChunk` vector search filtered by Layer 4's `knowledge_tags` against `industry_tags`); tier-strict retrieval ranking; context fusion; near-duplicate elimination with audit trail; evidence collection/traceability; source attribution; cross-tier conflict detection and priority-order resolution (recorded, not silently discarded); token-budget context prioritization; bundle-level grounding confidence scoring; retrieval metadata generation for the frontend's Grounding Sources transparency strip; and final `GroundedContext` creation — assembly and evidence-merging only, with no reasoning, prediction, explanation, recommendation, or output-format decisions.

### Changed
- `PROJECT_HANDOFF.md`:
  - Version bumped `0.1.4.4-architecture` → `0.1.4.5-architecture`.
  - Current Development Phase updated to reflect Layers 1–5 approved, Layer 6 in progress.
  - Architecture Status table updated: OCIF Layer 5 row marked Approved, OCIF Layer 6 row marked In progress.
  - Per-Document Detail section extended with an entry for document 10.
  - Completed Work / Pending Work checklists updated (Phase 1.4.5 moved to completed).
  - Immediate Next Task updated: OCIF Layer 6 (Cognition) Specification, now in progress.
  - Long-Term Roadmap updated: Phase 1.4.5 marked Approved, Phase 1.4.6 marked In progress.
  - Resume Instructions' architecture reading list extended with `10-ocif-layer5-synthesis-specification.md`.
- `PROJECT_MANIFEST.md`:
  - Manifest metadata updated: corresponds to `0.1.4.5-architecture`, last updated at Layer 5 approval, total approved documents 10 → 11.
  - Document Inventory and OCIF 8-Layer Repository Manifest tables updated: row 10 (Layer 5) marked Approved; row 11 (Layer 6) marked In progress.
  - Engine Specification Manifest updated: Project Context Engine and Grounding Engine (RAG) rows reflect Layer 5's approval and Layer 6's in-progress status.
  - Database Schema Coverage Manifest updated: `project_ctx` and `knowledge` rows record Layer 5 as the first approved reader of `ProjectContext`/`ProjectContextChunk` (for grounding) and of `KnowledgeChunk`/`industry_tags`.
  - API Endpoint Coverage Manifest updated: OCIF Layers row reflects five approved layer specs.
  - Cross-Reference Integrity Notes extended with Layer 5's diagram/section verification, confirmation of the Layer 4→5 `knowledge_tags` hand-off, its `GroundedContextCache`/persistence future-extension flags, and the tier-strict ranking boundary Layer 6 must reason over as given rather than re-rank.

### No changes made to
- `01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`, `04-api-specification.md`, `05-frontend-ux-spec.md`, `06-ocif-layer1-perception-specification.md`, `07-ocif-layer2-capture-specification.md`, `08-ocif-layer3-normalization-specification.md`, `09-ocif-layer4-enrichment-specification.md` — all remain exactly as previously approved.

---

## [0.1.4.4-architecture] — Layer 4 approved

### Approved
- **OCIF Layer 4 — Enrichment Specification** (`09-ocif-layer4-enrichment-specification.md`) — full 37-section spec: domain/industry/project-type classification via rule-based heuristics cross-checked with Claude; programming-language/framework detection built on Capture's structural signatures; database/API/module detection; industrial sensor/device detection (empty-by-default, evidence-gated); factual business-goal summarization; per-detection (not blanket) confidence scoring with `low_confidence_flags`; knowledge-tag generation for later Knowledge Base matching; finalization of `ProjectContext.status` (`ready`/`failed`) and `ActiveContextPointer`, resolving the deferral both Layer 2 and Layer 3 left open.

### Changed
- `PROJECT_HANDOFF.md`:
  - Version bumped `0.1.4.3-architecture` → `0.1.4.4-architecture`.
  - Current Development Phase updated to reflect Layers 1–4 approved, Layer 5 in progress.
  - Architecture Status table updated: OCIF Layer 4 row marked Approved, OCIF Layer 5 row marked In progress.
  - Per-Document Detail section extended with an entry for document 09.
  - Completed Work / Pending Work checklists updated (Phase 1.4.4 moved to completed).
  - Immediate Next Task updated: OCIF Layer 5 (Synthesis) Specification, now in progress.
  - Long-Term Roadmap updated: Phase 1.4.4 marked Approved, Phase 1.4.5 marked In progress.
  - Resume Instructions' architecture reading list extended with `09-ocif-layer4-enrichment-specification.md`.
- `PROJECT_MANIFEST.md`:
  - Manifest metadata updated: corresponds to `0.1.4.4-architecture`, last updated at Layer 4 approval, total approved documents 9 → 10.
  - Document Inventory and OCIF 8-Layer Repository Manifest tables updated: row 09 (Layer 4) marked Approved; row 10 (Layer 5) marked In progress.
  - Engine Specification Manifest updated: Project Context Engine and Grounding Engine (RAG) rows reflect Layer 4's approval and Layer 5's in-progress status.
  - Database Schema Coverage Manifest updated: `project_ctx` row records Layer 4 as the producer of `ProjectContext`'s classification fields, `status`, and `ActiveContextPointer`; `knowledge` row notes Layer 5 will be the first consumer of `KnowledgeChunk`/`industry_tags`.
  - API Endpoint Coverage Manifest updated: OCIF Layers row reflects four approved layer specs.
  - Cross-Reference Integrity Notes extended with Layer 4's diagram/section verification, its `EnrichmentMetadata` future-extension flag, and the `knowledge_tags` hand-off to Layer 5.

### No changes made to
- `01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`, `04-api-specification.md`, `05-frontend-ux-spec.md`, `06-ocif-layer1-perception-specification.md`, `07-ocif-layer2-capture-specification.md`, `08-ocif-layer3-normalization-specification.md` — all remain exactly as previously approved.

---

## [0.1.4.3-architecture] — Layer 3 approved; project-management docs introduced

### Added
- `PROJECT_MANIFEST.md` — structured document/artifact inventory, introduced at this version.
- `PROJECT_CHANGELOG.md` — this file, introduced at this version.

### Changed
- `PROJECT_HANDOFF.md`:
  - Version bumped `0.1.0-architecture` → `0.1.4.3-architecture`.
  - Current Development Phase updated to reflect Layers 1–3 approved, Layer 4 in progress.
  - Architecture Status table extended with rows for OCIF Layers 1–8 (Layers 1–3 marked Approved, 4 In progress, 5–8 Not started).
  - Per-Document Detail section extended with entries for documents 06, 07, 08.
  - Completed Work / Pending Work checklists updated.
  - Immediate Next Task updated: OCIF Layer 4 (Enrichment) Specification.
  - Long-Term Roadmap expanded with Phase 1.4 sub-phases (1.4.1–1.4.8) and Security Architecture renumbered to Phase 1.5.
  - Resume Instructions updated to reference `PROJECT_MANIFEST.md`, `PROJECT_CHANGELOG.md`, and the full (now 8-document) architecture reading list.

### Approved
- **OCIF Layer 3 — Normalization Specification** (`08-ocif-layer3-normalization-specification.md`) — full 37-section spec: text cleaning, semantic chunking (500–800 tokens, ~15% overlap), embedding generation, `ProjectContextChunk` persistence, structural-metadata carry-forward from Layer 2, LLM-light design (single ambiguous-boundary fallback prompt).

### No changes made to
- `01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`, `04-api-specification.md`, `05-frontend-ux-spec.md`, `06-ocif-layer1-perception-specification.md`, `07-ocif-layer2-capture-specification.md` — all remain exactly as previously approved.

---

## [0.1.4.2-architecture] — Layer 2 approved **[retrospective]**

### Approved
- **OCIF Layer 2 — Capture Specification** (`07-ocif-layer2-capture-specification.md`) — full 37-section spec: file ingestion (PDF/DOCX/MD/ZIP/code), ZIP-bomb depth/size guards, `ProjectContext`/`ProjectSourceFile` creation, per-file failure isolation, LLM-light design (single ambiguous-filetype fallback prompt).

---

## [0.1.4.1-architecture] — Layer 1 approved **[retrospective]**

### Approved
- **OCIF Layer 1 — Perception Specification** (`06-ocif-layer1-perception-specification.md`) — full 37-section spec: intent/language/request-type classification, entry-point behavior for every channel (chat, upload, layer-explain, diagram/image generation, project switch, admin action).

---

## [0.1.3-architecture] — Frontend UX Specification approved **[retrospective]**

### Approved
- **Frontend UX Specification** (`05-frontend-ux-spec.md`) — 50-section UX/UI design document: design philosophy, design-token system, full component inventory, every major page/screen, platform-wide behavior patterns.

---

## [0.1.2-architecture] — API Specification approved **[retrospective]**

### Approved
- **API Specification** (`04-api-specification.md`) — complete REST API contract: 20 endpoint groups, ~45 endpoints, 8 flow diagrams.

---

## [0.1.1-architecture] — Database Design approved **[retrospective]**

### Approved
- **Database Design** (`03-database-design.md`) — complete schema across 9 logical schemas, full ER diagram, 4 data-flow diagrams.

---

## [0.1.0-architecture] — Foundational architecture approved **[retrospective]**

### Added
- `PROJECT_HANDOFF.md` created as the project's single source of truth for continuity.

### Approved
- **Architecture** (`01-architecture.md`) — Clean Architecture layering, tech stack, OCIF 8-layer-to-module mapping, folder skeleton, deployment path.
- **Master Blueprint** (`02-master-blueprint.md`) — Documentation/Diagram/Image/Language/Context-Switching/Knowledge Base engines, Prompt Library, Template Library, master Enterprise AI Flow sequence diagram.

---

## Versioning Notes

- Version format: `0.<major-phase>.<sub-phase>-architecture` during the pre-implementation documentation period. E.g. `0.1.4.3-architecture` = Phase 1, sub-phase 4 (OCIF Layer Repository), 3rd layer of that sub-phase approved.
- No document is ever versioned down or re-approved silently — a revision to an approved document would produce a new version entry and, per `PROJECT_HANDOFF.md`'s Development Rules, a new versioned file (e.g. `01-architecture-v2.md`), never an overwrite.
- This changelog will switch to calendar-dated entries once real approval dates are available going forward; historical entries above are ordered but undated by necessity.
