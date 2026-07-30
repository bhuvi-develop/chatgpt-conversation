# OCIF AI Platform --- Phase 1.5

## Complete Software Architecture Specification --- Language Engine

**Status:** Architecture / permanent knowledge specification only. No
implementation code (no Python, FastAPI, React, or SQL). Builds on
`01-architecture.md`, `02-master-blueprint.md`, `03-database-design.md`,
`04-api-specification.md`, `05-frontend-ux-spec.md`, the OCIF Layer
Specifications (1-8), and Engines 14--20
(`20-rag-engine-specification.md`).

------------------------------------------------------------------------

## 1. Overview

The **Language Engine** is the enterprise localization and linguistic
processing subsystem of the OCIF AI Platform. It operates at the
system's perimeter (during Layer 1 Perception and Layer 8 Experience) to
ensure seamless multi-lingual interaction. It detects the user's
language, translates non-English inputs into English for core pipeline
processing, and localizes the AI's final grounded outputs back into the
user's preferred language while rigorously preserving technical
engineering terminology.

## 2. Purpose

To dismantle language barriers in enterprise engineering environments,
allowing users across diverse linguistic backgrounds (specifically
English and key Indian regional languages) to interact with complex
architectural AI, without sacrificing the precision of technical
standards and system design vocabulary.

## 3. Objectives

-   Accurately detect the source language and script of incoming user
    queries.
-   Translate regional inputs into English to maximize the semantic
    accuracy of the RAG and Cognition engines.
-   Translate and transliterate final grounded responses back to the
    user's preferred language.
-   Enforce strict terminology preservation (e.g., ensuring "Kubernetes"
    or "Centrifugal Force" is not awkwardly translated into native
    scripts).
-   Adapt the linguistic tone to match the target persona (e.g., formal
    documentation vs. conversational chat).

## 4. Business Need

In global and regional enterprise engineering teams, engineers often
think and communicate in their native language or a hybrid (e.g.,
Tanglish), but technical specifications, codebases, and LLM reasoning
models are predominantly optimized for English. The business requires a
bridge that allows a user to ask a complex architectural question in
Tamil or Hindi and receive a culturally nuanced, grammatically correct
response that correctly retains English technical terms for code and
infrastructure.

## 5. Problem Statement

**Given** that generative AI models often aggressively translate
technical jargon into unusable literal native words, and **given** that
vector similarity search performs poorly on mixed-language queries,
**the Language Engine must** standardize internal processing to English
and handle complex localization at the platform's edges, **without**
altering the grounded facts or corrupting Markdown/Mermaid syntax.

## 6. Responsibilities

  -----------------------------------------------------------------------
  Responsibility                      Excluded Responsibilities (Handled
                                      Elsewhere)
  ----------------------------------- -----------------------------------
  Auto-detecting user input language  Executing vector/semantic search
  and script.                         (RAG Engine).

  Translating queries to English for  Assembling grounded context
  the pipeline.                       (Grounding Engine).

  Rendering final outputs in the      Making architectural decisions
  target language.                    (Cognition Layer).

  Preserving technical terminology    Generating diagrams or images
  via glossaries.                     (Diagram/Image Engines).
  -----------------------------------------------------------------------

## 7. Engine Architecture

The Language Engine follows the Clean Architecture pattern as an
Infrastructure/Application boundary service. It acts as an interceptor.
Layer 1 (Perception) calls it to normalize inputs. Layer 8 (Experience)
calls it to localize outputs. It relies on a specialized LLM routing
strategy and caching mechanisms to provide real-time translation with
near-zero hallucination.

## 8. Internal Modules

-   **`LanguageDetector`**: Analyzes text to determine ISO language
    codes and script types.
-   **`TranslationPipeline`**: Orchestrates bidirectional translation
    (Native \<-\> English).
-   **`TransliterationService`**: Handles script conversions (e.g.,
    Latin-script Tanglish to standard Tamil script, or vice versa).
-   **`TerminologyVault`**: A dictionary/regex manager that shields
    technical terms from translation.
-   **`ToneAdapter`**: Adjusts the formality and politeness markers of
    the target language.
-   **`FormatValidator`**: Ensures translation did not break Markdown,
    JSON, or XML tags.

## 9. Language Detection Pipeline

When a user submits a prompt, the `LanguageDetector` uses a lightweight,
fast heuristic model (e.g., fastText or a local Naive Bayes classifier)
to instantly identify the language. It calculates a confidence score. If
the score is below the threshold, it defaults to English or prompts the
user for clarification.

## 10. Language Classification

Detected languages are classified into standard internal enums (e.g.,
`Lang.EN`, `Lang.TA`, `Lang.HI`). The engine also classifies the
*script* (e.g., distinguishing between Hindi written in Devanagari
vs. Hindi written in Latin script/Hinglish).

