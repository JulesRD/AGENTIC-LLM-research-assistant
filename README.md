# AI Research Assistant

A multi-agent AI system for scientific literature research, analysis, and synthesis. The application fetches academic articles from Semantic Scholar, analyzes them using LLMs, and generates comprehensive, cited responses to user queries.

> **TL;DR** — Ask a research question, get a cited answer built from real papers. An LLM orchestrator with a reflection loop decides when to search Semantic Scholar, when to retrieve from the RAG store and when to synthesize.
>
> - **Reasoning separated from execution**: the orchestrator only outputs strict JSON instructions, which the server interprets deterministically to call search, RAG and synthesis
> - **Full traceability**: every claim in the answer is linked to the article it comes from
> - **Observable**: the chat UI streams each pipeline step, tokens per agent, cost and latency
> - **Evaluated** on 7 scenarios (human-judged quality, tokens, latency), with an analysis of its limits: synthesis cost, sequential latency, small-model context
>
> *EPITA academic team project (3 people) · Python, FastAPI, LangChain, FAISS, Ollama (mxbai-embed-large), Mistral API*

## 🌟 Features

- **Intelligent Research**: Automatically searches Semantic Scholar for relevant academic articles
- **Multi-Agent Architecture**: Specialized agents for analysis, research, synthesis, and formatting
- **RAG-Enhanced**: FAISS vector store for efficient context retrieval
- **Real-time Streaming**: Server-Sent Events (SSE) for live response updates
- **Citation Tracking**: Automatic source attribution and bibliography generation
- **Cost Monitoring**: Built-in token usage and cost tracking
- **Flexible LLM Support**: Compatible with Ollama (local) and Mistral API

## 🏗️ Architecture

```
┌─────────────────┐
│  User Query     │
└────────┬────────┘
         │
    ┌────▼─────────┐
    │ Analysis     │ ◄── Orchestrator
    │ Agent        │     Decides: Research or Synthesize?
    └────┬─────────┘
         │
    ┌────┴─────────────────────┐
    │                          │
┌───▼──────────┐      ┌────────▼─────────┐
│ Research     │      │ Synthesis        │
│ Agent        │──────► Agent            │
│ (Semantic    │      │ (Summarize       │
│  Scholar)    │      │  Articles)       │
└──────────────┘      └──────┬───────────┘
                             │
                      ┌──────▼───────────┐
                      │ Formatting       │
                      │ Agent            │
                      │ (Citations)      │
                      └──────────────────┘
```

### Agent Descriptions

- **AnalysisAgent**: Orchestrates the workflow, uses RAG to check existing context, decides whether to trigger research or synthesis
- **ResearchAgent**: Generates search queries and fetches articles from Semantic Scholar API
- **SynthesisAgent**: Summarizes individual articles and synthesizes them into a coherent response
- **FormattingAgent**: Formats output as Markdown and extracts used sources for citation

## 📋 Prerequisites

- Python 3.8+
- [Ollama](https://ollama.ai/) installed (for local LLM) OR Mistral API key
- Embedding model: `mxbai-embed-large` (install via Ollama)

## 🚀 Installation

1. **Clone the repository**
```bash
git clone https://github.com/JulesRD/AGENTIC-LLM-research-assistant.git
cd AGENTIC-LLM-research-assistant
```

2. **Create a virtual environment**
```bash
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

3. **Install dependencies**
```bash
pip install -r requirements.txt
```

4. **Set up Ollama (if using local LLM)**
```bash
# Pull the embedding model
ollama pull mxbai-embed-large

# Pull a chat model (example)
ollama pull mistral
```

5. **Configure environment variables**

Create a `.env` file in the project root:

```env
# LLM Configuration
MODEL=mistral  # or "mistral-small-latest" for Mistral API
OLLAMA_HOST=http://localhost:11434  # Optional: custom Ollama host

# Mistral API (if using Mistral instead of Ollama)
MISTRAL_API_KEY=your_api_key_here

# Cost Tracking (optional)
INPUT_PRICE_PER_MILLION=0.0  # Price per 1M input tokens
OUTPUT_PRICE_PER_MILLION=0.0  # Price per 1M output tokens
```

## 🎯 Usage

### Start the Server

```bash
# Using the launch script
./launch_server.sh

# Or directly
uvicorn src.server:app --reload --host 0.0.0.0 --port 8000
```

The web interface will be available at: **http://localhost:8000**

### API Endpoints

#### Chat with the Assistant
```bash
POST /chat
Content-Type: application/json

{
  "message": "What are the latest advances in AI for healthcare?",
  "max_iterations": 15,
  "max_articles": 5
}
```

Returns: Server-Sent Events (SSE) stream with real-time updates

#### Get Chat History
```bash
GET /history
```

#### Get Cost Statistics
```bash
GET /costs
```

#### Clear Chat History
```bash
DELETE /history
```

#### Stop Generation
```bash
POST /stop
Content-Type: application/json

{
  "session_id": "optional-session-id"
}
```

## 🔧 Configuration

### Model Selection

The system supports two LLM backends:

**Ollama (Local)**:
- Set `MODEL=mistral` (or any Ollama model name)
- Ensure Ollama is running on `http://localhost:11434`

**Mistral API (Cloud)**:
- Set `MODEL=mistral-small-latest` (or any Mistral model)
- Provide `MISTRAL_API_KEY` in `.env`

### Adjusting Search Parameters

Modify in chat request:
- `max_iterations`: Maximum agent iterations (default: 15)
- `max_articles`: Maximum articles per query (default: 5)

### Customizing Prompts

Agent prompts are located in `src/prompt/`:
- `system/analyse.md`: Analysis agent system prompt
- `system/synthese.md`: Synthesis agent system prompt  
- `system/formatting.md`: Formatting agent system prompt
- `analyse.md`, `synthese.md`, `formatting.md`: User prompts

## 📊 Cost Tracking

All LLM calls are automatically logged to `costs.csv` with:
- Timestamp
- Session ID
- Model name
- Token counts (input/output)
- Latency
- Estimated costs

View statistics via the `/costs` endpoint or in the web UI sidebar.
