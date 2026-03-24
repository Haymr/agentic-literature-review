# Changelog

All notable changes to this project will be documented in this file.

## [v2.0.0] - Dynamic Revision Engine (feature/dynamic-revision-engine)

This release introduces a paradigm shift in the pipeline architecture, moving from a single-shot deterministic workflow to a **Closed-Loop Self-Correcting System**.

### 🌟 Major Architectural Changes

*   **Dinamik Revizyon Motoru (Cognitive Context Switch) - `04-review-writer.json`**:
    *   **Webhook Integration**: The Writer Agent now listens on `POST /review-writer-revision`. It is no longer just triggered by the Supervisor.
    *   **Context Switcher**: A new router node detects if the trigger is `INITIAL` (from Supervisor) or `REVISION` (from Fact Checker).
    *   **Dynamic Prompting**: Instead of hardcoded prompts, the LLM agent compiles its System and User Prompts on the fly. If revising, it reads `temp_draft.txt` and only fixes the specific fabricated claims detected by the Fact Checker without rewriting the entire paper from scratch.

*   **Agent to Basic LLM Chain Conversion - `05-fact-checker.json`**:
    *   Migrated the LangChain Agent node to a **Basic LLM Chain** node.
    *   *Why?* LangChain Agents in n8n crash (Configuration Error) when no executable Tools are provided. Since we inject RAG/document knowledge directly into the context payload, tools were unnecessary. The Basic Chain fulfills the exact same role without crashing.

### 🐛 Critical Bug Fixes

*   **Deep Freeze Constraint Bypass (Read-Only Property Error)**: 
    *   *Issue:* n8n v1+ locks input objects (`$input.all()`) as read-only proxies. Attempting to modify or merge these caused pipeline crashes (`Cannot assign to read-only property`).
    *   *Fix:* Implemented deep cloning (`JSON.parse(JSON.stringify(item.json))`) at the start of complex JavaScript code blocks, allowing isolated data mutation.
*   **"Double-Stringified JSON" Data Loss & LLM Hallucinations**:
    *   *Issue:* LLMs occasionally return raw JSON strings instead of parsable objects. When n8n saved this to `temp_analysis.json` and read it back, properties like `.themes` resolved to `undefined`, causing the Writer to hallucinate papers without actual data.
    *   *Fix:* Embedded **Defensive Parsing** mechanisms in both `03-thematic-analyzer.json` (before writing) and `04-review-writer.json` (during reading). If the payload is detected as a string, n8n automatically runs `JSON.parse` to guarantee an Object is passed to the LLM.

## [v1.0.0] - Initial Release (main)
*   Initial 5-agent linear pipeline (Supervisor → Extractor → Analyzer → Writer → Checker).
*   Complete offline HTML and PDF generation.
*   Filesystem-as-State pattern implemented to avoid Token explosion.
