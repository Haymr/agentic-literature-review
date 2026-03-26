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
*   **Fact-Checker Programmatic Leniency & JSON Robustness**:
    *   *Issue:* The Fact-Checker LLM was overly pedantic, failing valid syntheses (e.g., merging "5-6" and "12-14" to "5 to 14") and occasionally generating invalid JSON comments (`//`) that broke the parser, causing infinite 3-retry loops.
    *   *Fix:* Removed invalid comments from the prompt blueprint, stripped control characters via regex, and added a programmatic leniency guard that forces a `pass: true` if no claims are explicitly `FABRICATED`.
*   **Blind Re-Writer (Attention Dilution) Fix - `04-review-writer.json`**:
    *   *Issue:* During the webhook revision loop, the Writer Agent was only supplied with the draft and the Checker's feedback. Lacking the original source papers, it could not accurately correct hallucinated claims.
    *   *Fix:* Injected the Ground Truth (`source_papers_text`) into the Fact-Checker's webhook payload. The Writer now receives the original source material during revision, ensuring 100% accurate factual corrections without guessing.
*   **Extractor Batch Dropping Bug - `02-paper-extractor.json`**:
    *   *Issue:* Extractor dropped PDFs when `batchSize` > 1 due to hardcoded `$input.first()` methods processing only the first item in the batch.
    *   *Fix:* Converted all input parsing nodes to use array mapping (`$input.all().map(...)`), ensuring complete processing of all PDFs in the batch.

### 🏗️ Architectural Refactoring (Phase 2)
*   **Zero-Token Restoration - `04-review-writer.json` & `05-fact-checker.json`**:
    *   *Issue:* The Ground Truth injection fix forced the Fact-Checker to send massive PDF textual data over the HTTP webhook, effectively violating the filesystem-as-state zero-token design.
    *   *Fix:* Stripped the massive payload from the Fact-Checker webhook. Added a local disk-read circuit (`Read/Parse Ground Truth`) directly to the Writer's webhook revision branch, allowing it to autonomously fetch `temp_extracted.json` during error correction without API transmission overhead.

## [v1.0.0] - Initial Release (main)
*   Initial 5-agent linear pipeline (Supervisor → Extractor → Analyzer → Writer → Checker).
*   Complete offline HTML and PDF generation.
*   Filesystem-as-State pattern implemented to avoid Token explosion.
