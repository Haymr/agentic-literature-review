# Agentic AI Literature Review Pipeline

This repository contains a fully automated, multi-agent AI pipeline for conducting comprehensive academic literature reviews. Built entirely on **n8n**, this system serves as a scalable, in-house alternative to tools like NotebookLM, orchestrating 5 specialized AI agents to extract, analyze, draft, and verify academic papers autonomously.

## Architecture: "Filesystem as State"

To avoid LLM token explosions and schema errors common with massive academic JSON payloads, this pipeline delegates state management to the local filesystem. Agents communicate strictly through read/write disk operations, transferring only short strings (e.g., "start") through the LangGraph context window.

The pipeline consists of 5 modular n8n workflows:

1. **01 - Main Supervisor:** The orchestrator agent. It sequentially triggers the specialized sub-agents and monitors their success.
2. **02 - Paper Extractor:** Reads loaded academic PDFs, extracts core text, and dumps structured JSON data to the local disk (`temp_extracted.json`).
3. **03 - Thematic Analyzer:** Processes the extracted JSON to formulate cross-cutting themes, research gaps, and methodologies (`temp_analysis.json`).
4. **04 - Review Writer:** Ingests the themes and structures a comprehensive literature review in IEEE format. Includes a custom Javascript parser to generate a polished GitHub-Flavored Markdown (GFM) table and Academic CSS-styled HTML page (`literature_review.html`).
5. **05 - Fact Checker:** The final verification agent. It audits the generated review against the original PDF contents, identifies hallucinations or fabricated claims, and outputs a strict JSON evaluation report.

## Key Features

- **End-to-End Autonomous Output:** Put PDFs in a folder, click "Start", and retrieve a styled HTTP/PDF literature review.
- **API Rate Limit Protections:** Built-in asynchronous delay nodes (5 seconds) between sub-workflows to respect strict Cloud LLM (Gemini Flash Lite) rate limits.
- **Offline HTML Rendering:** Bypasses n8n's basic Markdown limitations using a custom GFM-to-HTML regex script, outputting pristine tables natively without third-party PDF APIs.
- **Extensible LLM Integrations:** Operates with Langchain base nodes, making it trivial to swap Gemini API models for local models via Ollama.

## Quick Start
Please refer to [TUTORIAL.md](./TUTORIAL.md) for a beginner-friendly installation and execution guide. For Turkish instructions, see [TUTORIAL_TR.md](./TUTORIAL_TR.md).