## 11. Auto Language Detection

Operates continuously in the background. If a user switches from English
to Malayalam mid-session, the engine detects the shift at Layer 1,
updates the active session's language preference in the
`ProjectContext`, and seamlessly transitions the conversation without
requiring explicit UI toggles.

## 12. Translation Pipeline

1.  **Pre-processing:** Extract code blocks, Mermaid diagrams, and UUID
    tags to a safe-hold dictionary.
2.  **Terminology Masking:** Replace known technical terms with
    non-translatable tokens (e.g., `[[TERM_01]]`).
3.  **Translation:** Pass the masked text to the LLM optimized for
    translation.
4.  **Post-processing:** Re-inject the safe-hold code blocks and unmask
    the technical terms.

## 13. Transliteration Pipeline

Specifically designed for hybrid languages. If a user types "Idhu eppadi
scale aagum?" (How will this scale?), the pipeline recognizes this as
Tanglish. It uses phoneme mapping to understand the underlying Tamil
grammar while retaining the English technical word "scale."

## 14. Technical Terminology Management

The most critical feature of the engine. It utilizes Named Entity
Recognition (NER) and enterprise glossaries to identify components, API
names, protocols, and database types. These entities are explicitly
excluded from translation matrices to ensure engineering accuracy.

## 15. Engineering Vocabulary Preservation

The engine connects to the Knowledge Engine to load industry-specific
glossaries. If the project is "Maritime Shipping," terms like "Ballast
Water" are flagged to ensure they are rendered correctly according to
maritime engineering standards in the target language, rather than
translated literally.

## 16. Tone Management

Different languages encode respect and formality differently (e.g., the
use of "Aap" vs "Tum" in Hindi, or "Neenga" vs "Nee" in Tamil). The
`ToneAdapter` uses the user's role profile to ensure the generated text
respects enterprise hierarchical norms without sounding robotic.

## 17. Style Profiles

Maintains linguistic profiles:

-   **Conversational:** Used for chatbot UI (Layer 8 Experience).
-   **Authoritative:** Used for generating final documentation
    (Documentation Engine).
-   **Diagnostic:** Short, imperative sentences used for
    troubleshooting.

## 18. Localization

Beyond literal translation, the engine localizes date formats, currency
(if applicable to billing microservices), and measurement units (Metric
vs. Imperial) based on the user's organization settings stored in the
database.

## 19. Response Formatting

The engine must guarantee that structural artifacts survive translation.
If Layer 6 generates `<answer>123</answer>`, the Language Engine ensures
the XML tags remain untouched in the target language output.

## 20. Grammar Validation

A final pass utilizing a lightweight language-specific ruleset to ensure
the reconstructed sentence (which mixes native grammar with English
technical nouns) is syntactically sound and flows naturally to a native
speaker.

## 21. Multi-language Support

The engine is architected to support dynamic addition of languages, but
Phase 1.5 explicitly mandates support for a specific matrix of seven
languages critical to the target demographic.

## 22. English Support

The baseline language. Core reasoning (Layers 2-7) operates entirely in
English. If the user interacts in English, the Translation and
Transliteration pipelines act as transparent pass-throughs, adding zero
latency.

## 23. Tamil Support

Full support for native Tamil script (தமிழ்). The engine maps complex
technical concepts to established Tamil engineering vocabulary where
appropriate, while preserving strict technical nouns (e.g., "API") in
English.

## 24. Tanglish Support

Support for Tamil written in Latin script mixed with English words. The
engine treats this as a first-class language input, parsing the phonetic
intent and translating it to standard English for core platform
processing, then rendering the response back in conversational Tanglish.

## 25. Hindi Support

Support for Devanagari script (हिन्दी) and Hinglish. Applies strict
formal/informal tone management based on enterprise context.

## 26. Telugu Support

Support for native Telugu script (తెలుగు). Utilizes specific localized
glossaries for IT and infrastructure terminology prevalent in the
Hyderabad tech sector.

## 27. Kannada Support

Support for native Kannada script (ಕನ್ನಡ). Utilizes glossaries aligned
with the aerospace and heavy engineering sectors prevalent in the
region.

## 28. Malayalam Support

Support for native Malayalam script (മലയാളം). Ensures complex
agglutinative grammar structures correctly encapsulate English technical
nouns without breaking sentence flow.

## 29. Project Context Engine Integration

Queries the active user session to determine the default or currently
active language and localization settings, ensuring continuity if the
conversation spans multiple days.

## 30. Knowledge Engine Integration

Fetches the `org_id` specific glossaries and translation dictionaries.
If an enterprise has a specific approved translation for a proprietary
company process, the Language Engine enforces it.

