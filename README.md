# Agentic AI Literature Review Pipeline

A multi-agent architecture built entirely on n8n for fully autonomous, offline academic literature reviews.

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-critical.svg)
![LangChain](https://img.shields.io/badge/LangChain-Integration-blue.svg)
![Ollama](https://img.shields.io/badge/Ollama-Local_LLM-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

> **Note on Local LLMs:** The repository is configured by default to use **Google Gemini (PaLM) API** (`gemini-3.1-flash-lite-preview`) for out-of-the-box demonstration. To achieve the strict data privacy of Local LLMs (Ollama), simply replace the LangChain Gemini Model nodes with Ollama Chat Model nodes in your n8n workspace.

![Animated Demo](/resources/demo.gif)
*(Above: The Supervisor Agent orchestrating a 50-paper extraction, analysis, and IEEE formatting sequence in real-time.)*

📚 **Tutorials** (No coding required!): [English](TUTORIAL.md) | [Türkçe](TUTORIAL_TR.md)

## 📈 Business Value & ROI

Transforming days of manual research into mere minutes. This multi-agent pipeline is designed to scale, offering immense efficiency for R&D teams and academic researchers:

* **Massive Time Savings**: A literature review of 50 academic papers that typically requires **2-3 weeks** of human effort can be autonomously orchestrated and drafted in approximately **10-15 minutes** (depending on API response times).
* **Cost Efficiency (Benchmark)**: The estimated cloud API cost for processing a standard batch of 50 papers is **under $0.50** when using the default lightweight models (Gemini 1.5 Flash).
* **Fatigue-Free Consistency**: Operates continuously without human fatigue, fully parsing methodologies, extracting cross-cutting findings, and mitigating hallucinations via closed-loop fact-checking.

> *Note on Benchmarks: Time and cost metrics are estimates based on standard 10-15 page academic PDFs and Gemini 1.5 Flash API pricing. Real-world performance may vary depending on document length, active rate limits, and the selected LLM backend.*

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

```mermaid
graph TD
    A[Raw Academic PDFs] -->|Read via n8n| E[02 - Extractor Agent]
    
    subgraph Filesystem as State
        E -->|Writes| F1[(temp_extracted.json)]
        F1 -->|Reads| AN[03 - Analyzer Agent]
        AN -->|Writes| F2[(temp_analysis.json)]
        
        F1 -->|Reads| W[04 - Writer Agent]
        F2 -->|Reads| W
        W -->|Writes & Reads| F3[(temp_draft.txt)]
        
        F1 -->|Reads| C[05 - Fact-Checker Agent]
        F3 -->|Reads| C
    end
    
    C -- "pass: false (Fabricated Claims)" -->|Webhook Trigger| W
    C -- "pass: true" -->|Verified| OUT[Final HTML & PDF]
    
    S((01 - Supervisor)) -.-|Orchestrates| E
    S -.-|Orchestrates| AN
    S -.-|Orchestrates| W
    S -.-|Orchestrates| C
```

**Key Innovations:**
- **Zero-Token Context Shift**: The Supervisor sends isolated "start" text nodes instead of gigabytes of JSON objects, saving millions of tokens.
- **GFM Regex Injection**: A custom Javascript sub-node forces Native HTML compilation of GitHub-Flavored Markdown Tables.
- **Closed-Loop Revision**: The Fact-Checker autonomously forces the Writer to revise hallucinated metrics via a dedicated Webhook circuit.

## ⚠️ Known Limitations
- **Context Window Scaling**: This system operates via full-text Context Injection (sending `temp_extracted.json` directly into the prompt) rather than a Vector Database (RAG). As a result, the maximum volume of papers processed in a single run is limited by the LLM's context window. The system behaves reliably for up to **~50 papers**; beyond this, semantic search (RAG) implementation is required. *(Note: Work in progress 🚧).*

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
