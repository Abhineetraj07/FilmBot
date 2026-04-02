<h1 align="center">🎬 FilmBot V2 — Multi-Modal AI Movie Agent</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/LangGraph-1.0-2CA5E0?style=for-the-badge" />
  <img src="https://img.shields.io/badge/FastAPI-0.135-009688?style=for-the-badge&logo=fastapi&logoColor=white" />
  <img src="https://img.shields.io/badge/Neo4j-5.28-008CC1?style=for-the-badge&logo=neo4j&logoColor=white" />
  <img src="https://img.shields.io/badge/ChromaDB-1.1-FF6F00?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Streamlit-UI-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white" />
</p>

<p align="center">
  An intelligent movie chatbot that dynamically routes questions across <b>SQL, Vector Search, and Knowledge Graph</b> to deliver accurate, grounded answers over an IMDB dataset of 1,000 movies — with production-grade guardrails.
</p>

---

## ✨ Key Features

- 🤖 **Agentic Routing** — LangGraph agent auto-selects the best retrieval tool per query
- 🧠 **3 Retrieval Modes** — SQL (rankings), Vector (semantic), Knowledge Graph (relationships)
- 🛡️ **7-Category Guardrails** — Input validation, injection protection, role-based access
- ⚡ **FastAPI + Redis** — Async server with caching and batch query support
- 📊 **Graph Visualization** — Live Neo4j graph rendering via pyvis in Streamlit

---

## 🏗️ Architecture

```
                        ┌─────────────┐
                        │   User      │
                        │  Question   │
                        └──────┬──────┘
                               │
                        ┌──────▼──────┐
                        │  Guardrails │ ← Input validation (7 categories)
                        │   Engine    │
                        └──────┬──────┘
                               │
                        ┌──────▼──────┐
                        │  LangGraph  │ ← Agent with dynamic tool routing
                        │    Agent    │
                        └──┬───┬───┬──┘
                           │   │   │
              ┌────────────┘   │   └────────────┐
              │                │                │
       ┌──────▼──────┐ ┌──────▼──────┐ ┌───────▼───────┐
       │   SQLite    │ │  ChromaDB   │ │    Neo4j      │
       │   (SQL)     │ │  (Vector)   │ │  (Knowledge   │
       │             │ │             │ │    Graph)      │
       └─────────────┘ └─────────────┘ └───────────────┘
       Rankings,        Semantic plot    Relationships,
       stats, counts    similarity       actor networks
```

---

## 🔍 Retrieval Modes

| Mode | Tool | Best For | Example Query |
|------|------|----------|---------------|
| **SQL** (SQLite) | `execute_sql` | Rankings, counts, averages, filtering | "Top 5 movies by rating" |
| **Vector Search** (ChromaDB) | `vector_search` | Plot similarity, thematic queries | "Movies about a heist gone wrong" |
| **Knowledge Graph** (Neo4j) | `query_knowledge_graph` | Relationships, collaborations | "Actors who worked with Nolan" |

---

## 🛡️ Guardrails

| Guardrail | Description |
|-----------|-------------|
| **Data/Content Accuracy** | Grounds responses in DB results, prevents hallucination |
| **Role-Based Restrictions** | User vs Admin with different permissions & rate limits |
| **Data Access & Compliance** | Blocks off-topic queries outside the movie domain |
| **Ethical & Compliance** | Filters biased, offensive, or harmful content |
| **Real-Time Monitoring** | Logs all interactions with timestamps |
| **Security & Privacy** | Blocks SQL injection, prompt injection, and PII |
| **Customizable** | All patterns, keywords, thresholds are configurable |

---

## 📊 Knowledge Graph Stats

```
4,277 nodes  → 999 Movies · 2,709 Actors · 548 Directors · 21 Genres
7,535 relationships
```

Schema:
```
(:Director) -[:DIRECTED]-> (:Movie) -[:HAS_GENRE]-> (:Genre)
(:Actor)    -[:ACTED_IN]-> (:Movie)
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- [Ollama](https://ollama.ai/) with `qwen2.5:7b` + `nomic-embed-text`
- [Neo4j](https://neo4j.com/) Community Edition
- Redis (optional, for caching)

### Installation

```bash
git clone https://github.com/Abhineetraj07/FilmBot.git
cd FilmBot

# Create environment
conda create -n filmbot python=3.11 -y
conda activate filmbot
pip install -r requirements.txt

# Pull Ollama models
ollama pull qwen2.5:7b
ollama pull nomic-embed-text

# Configure environment
cp .env.example .env

# Start Neo4j, then ingest data
neo4j start
python ingest.py
```

### Run the App

```bash
# 🖥️ Streamlit UI (recommended)
python -m streamlit run ui.py
# → http://localhost:8501

# ⌨️ Terminal chat
python chat.py

# 🌐 FastAPI server
python app.py
# → http://localhost:8000/docs
```

---

## 💬 Example Interactions

**SQL Mode:**
```
You: How many movies have an IMDb rating above 8?
Bot: 189 movies have a rating above 8.0. [sql | 10.4s]
```

**Vector Mode:**
```
You: Find movies about a heist gone wrong
Bot: Reservoir Dogs (8.3), Dog Day Afternoon (8.0)... [vector | 12.1s]
```

**Knowledge Graph Mode:**
```
You: Which actors worked with Christopher Nolan?
Bot: Tom Hardy, Christian Bale, Michael Caine, Leonardo DiCaprio... [kg | 8.5s]
```

---

## 📁 Project Structure

```
FilmBot/
├── config.py           # Centralized configuration
├── ingest.py           # Data pipeline: SQLite → ChromaDB + Neo4j
├── tools.py            # 6 tools across 3 retrieval modes
├── agent.py            # LangGraph agent with guardrail integration
├── guardrails.py       # 7 guardrail categories
├── app.py              # FastAPI async server + Redis caching
├── chat.py             # Terminal chat interface
├── ui.py               # Streamlit UI with graph visualization
├── imdb.db             # IMDB dataset (1,000 movies)
├── requirements.txt
└── .env.example
```

---

## 👨‍💻 Author

**Abhineet Raj** · CS @ SRM Institute of Science & Technology  
🌐 [Portfolio](https://aabhineet07-portfolio.netlify.app/) · 🐙 [GitHub](https://github.com/Abhineetraj07)

---

## 📄 License

MIT License
