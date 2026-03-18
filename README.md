<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&amp;color=6366f1&amp;height=200&amp;section=header&amp;text=AI%20Research%20Crew&amp;fontSize=52&amp;fontColor=ffffff&amp;fontAlignY=38&amp;desc=Autonomous%20multi-agent%20literature%20discovery%20%26%20research%20gap%20synthesis&amp;descAlignY=58&amp;descColor=c7d2fe" width="100%"/>

<br/>

[![CrewAI](https://img.shields.io/badge/Built%20With-CrewAI-6366f1?style=for-the-badge)](https://github.com/crewAIInc/crewAI)
[![OpenAI](https://img.shields.io/badge/LLM-GPT--4o--mini-10a37f?style=for-the-badge&amp;logo=openai&amp;logoColor=white)](https://platform.openai.com)
[![Gradio](https://img.shields.io/badge/UI-Gradio-ff7c00?style=for-the-badge)](https://gradio.app)
[![ArXiv](https://img.shields.io/badge/Source-ArXiv-b31b1b?style=for-the-badge)](https://arxiv.org)
[![Colab](https://img.shields.io/badge/Platform-Google%20Colab-f9ab00?style=for-the-badge&amp;logo=googlecolab&amp;logoColor=white)](https://colab.research.google.com)
[![License](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)](LICENSE)

<br/>

> **Type a topic. Let AI agents do the rest.**
> *Search ArXiv → Download PDFs → Extract insights → Synthesize research gaps.*

<br/>

[🚀 Quick Start](#-quick-start) · [🏗️ Architecture](#️-architecture) · [🧪 Example Output](#-example-output) · [🐛 Known Issues](#-known-issues--fixes) · [🗺️ Roadmap](#️-roadmap)

</div>

---

## 🌟 What Is This?

**AI Research Crew** is a fully autonomous, multi-agent research assistant that takes a single topic as input and returns a complete literature review with synthesized research gaps — without you touching a single paper.

It orchestrates two specialist AI agents using **CrewAI** to automate the most tedious part of academic work:

```
 You type: "Fake News Detection using LLMs"
                          │
          ┌───────────────▼────────────────┐
          │      🔎 Research Librarian      │
          │                                │
          │  · Searches ArXiv for papers   │
          │  · Downloads 3 PDFs locally    │
          │  · Returns exact file paths    │
          └───────────────┬────────────────┘
                          │  passes context
          ┌───────────────▼────────────────┐
          │      🧪 Scientific Analyst      │
          │                                │
          │  · RAG-searches each PDF       │
          │  · Extracts methodology        │
          │  · Identifies limitations      │
          │  · Synthesizes research gap    │
          └───────────────┬────────────────┘
                          │
          ┌───────────────▼────────────────┐
          │         📄 Final Output         │
          │                                │
          │  Structured literature review  │
          │  + Clearly defined gap 🎯      │
          └────────────────────────────────┘
```

---

## ✨ Features

- 🔍 **Autonomous ArXiv Search** — fetches and downloads real PDFs, no manual browsing
- 🧠 **RAG-based PDF Analysis** — reads and reasons over actual document content
- 🎯 **Research Gap Synthesis** — compares papers and identifies what's missing in the field
- 🖥️ **Clean Gradio UI** — sidebar config, tabbed results, real-time status
- ⚡ **Two Process Modes** — Sequential (stable) and Hierarchical (experimental) both implemented
- 🔁 **Retry-resilient** — `max_iter` guards and sequential flow prevent runaway loops

---

## 🏗️ Architecture

### Project Structure

```
ai-research-crew/
│
├── app_sequential.py            # ✅ Stable: sequential agent pipeline
├── app_hierarchical.py          # 🧪 Experimental: manager-led orchestration
│
├── arxiv_pdfs/                  # Auto-created: downloaded papers land here
└── Literature-review-&-Research-Gap/   # Auto-created: output directory
```

### Agent Roster

| Agent | Role | Tools | Process |
|-------|------|-------|---------|
| 🔎 **Research Librarian** | Searches ArXiv, downloads PDFs, returns local paths | `ArxivPaperTool` | Runs first |
| 🧪 **Scientific Analyst** | RAG-searches PDFs, extracts insights, finds gaps | `PDFSearchTool` | Runs second, receives context |
| 🧑‍💼 **Research Manager** *(hierarchical only)* | Delegates tasks between agents, coordinates flow | — | Oversees both |

### Process Modes Explained

This project implements and compares **both** CrewAI process modes:

<table>
<tr>
<th width="50%">✅ Sequential (Recommended)</th>
<th width="50%">🧪 Hierarchical (Experimental)</th>
</tr>
<tr>
<td>

```python
Crew(
  agents=[researcher, analyst],
  tasks=[research_task, analysis_task],
  process=Process.sequential,
)
```

</td>
<td>

```python
Crew(
  agents=[researcher, analyst],
  tasks=[research_task, analysis_task],
  process=Process.hierarchical,
  manager_agent=manager,
)
```

</td>
</tr>
<tr>
<td>
✔ Reliable, no OpenAI 400 errors<br/>
✔ Direct task → task context passing<br/>
✔ No manager LLM cost<br/>
✔ Production-ready
</td>
<td>
⚠ Occasional tool_call_id mismatch errors<br/>
⚠ Manager adds delegation overhead<br/>
✔ Self-recovers via CrewAI retry logic<br/>
✔ Useful for studying agent coordination
</td>
</tr>
</table>

---

## 🚀 Quick Start

### Prerequisites

- Python 3.9+
- An OpenAI API key ([get one here](https://platform.openai.com/api-keys))

### 1. Clone & Install

```bash
git clone https://github.com/YOUR_USERNAME/ai-research-crew.git
cd ai-research-crew

pip install crewai crewai-tools gradio openai
```

### 2. Set Your API Key

Open `app_sequential.py` and replace the placeholder:

```python
os.environ['OPENAI_API_KEY'] = "your-openai-api-key-here"
```

### 3. Launch

```bash
# Recommended: Sequential mode (stable)
python app_sequential.py

# Experimental: Hierarchical mode
python app_hierarchical.py
```

A Gradio UI will open with a public share link automatically.

---

### 🧪 Running on Google Colab

```python
# Step 1: Install
!pip install crewai crewai-tools gradio openai

# Step 2: Set key
import os
os.environ['OPENAI_API_KEY'] = "your-key-here"

# Step 3: Run
!python app_sequential.py
```

> **Colab note:** PDFs save to `/content/arxiv_pdfs/`. ArXiv IDs use underscores in filenames — `2003.05096v1` becomes `2003_05096v1.pdf`.

---

## ⚙️ Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `model` | `gpt-4o-mini` | OpenAI model for all agents |
| `temperature` | `0.2` | Low = consistent, factual output |
| `max_iter` | `5` | Max reasoning steps per agent before stopping |
| `timeout` | `120s` | LLM call timeout to prevent hanging |
| `save_dir` | `./arxiv_pdfs` | Where downloaded PDFs are stored |
| Papers fetched | `3` | Number of ArXiv papers per run |

---

## 🧪 Example Output

**Input topic:** `"Temporal Graph RAG in Medicine"`

```markdown
## 📚 Literature Review

### Paper 1 — RAG-Gym: Systematic Optimization of Language Agents
- **Methodology:** Multi-round agentic RAG with RL-based reward optimization
- **Key Findings:** Significant improvement over naive RAG on QA benchmarks
- **Limitations:** Does not address temporal knowledge drift; tested only on static corpora

---

### Paper 2 — MultiHop-RAG: Benchmarking Multi-Hop Queries
- **Methodology:** Chain-of-retrieval over multi-hop QA datasets
- **Key Findings:** Standard RAG pipelines fail on questions requiring 3+ hops
- **Limitations:** Benchmark limited to English; ignores graph structure entirely

---

### Paper 3 — RAG-Star: Deliberative Reasoning with Retrieval Verification
- **Methodology:** Tree-search combined with external retrieval for step verification
- **Key Findings:** Outperforms chain-of-thought on complex multi-step reasoning
- **Limitations:** High latency; no domain adaptation for medical ontologies

---

## 🎯 Research Gap

No existing framework combines **temporal graph-aware retrieval** with **medical
ontology grounding** for RAG systems. Current approaches treat knowledge bases as
static snapshots, missing time-sensitive clinical relationships critical in medicine.
A system that dynamically updates graph embeddings from clinical event streams
while grounding retrieval in ontologies like SNOMED-CT remains an open problem.
```

---

## 🐛 Known Issues & Fixes

### ❌ OpenAI Error 400 — Unresolved `tool_call_id`

**What it looks like:**
```
Error code: 400 - An assistant message with 'tool_calls' must be followed 
by tool messages responding to each 'tool_call_id'.
```

**Root cause:** When using `Process.hierarchical`, the manager delegates a task to a sub-agent. If that sub-agent's tool call doesn't resolve cleanly, the OpenAI message thread has an orphaned `tool_call_id` — which the API rejects.

**Fix — use sequential mode:**
```python
# ❌ Causes the error
Crew(..., process=Process.hierarchical, manager_agent=manager)

# ✅ Clean and reliable
Crew(..., process=Process.sequential)
```

**Why the app still works despite the error:** CrewAI has built-in retry logic. It retries the failed delegation until the tool call succeeds — you'll see the same agent task appear multiple times in logs.

---

### ❌ PDF File Not Found

**Root cause:** ArXiv paper IDs use dots (`2003.05096v1`) but saved filenames use underscores (`2003_05096v1.pdf`). Agents sometimes hallucinate the wrong path.

**Fix — normalize paths:**
```python
def normalize_arxiv_path(path: str) -> str:
    filename = os.path.basename(path)
    name, ext = os.path.splitext(filename)
    return f"./arxiv_pdfs/{name.replace('.', '_')}{ext}"
```

---

### ❌ Infinite Tool Loop

**Root cause:** Without an iteration cap, an agent can keep re-calling `arxiv_paper_fetcher` with a cached generic query and never terminate.

**Fix:**
```python
researcher = Agent(
    ...
    max_iter=5,          # Hard cap on reasoning steps
    max_retry_limit=3    # Cap on task-level retries
)
```

---

## 🛠️ Tech Stack

| Technology | Purpose |
|------------|---------|
| [CrewAI](https://github.com/crewAIInc/crewAI) | Multi-agent orchestration |
| [GPT-4o-mini](https://platform.openai.com) | LLM backbone for all agents |
| [ArxivPaperTool](https://docs.crewai.com/tools/arxivtool) | ArXiv search & PDF download |
| [PDFSearchTool](https://docs.crewai.com/tools/pdfsearchtool) | RAG over local PDFs |
| [Gradio](https://gradio.app) | Web UI with sidebar + tabs |

---

## 🗺️ Roadmap

- [ ] 🔄 Support Gemini / Claude as alternative LLM backends
- [ ] 🌐 Add `SerperDevTool` agent for web-supplemented research
- [ ] 📄 Export literature review as `.docx` / `.pdf`
- [ ] 📚 Citation formatting — APA, MLA, BibTeX
- [ ] 💾 Persistent session history across Gradio runs
- [ ] 🐳 Docker container for one-command deployment
- [ ] 📊 Side-by-side comparison: Sequential vs Hierarchical outputs

---

## 👤 Author

<div align="center">

**Junaid Solangi**

*Built to automate the grunt work of academic research — so researchers can focus on ideas, not PDFs.*

[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&amp;logo=github)](https://github.com/YOUR_USERNAME)

</div>

---

## 📄 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&amp;color=6366f1&amp;height=100&amp;section=footer" width="100%"/>

*If this saved you hours of literature hunting, consider giving it a ⭐*

</div>
