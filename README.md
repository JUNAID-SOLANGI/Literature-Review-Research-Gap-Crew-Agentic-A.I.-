# 🤖 AI Research Crew
### *Autonomous multi-agent literature discovery, PDF analysis, and research gap synthesis*

<p align="center">
  <img src="https://img.shields.io/badge/Built%20With-CrewAI-6366f1?style=for-the-badge" />
  <img src="https://img.shields.io/badge/LLM-GPT--4o--mini-10a37f?style=for-the-badge&logo=openai" />
  <img src="https://img.shields.io/badge/UI-Gradio-ff7c00?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Source-ArXiv-b31b1b?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Google%20Colab-f9ab00?style=for-the-badge&logo=googlecolab" />
</p>

<p align="center">
  <b>Enter a research topic. Let AI agents do the rest.</b><br/>
  Searches ArXiv → Downloads PDFs → Reads them → Finds the research gap.
</p>

---

## ✨ What It Does

**AI Research Crew** orchestrates a team of specialized AI agents that automate the most tedious parts of academic research. You type a topic — the agents handle everything else.

```
User Input: "Fake News Detection using LLMs"
     │
     ▼
┌─────────────────────┐     ┌──────────────────────┐     ┌────────────────────────┐
│   Research          │────▶│   Scientific          │────▶│   Literature Review    │
│   Librarian 📚      │     │   Analyst 🔬          │     │   + Research Gap 🎯    │
│                     │     │                       │     │                        │
│  Searches ArXiv     │     │  RAG-searches PDFs    │     │  Methodology summary   │
│  Downloads 3 PDFs   │     │  Extracts insights    │     │  Limitations analysis  │
│  Returns file paths │     │  Identifies gaps      │     │  Novel gap synthesis   │
└─────────────────────┘     └──────────────────────┘     └────────────────────────┘
```

---

## 🖥️ UI Preview

The Gradio interface features a **sidebar** for configuration and a **tabbed main panel** for results and logs.

| Panel | Description |
|-------|-------------|
| ⚙️ **Sidebar** | Enter your topic, set max papers, choose LLM model |
| 📖 **Literature Review Tab** | Full structured review rendered as markdown |
| 📋 **Process Logs Tab** | Real-time agent status tracker |

---

## 🏗️ Architecture

```
ai-research-crew/
│
├── app.py                  # Main application + Gradio UI
├── arxiv_pdfs/             # Auto-created: downloaded PDFs stored here
└── Literature-review-&-Research-Gap/   # Auto-created: output directory
```

### Agents

| Agent | Role | Tools |
|-------|------|-------|
| 🔎 **Research Librarian** | Searches ArXiv, downloads PDFs, returns local file paths | `ArxivPaperTool` |
| 🧪 **Scientific Analyst** | RAG-searches downloaded PDFs, extracts methodology & limitations | `PDFSearchTool` |

### Process Flow
- **`Process.sequential`** — Researcher runs first, passes context to Analyst
- No hierarchical manager (avoids OpenAI tool-call ID mismatch errors)
- `max_iter=5` per agent prevents runaway loops
- `memory=False` prevents stale context accumulation across runs

---

## 🚀 Quick Start

### 1. Clone & Install

```bash
git clone https://github.com/YOUR_USERNAME/ai-research-crew.git
cd ai-research-crew
pip install crewai crewai-tools gradio openai
```

### 2. Set Your API Key

```python
# In app.py, replace:
os.environ['OPENAI_API_KEY'] = "your-openai-api-key-here"
```

### 3. Run

```bash
python app.py
```

The Gradio UI will launch with a public share link automatically.

---

### 🧪 Running on Google Colab

```python
# Install dependencies
!pip install crewai crewai-tools gradio openai

# Set your key
import os
os.environ['OPENAI_API_KEY'] = "your-key-here"

# Run
!python app.py
```

> **Note for Colab users:** PDFs are saved to `/content/arxiv_pdfs/`. ArXiv filenames use **underscores**, not dots — `2003.05096v1` becomes `2003_05096v1.pdf`.

---

