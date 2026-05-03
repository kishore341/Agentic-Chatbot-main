# LangGraph Agentic AI Platform — Stateful Multi-Agent Chatbot

> A modular, production-ready agentic AI framework built with LangGraph supporting stateful graph-based conversation flows, web-augmented search, and automated AI news aggregation.

---

## Overview

This project demonstrates a real-world implementation of **agentic AI** using LangGraph's StateGraph architecture. Instead of a simple linear chatbot, it uses a graph of nodes and conditional edges — allowing the AI to reason, choose tools, and maintain state across multiple conversation turns.

Three fully working use cases are supported from a single unified graph builder:

| Use Case | Description |
|---|---|
| 🤖 Basic Chatbot | Stateful LLM conversation with message history |
| 🌐 Web Search Agent | Chatbot augmented with Tavily web search tool |
| 📰 AI News Aggregator | Fetches, summarizes & saves AI news (daily/weekly/monthly) |

---

## Architecture

```
User Input
    │
    ▼
┌─────────────────────────────────┐
│         Streamlit UI            │
│  (LLM selector, use case, keys) │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│         Graph Builder           │
│   Selects graph based on        │
│   chosen use case               │
└──────┬────────────┬─────────────┘
       │            │            │
       ▼            ▼            ▼
┌──────────┐ ┌──────────┐ ┌──────────────────┐
│  Basic   │ │ Web Tool │ │   AI News Graph  │
│ Chatbot  │ │ Chatbot  │ │                  │
│  Graph   │ │  Graph   │ │ fetch_news node   │
│          │ │          │ │      ↓           │
│ START    │ │ START    │ │ summarize_news   │
│   ↓      │ │   ↓      │ │      ↓           │
│ chatbot  │ │ chatbot  │ │  save_result     │
│   ↓      │ │   ↓      │ │      ↓           │
│  END     │ │ tools?   │ │     END          │
│          │ │   ↓      │ └──────────────────┘
│          │ │ tool_node│
│          │ │   ↓      │
│          │ │ chatbot  │
│          │ │   ↓      │
│          │ │  END     │
└──────────┘ └──────────┘
               │
               ▼
     LangGraph State (add_messages)
     Stateful memory across turns
```

---

## Tech Stack

| Component | Technology |
|---|---|
| Agentic Framework | LangGraph, LangChain |
| LLM Provider | Groq API |
| LLM Models | Qwen3-32B (`qwen/qwen3-32b`), LLaMA 3 70B (`llama3-70b-8192`, `llama-3.3-70b-versatile`), Gemma2-9B-IT (`gemma2-9b-it`) |
| Web Search | Tavily API (TavilySearchResults) |
| UI | Streamlit |
| State Management | LangGraph `add_messages` annotation |
| Config | Python ConfigParser (`.ini` file) |

---

## Features

### 1. Stateful Graph Architecture
- Uses `TypedDict` State with `Annotated[List, add_messages]` for automatic message accumulation
- Conditional edges route between chatbot and tool nodes based on LLM tool-call decisions
- `START` and `END` nodes define clean entry/exit points per use case

### 2. Three Autonomous Use Cases

**Basic Chatbot**
```
START → chatbot_node → END
```
- Direct LLM conversation using Groq models
- Full message history maintained in graph state

**Chatbot With Web Search**
```
START → chatbot_node → [tools_condition] → tool_node → chatbot_node → END
```
- LLM decides when to call `TavilySearchResults` (max 2 results)
- Tool results fed back into chatbot for synthesis
- Loops until LLM produces a final answer without tool calls

**AI News Aggregator**
```
START → fetch_news → summarize_news → save_result → END
```
- Fetches top AI news (India + global) via Tavily API
- Supports `daily`, `weekly`, and `monthly` time frames
- LLM summarizes into structured markdown format (sorted by IST date, latest first)
- Saves to `./AINews/{frequency}_summary.md`

### 3. Dynamic Model Selection
- All Groq models selectable from Streamlit sidebar
- Config-driven via `uiconfigfile.ini` — no code changes needed to add new models

### 4. News Summary Format
Each news summary is saved as:
```markdown
### [2025-10-27]
- [Article headline and summary](source_url)

### [2025-10-24]
- [Article headline and summary](source_url)
```

---

## Project Structure

```
langgraph-agentic-ai/
├── app.py                          # Entry point
├── src/langgraphagenticai/
│   ├── main.py                     # App orchestrator
│   ├── graph/
│   │   └── graph_builder.py        # All graph definitions
│   ├── nodes/
│   │   ├── basic_chatbot_node.py   # Basic LLM node
│   │   ├── chatbot_with_Tool_node.py # Tool-augmented chatbot
│   │   └── ai_news_node.py         # News fetch/summarize/save
│   ├── state/
│   │   └── state.py                # Shared graph state
│   ├── tools/
│   │   └── search_tool.py          # Tavily tool definition
│   ├── LLMS/
│   │   └── groqllm.py              # Groq LLM initializer
│   └── ui/
│       ├── uiconfigfile.ini        # Config (models, usecases)
│       ├── uiconfigfile.py         # Config reader
│       └── streamlitui/
│           ├── loadui.py           # Sidebar + controls
│           └── display_result.py   # Result renderer
├── AINews/                         # Auto-generated news summaries
│   ├── daily_summary.md
│   ├── weekly_summary.md
│   └── monthly_summary.md
└── requirements.txt
```

---

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/kishore341/langgraph-agentic-ai
cd langgraph-agentic-ai

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the app
streamlit run app.py
```

---

## Configuration

Edit `src/langgraphagenticai/ui/uiconfigfile.ini` to add models or use cases:

```ini
[DEFAULT]
PAGE_TITLE = LangGraph: Build Stateful Agentic AI graph
LLM_OPTIONS = Groq
USECASE_OPTIONS = Basic Chatbot, Chatbot With Web, AI News
GROQ_MODEL_OPTIONS = qwen/qwen3-32b, llama3-70b-8192, gemma2-9b-it, llama-3.3-70b-versatile
```

---

## API Keys Required

| Key | Where to get |
|---|---|
| `GROQ_API_KEY` | https://console.groq.com/keys |
| `TAVILY_API_KEY` | https://app.tavily.com/home |

Enter both in the Streamlit sidebar before running.

---

## Requirements

```
langchain
langgraph
langchain_community
langchain_core
langchain_groq
langchain_openai
faiss-cpu
streamlit
tavily-python
```

---

## Author

**Kishore Kumar Kunuku** — AI Engineer  
Personal Project  
[LinkedIn](https://linkedin.com/in/kishore-kumar-kunuku-9bb91830b) | [GitHub](https://github.com/kishore341)