## 31. Grounding Engine Integration

The Language Engine translates queries *before* RAG retrieval, allowing
the RAG/Grounding Engines to operate exclusively on English embeddings.
The final Grounded Context Bundle remains in English; localization
happens *after* synthesis.

## 32. RAG Engine Integration

By translating all queries to English at Layer 1, the RAG Engine
achieves maximum vector similarity against standard technical
documentation, bypassing the degradation usually associated with
cross-lingual embeddings.

## 33. Documentation Engine Integration

When the user requests an architecture document in Tamil, the core
pipeline generates the document in English. The Documentation Engine
then passes the completed document through the Language Engine's
`TranslationPipeline` in batches to produce the localized artifact.

## 34. Diagram Engine Integration

The Language Engine translates diagram titles and narrative
descriptions, but strictly avoids translating Node IDs, variables, or
database table names inside the Mermaid syntax to ensure the rendered
diagram remains technically accurate.

## 35. Image Engine Integration

Image prompts are strictly generated and executed in English to maximize
compatibility with models like Stable Diffusion and DALL-E. The Language
Engine translates only the user-facing captions of the resulting images.

## 36. Validation Rules

-   Must never translate text inside codeblocks (`...`).
-   Must never translate text inside explicit `[[DONT_TRANSLATE]]`
    markers.
-   Must validate that all Markdown links and image references remain
    intact post-translation.

## 37. Error Handling

-   `422 Unprocessable Entity`: The input text contains an unrecognized
    script or encoding error.
-   If translation fails due to LLM timeout, the engine must degrade
    gracefully, falling back to English and appending a localized
    apology message.

## 38. Retry Strategy

Translation relies on LLM endpoints which can experience latency. The
engine employs a circuit breaker with 2 retries (exponential backoff).
Extensive use of Redis caching for repeated phrases minimizes API calls.

## 39. Logging

Logs the `detected_language`, `translation_latency_ms`, and
`cache_hit_ratio`. Strictly scrubs all PII and proprietary technical
data before logging.

## 40. Performance

-   Language Detection: \< 10ms.
-   Pre/Post processing and Terminology Masking: \< 20ms.
-   Translation API overhead is the primary bottleneck; must resolve
    within standard platform generation streaming SLAs.

## 41. Security

Translation must not expose the system to prompt injection. The
`TranslationPipeline` treats the input as untrusted strings, strictly
escaping them before passing them to the translation model, preventing
malicious instructions hidden in regional languages from executing in
the English core.

## 42. Database Mapping

-   `identity.UserPreferences`: Stores `preferred_language`.
-   `conversation.ConversationMessage`: Stores `detected_language`
    alongside the raw text.

## 43. REST API Mapping

-   `POST /api/v1/internal/language/detect`
-   `POST /api/v1/internal/language/translate`
-   `GET /api/v1/internal/language/glossary`

## 44. Folder Structure

``` text
backend/app/engines/language/
├── __init__.py
├── detector.py
├── translation_pipeline.py
├── transliteration_service.py
├── terminology_vault.py
├── format_validator.py
└── exceptions.py
```

## 45. Mermaid Architecture Diagram

``` mermaid
flowchart TB
    subgraph L1["Layer 1: Perception"]
        IN[User Input]
    end

    subgraph LE["Language Engine"]
        DET[LanguageDetector]
        VAULT[TerminologyVault]
        TRANS[TranslationPipeline]
        FMT[FormatValidator]
    end

    subgraph Core["Core OCIF Pipeline (English)"]
        RAG[RAG Engine]
        COG[Cognition Layer]
    end

    subgraph L8["Layer 8: Experience"]
        OUT[User Output]
    end

    IN --> DET
    DET --> VAULT
    VAULT --> TRANS
    TRANS -->|English Query| Core
    Core -->|English Response| TRANS
    TRANS --> FMT
    FMT --> OUT
```

## 46. Mermaid Sequence Diagram

``` mermaid
sequenceDiagram
    participant User
    participant L1 as Perception
    participant LE as Language Engine
    participant Core as Core Pipeline
    participant L8 as Experience

    User->>L1: "Idhu microservices use pannuma?" (Tanglish)
    L1->>LE: detect_and_translate()
    LE->>LE: Detect: Tanglish
    LE->>LE: Mask: [[TERM_1]] = "microservices"
    LE->>LE: Translate: "Will this use [[TERM_1]]?"
    LE-->>L1: "Will this use microservices?" (English)
    L1->>Core: Process request
    Core-->>L8: "Yes, the architecture uses microservices."
    L8->>LE: localize(text, target=Tanglish)
    LE->>LE: Translate & Unmask
    LE-->>L8: "Aama, intha architecture microservices use pannuthu."
    L8-->>User: (Tanglish Response)
```