## ⚙️ Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `model` | `gpt-4o-mini` | OpenAI model for all agents |
| `temperature` | `0.2` | Low temp for consistent, factual output |
| `max_results` | `3` | Number of ArXiv papers to fetch |
| `max_iter` | `5` | Max reasoning iterations per agent |
| `save_dir` | `./arxiv_pdfs` | Local directory for downloaded PDFs |

---

## 🧠 Example Output

**Topic:** `"Temporal Graph RAG in Medicine"`

```markdown
## Literature Review

### Paper 1: RAG-Gym — Systematic Optimization of Language Agents
- **Methodology:** Multi-round agentic RAG with RL-based optimization
- **Limitations:** Does not address temporal knowledge drift; tested only
  on static corpora

### Paper 2: MultiHop-RAG — Benchmarking for Multi-Hop Queries  
- **Methodology:** Chain-of-retrieval over multi-hop QA datasets
- **Limitations:** Benchmark limited to English; ignores graph structure

### Paper 3: RAG-Star — Deliberative Reasoning with Verification
- **Methodology:** Tree-search + external retrieval for reasoning steps
- **Limitations:** High latency; no domain adaptation for medical ontologies

---

## 🎯 Research Gap

There is no existing framework that combines **temporal graph-aware retrieval**
with **medical ontology grounding** for RAG systems. Current approaches treat
knowledge as static, missing critical time-sensitive clinical relationships.
```

---

## 🐛 Known Issues & Fixes

### ❌ Error 400 — Unresolved `tool_call_id`
**Cause:** Using `Process.hierarchical` with a manager agent causes OpenAI to reject malformed message histories when a sub-agent tool call fails mid-execution.

**Fix:** Use `Process.sequential` — no manager LLM, no delegation chain, no broken tool IDs.

```python
# ❌ Don't do this
research_crew = Crew(..., process=Process.hierarchical, manager_agent=manager)

# ✅ Do this
research_crew = Crew(..., process=Process.sequential)
```

---

### ❌ PDF File Not Found Error
**Cause:** ArXiv IDs use dots (`2003.05096v1`) but saved filenames use underscores (`2003_05096v1.pdf`). The agent hallucinates the wrong path.

**Fix:** Normalize paths before passing to `PDFSearchTool`:

```python
def normalize_arxiv_path(path: str) -> str:
    filename = os.path.basename(path)
    name, ext = os.path.splitext(filename)
    return f"/content/arxiv_pdfs/{name.replace('.', '_')}{ext}"
```

---

### ❌ Infinite Tool Loop (70+ cached calls)
**Cause:** Agent kept re-calling `arxiv_paper_fetcher_and_downloader` with the generic query `'RAG'` from cache without terminating.

**Fix:** Set `max_iter=5` on each agent and enforce a specific search query in the task description.

---

## 🛠️ Tech Stack

| Technology | Purpose |
|------------|---------|
| [CrewAI](https://github.com/crewAIInc/crewAI) | Multi-agent orchestration framework |
| [OpenAI GPT-4o-mini](https://platform.openai.com) | LLM backbone for all agents |
| [ArxivPaperTool](https://docs.crewai.com/tools/arxivtool) | Searches & downloads ArXiv papers |
| [PDFSearchTool](https://docs.crewai.com/tools/pdfsearchtool) | RAG over local PDF files |
| [Gradio](https://gradio.app) | Web UI with sidebar + tabs |

---

## 🗺️ Roadmap

- [ ] Support Gemini / Claude as alternative LLM backends
- [ ] Add a `SerperDevTool` agent for web-based supplementary research
- [ ] Export literature review as `.docx` / `.pdf`
- [ ] Add citation formatting (APA / MLA / BibTeX)
- [ ] Persistent history across Gradio sessions
- [ ] Docker container for one-command deployment

---

## 👤 Author

**Junaid Solangi**

> Built with CrewAI to automate the grunt work of academic research — so researchers can focus on ideas, not PDFs.

---

## 📄 License

This project is licensed under the MIT License.

---

<p align="center">
  <i>If this saved you hours of literature hunting, give it a ⭐</i>
</p>
