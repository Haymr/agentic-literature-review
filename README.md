# Agentic AI Literature Review Pipeline

A multi-agent architecture built entirely on n8n for fully autonomous, offline academic literature reviews.

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-critical.svg)
![LangChain](https://img.shields.io/badge/LangChain-Integration-blue.svg)
![Ollama](https://img.shields.io/badge/Ollama-Local_LLM-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

📚 **Tutorials** (No coding required!): [English](TUTORIAL.md) | [Türkçe](TUTORIAL_TR.md)

## 📋 Features

- **Multi-Agent Orchestration**: 5 specialized agents mimicking human research workflows.
- **Closed-Loop Self-Correction**: Fact-Checker and Review Writer automatically iterate over factual errors using Webhooks and dynamic context switching.
- **Filesystem as State**: Eliminates `Token Explosion` and `Schema Parsing` errors by reading/writing directly to disk.
- **Offline HTML & PDF Generation**: GFM-compliant HTML tables natively compiled avoiding standard n8n Markdown limitations.
- **API Rate Tracking**: Embedded delay logic to ensure zero `429 Too Many Requests` when using constrained Cloud LLMs (Gemini).

## 🚀 Quick Start

### Installation

```bash
git clone https://github.com/Haymr/agentic-literature-review.git
```
Then, import the 5 JSON files sequentially into your n8n workspace, and activate the workflows.

### Running the Pipeline

Drop your designated PDF literature files into `~/.n8n/local-files/` (or the equivalent mapped Docker volume).

Open the **01-main-supervisor** workflow, set it to Test Mode, and simply type:
```text
Başla (or Start)
```
The Supervisor Agent will autonomously orchestrate the entire review process.

## 📁 Project Structure

```
├── workflows/                    # n8n Agent Deployments
│   ├── 01-main-supervisor.json   # Orchestrator & Chat Memory
│   ├── 02-paper-extractor.json   # Base Reading Agent
│   ├── 03-thematic-analyzer.json # Analysis & Gap Identification
│   ├── 04-review-writer.json     # IEEE Drafting & HTML CSS Render
│   └── 05-fact-checker.json      # Final Validation Protocol
│
├── TUTORIAL.md                   # English Beginner Guide
├── TUTORIAL_TR.md                # Turkish Beginner Guide
├── LICENSE                       # MIT License
└── README.md                     # Project Documentation
```

## 🔬 Pipeline Architecture

```text
Input (Raw Academic PDFs)
    │
    ▼
┌─────────────────────────────────┐
│     Filesystem as State         │
│                                 │
│  [1] Extractor -> temp_ext.json │
│  [2] Analyzer  -> temp_aly.json │
│  [3] Writer   <-> temp_draft.txt│ (Iterative Revision via Webhook)
│  [4] Checker   -> out_check.txt │
└─────────────────────────────────┘
    │
    ▼
Output (literature_review.html)
```

**Key Innovations:**
- **Zero-Token Context Shift**: The Supervisor sends isolated "start" text nodes instead of gigabytes of JSON objects, saving millions of tokens.
- **GFM Regex Injection**: A custom Javascript sub-node forces Native HTML compilation of GitHub-Flavored Markdown Tables.

## 📊 File Artifacts

| Artifact Generated | Read By | Purpose |
|-------------------|----------|---------|
| `temp_extracted.json` | Analyzer, Writer, Checker | Structured PDF raw data |
| `temp_analysis.json` | Writer | Thematic groupings and gaps |
| `temp_draft.txt` | Checker, HTML Compiler| The raw IEEE text payload |
| `out_check.txt` | Supervisor | JSON Verification Audit |

## 🎯 Use Cases

- **Academia**: Automate 80% of rigorous initial literature screening.
- **Enterprise**: In-house R&D (Alternative to NotebookLM) prioritizing strict data privacy via Local LLMs.
- **Data Engineering**: Demonstrating robust multi-agent loops decoupled from monolithic single-prompt inferences.

## 🛠️ Requirements

- n8n instance (Local/Docker Desktop)
- Gemini API Key / Local Ollama Instance
- Node.js environment (for custom CSS JS nodes)

## 📝 License

MIT License - See [LICENSE](LICENSE) for details.
