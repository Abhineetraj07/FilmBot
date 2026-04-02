# FilmBot V2 — Multi-Modal AI Movie Agent

An intelligent movie chatbot that dynamically routes questions across **three retrieval systems** — SQL, Vector Search, and Knowledge Graph — to deliver accurate, grounded answers over an IMDB dataset of 1,000 movies.

![Python](https://img.shields.io/badge/Python-3.11-blue)
![LangGraph](https://img.shields.io/badge/LangGraph-1.0-green)
![FastAPI](https://img.shields.io/badge/FastAPI-0.135-teal)
![Neo4j](https://img.shields.io/badge/Neo4j-5.28-blue)
![ChromaDB](https://img.shields.io/badge/ChromaDB-1.1-orange)

## Architecture

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
       │  (SQL)      │ │  (Vector)   │ │  (Knowledge   │
       │             │ │             │ │    Graph)      │
       └─────────────┘ └─────────────┘ └───────────────┘
       Rankings,        Semantic plot    Relationships,
       stats, counts    similarity      actor-director
                                        networks
```

## Retrieval Modes

| Mode | Tool | Best For | Example |
|------|------|----------|---------|
| **SQL** (SQLite) | `execute_sql` | Rankings, counts, averages, filtering | "Top 5 movies by rating" |
| **Vector Search** (ChromaDB) | `vector_search` | Plot similarity, thematic queries, mood | "Movies about a heist gone wrong" |
| **Knowledge Graph** (Neo4j) | `query_knowledge_graph` | Relationships, collaboration networks | "Actors who worked with Nolan" |

The agent **automatically decides** which retrieval mode to use based on the question — and can combine multiple modes for complex queries.

## Guardrails

Seven guardrail categories ensure safe, accurate, and scoped responses:

| Guardrail | Description |
|-----------|-------------|
| **Data/Content Accuracy** | Ensures responses are grounded in database results, not hallucinated |
| **Role-Based Restrictions** | User vs Admin roles with different permissions and rate limits |
| **Data Access & Compliance** | Blocks off-topic queries outside the movie domain |
| **Ethical & Compliance** | Filters biased, offensive, or harmful content |
| **Real-Time Monitoring** | Logs all interactions with timestamps to `filmbot_interactions.log` |
| **Security & Privacy** | Blocks SQL injection, prompt injection, and PII in input/output |
| **Customizable** | All patterns, keywords, and thresholds are configurable |

## Tech Stack

- **Agent Framework:** LangGraph (StateGraph with conditional tool routing)
- **LLM:** Ollama (qwen2.5:7b, local) — swappable to OpenAI/Groq/Gemini
- **Vector Store:** ChromaDB with Nomic embeddings
- **Knowledge Graph:** Neo4j (Cypher queries)
- **Structured DB:** SQLite (IMDB 1,000 movies)
- **API Server:** FastAPI (async endpoints + Redis caching)
- **UI:** Streamlit with pyvis graph visualization
- **Guardrails:** Custom engine with 7 validation categories

## Knowledge Graph Schema

```
(:Director {name}) -[:DIRECTED]-> (:Movie {name, year, rating, votes, gross, runtime})
(:Actor {name})    -[:ACTED_IN]-> (:Movie)
(:Movie)           -[:HAS_GENRE]-> (:Genre {name})
```

**Stats:** 4,277 nodes (999 Movies, 2709 Actors, 548 Directors, 21 Genres) | 7,535 relationships

## Setup

### Prerequisites

- Python 3.11+
- [Ollama](https://ollama.ai/) with `qwen2.5:7b` and `nomic-embed-text` models
- [Neo4j](https://neo4j.com/) (Community Edition)
- Redis (optional, for caching)

### Installation

```bash
git clone https://github.com/Abhineetraj07/FilmBot.git
cd FilmBot

# Create environment
conda create -n filmbot python=3.11 -y
conda activate filmbot

# Install dependencies
pip install -r requirements.txt

# Pull Ollama models
ollama pull qwen2.5:7b
ollama pull nomic-embed-text

# Start Neo4j
neo4j start

# Ingest data into ChromaDB + Neo4j
python ingest.py
```

### Configuration

Copy `.env.example` to `.env` and update if needed:

```bash
cp .env.example .env
```

## Usage

### Streamlit UI (recommended)

```bash
python -m streamlit run ui.py
```

Opens at `http://localhost:8501` — chat interface with live knowledge graph visualization.

### Terminal Chat

```bash
python chat.py
```

### FastAPI Server

```bash
python app.py
```

API available at `http://localhost:8000/docs` (Swagger UI).

**Endpoints:**
- `POST /query` — Ask a question (with Redis caching)
- `POST /query/batch` — Run multiple questions concurrently
- `GET /health` — Service health check
- `DELETE /cache` — Clear Redis cache

## Project Structure

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
├── requirements.txt    # Python dependencies
└── .env.example        # Environment variables template
```

## Example Interactions

**SQL Mode:**
```
You: How many movies have an IMDb rating above 8?
FilmBot: There are 189 movies with an IMDb rating above 8.0.
  [Tools: execute_sql | Latency: 10.4s]
```

**Vector Search Mode:**
```
You: Find movies about a heist gone wrong
FilmBot: Here are movies about heists gone wrong:
  1. Reservoir Dogs (1992) — Rating: 8.3
  2. Dog Day Afternoon (1975) — Rating: 8.0
  [Tools: vector_search | Latency: 12.1s]
```

**Knowledge Graph Mode:**
```
You: Which actors worked with Christopher Nolan?
FilmBot: Christopher Nolan has worked with Tom Hardy, Christian Bale,
  Michael Caine, Leonardo DiCaprio...
  [Tools: query_knowledge_graph | Latency: 8.5s]
```
