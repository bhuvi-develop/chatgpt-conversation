# AXIOM AI PLATFORM --- Enterprise Backend Intelligence Evolution (Master Implementation Prompt)

## IMPORTANT

-   Analyze the entire repository.
-   Treat the repository as the single source of truth.
-   Do not redesign the architecture.
-   Do not modify the frontend, database schema, APIs, or existing OCIF
    architecture.
-   Only extend the backend.

## Phase 1 --- Project Intelligence Engine

Create a Project Intelligence Engine responsible for: - Industry
Detection - Domain Detection - Business Goal Detection - Engineering
Goal Detection - Technology Detection - Architecture Detection - Entity
Extraction - Actor Extraction - Relationship Building - Workflow
Analysis - Sensor, API, Database and Protocol Detection - Project
Intelligence Model Generation

This engine becomes the first stage of the backend pipeline.

## Phase 2 --- Documentation Intelligence

Integrate the Documentation Engine with the Project Intelligence Model.
Generate documentation from structured project intelligence instead of
raw prompts.

## Phase 3 --- Diagram Intelligence

Integrate the Diagram Engine with the Project Intelligence Model.
Generate project-specific OCIF Layer 1--8 diagrams. Never use one
generic Mermaid template.

## Phase 4 --- Domain Knowledge Engine

Create reusable Domain Knowledge Profiles for: Industrial AIoT, HVAC,
Water Pump, Healthcare, Hospital, Education, Attendance, Banking,
Insurance, Retail, Construction, Energy, Agriculture, Automobile,
Government, Manufacturing, Smart Building and ESG.

## Phase 5 --- Dynamic Planning Engine

Create: - Document Planner - Diagram Planner - Report Planner - Image
Planner - Architecture Planner

Determine required outputs before generation.

## Phase 6 --- Multimodal Project Understanding

Support: - PDF - DOCX - PPTX - Images - ZIP - Source Code - Markdown -
CSV - JSON

All inputs must be converted into one unified Project Intelligence
Model.

## Execution Pipeline

User Input → Input Processing → Project Intelligence Engine → Domain
Knowledge Engine → Planning Engine → Project Intelligence Model →
Existing OCIF Layer 1--8 → Documentation Engine → Diagram Engine → Image
Engine → Response

## Expected Outcome

Every uploaded project should produce: - Different reasoning - Different
documentation - Different diagrams - Different architecture - Different
recommendations

## Implementation Strategy

1.  Analyze repository.
2.  Reuse existing backend modules.
3.  Avoid duplicate logic.
4.  Preserve backward compatibility.
5.  Run regression tests after each phase.
6.  Complete one phase before starting the next.