## 47. Mermaid Component Diagram

``` mermaid
componentDiagram
    component "Language Engine" {
        [LanguageDetector]
        [TranslationPipeline]
        [TerminologyVault]
        [FormatValidator]
    }
    
    [Layer 1] --> [LanguageDetector]
    [LanguageDetector] --> [TranslationPipeline]
    [TerminologyVault] ..> [TranslationPipeline] : Supplies protected terms
    [TranslationPipeline] --> [FormatValidator]
    [FormatValidator] --> [Layer 8]
```

## 48. Mermaid Deployment Diagram

``` mermaid
flowchart LR
    subgraph AppContainer["Application Container (FastAPI)"]
        LE[Language Engine]
    end

    subgraph CacheTier["Redis"]
        TRANS_CACHE[Translation Cache]
    end

    subgraph AI_Tier["LLM Provider"]
        CLAUDE[Translation Model]
    end

    LE <--> TRANS_CACHE
    LE <--> CLAUDE
```

## 49. Industrial Examples

The Language Engine empowers regional factory floors and local
engineering teams to interact with high-level AI without language
friction.

## 50. Water Pump Example

**Scenario:** A technician in Madurai asks, "Pump-oda vibration level
adhigama irukku, enna panrathu?" (Tamil/Tanglish). **Action:** The
engine detects Tanglish, masks "vibration level", translates the query
to "The pump's vibration level is high, what should I do?". The core
pipeline retrieves maintenance SOPs. The response is translated back:
"Vibration level-a check pannunga. Bearing theinju irukkalam."

## 51. Smart Building Example

**Scenario:** Generating a Layer 2 Capture summary for a building
management system in Hindi. **Action:** The documentation is generated
in English. The Language Engine translates the prose into formal Hindi
but uses the `TerminologyVault` to ensure terms like "HVAC", "IoT
Sensors", and "BACnet Protocol" remain exactly as written in English,
preventing confusing literal translations like "हीटिंग वेंटिलेशन और एयर
कंडीशनिंग".

## 52. Attendance System Example

**Scenario:** Developer asks about database sharding in Malayalam.
**Action:** The Language Engine preserves code blocks. The explanation
of horizontal scaling is delivered in Malayalam, but the SQL query
`CREATE TABLE attendance PARTITION BY RANGE` remains untouched by the
`FormatValidator`, guaranteeing the code is ready to copy-paste.

## 53. Future Extensions

-   **Voice-to-Text Integration:** Extending the Language Engine with a
    whisper-based transcription port to handle audio queries in regional
    languages.
-   **Custom Org Dictionaries:** Creating a UI where enterprise admins
    can manually add proprietary company acronyms to the
    `TerminologyVault` to prevent them from ever being translated.

## 54. Best Practices

-   **English Core:** Always isolate translation to the perimeter.
    Forcing LLMs to reason deeply about complex engineering
    architectures in regional languages degrades logic performance;
    reason in English, present in local.
-   **Mask Before Translating:** Never rely on the LLM to "figure out"
    what a technical word is during translation. Explicitly mask it out
    using Regex/NER before the prompt is sent.
-   **Cache Aggressively:** UI strings, common greetings, and error
    messages should always hit the Redis cache to save token costs and
    latency.

## 55. Common Mistakes

-   **Translating Markdown:** Accidentally translating
    `[Click Here](url)` into a broken link syntax because the LLM
    altered the brackets. `FormatValidator` must prevent this.
-   **Ignoring Script:** Assuming that because the language is Tamil,
    the user wants Tamil script, when they actually typed in Latin
    script (Tanglish) and expect the response in the same format.
-   **Over-Translating:** Translating a variable name `user_id` to a
    local language string, instantly breaking the user's codebase.

## 56. Interview Questions

1.  **Why doesn't the RAG Engine just embed the regional language
    queries directly?** *Answer:* Because the underlying technical
    knowledge bases and source code are almost entirely in English. If
    you embed a Tamil query, it will land far away in vector space from
    the English documentation. Translating the query to English first
    ensures highly accurate semantic matching.
2.  **How does the Terminology Vault protect code?** *Answer:* Through
    pre-processing. The Language Engine strips out all markdown code
    blocks (`...`) and replaces them with placeholder UUIDs *before*
    sending the text to the translation model, guaranteeing zero
    hallucination in the code.

## 57. Summary

The Language Engine is the critical localization perimeter of the OCIF
AI Platform. By seamlessly detecting user language, shielding technical
engineering terminology from destructive translation, and standardizing
all core processing to English, it democratizes access to enterprise
architecture without compromising the precision, grammar, or technical
integrity of the generated outputs.
