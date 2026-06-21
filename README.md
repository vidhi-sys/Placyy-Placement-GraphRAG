# Placyy —> Placement GraphRAG

A GraphRAG (Graph Retrieval-Augmented Generation) system for placement preparation, built on a **company interview knowledge graph**. Enter a company name and the system traverses a Neo4j graph of companies, topics, and problems, then generates a personalized prep roadmap via an LLM hosted on Hugging Face.

---

## What This Project Demonstrates

| Concept | How it appears in this project |
|---|---|
| Graph database fundamentals | Neo4j Aura with a Company → Topic → Problem ontology |
| Cypher query language | Pattern matching with `OPTIONAL MATCH` and `collect()` |
| GraphRAG pipeline | Graph traversal → context injection → LLM generation |
| LLM inference | Hugging Face Inference API (Qwen2.5-7B-Instruct) |
| Streamlit frontend | Interactive web UI with sidebar, spinners, and markdown rendering |
| CLI interface | Simple terminal-based version for quick testing |

---

## Knowledge Graph Ontology

```
NODES
──────────────────────────────────────────────────────────
Company   {name}
Topic     {name}
Problem   {name}

RELATIONSHIPS
──────────────────────────────────────────────────────────
(Company) -[:ASKS]->         (Topic)
(Topic)   -[:HAS_PROBLEM]->  (Problem)
```

---

## Setup

### Prerequisites

| Tool | Version | Notes |
|---|---|---|
| Python | 3.9+ | For running the app |
| Neo4j Aura account | — | Free tier works fine |
| Hugging Face account | — | For an Inference API token |

### 1. Clone the repo

```
git clone https://github.com/<your-username>/placyy-graphrag.git
cd placyy-graphrag
```

### 2. Install Python dependencies

```
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install streamlit neo4j huggingface_hub
```

### 3. Configure credentials

Open `graph_retriever.py` and set your Neo4j Aura URI, username, and password:

```python
driver = GraphDatabase.driver(
    "neo4j+s://your-instance-id.databases.neo4j.io",
    auth=("your-username", "your-password")
)
```

Open `hf_helper.py` and set your Hugging Face token:

```python
client = InferenceClient(
    api_key="hf_your_token_here"
)
```

> **Note:** This setup keeps credentials directly in code for simplicity. If you plan to push this to a public GitHub repo, move them into environment variables first — see Anthropic's docs or ask Claude for a quick walkthrough.

### 4. Load your graph data

Populate your Neo4j Aura instance with `Company`, `Topic`, and `Problem` nodes connected via `ASKS` and `HAS_PROBLEM`, either through the Neo4j Browser or a loader script of your own.

### 5. Verify the connection

```
python test_connection.py
```

Expected output:

```
SUCCESS!
```

### 6. Test graph retrieval

```
python test_retrival.py
```

This prints the topics and problems retrieved for "Amazon".

### 7. Test the LLM

```
python test.py
```

This sends a sample prompt to the Hugging Face model and prints the response.

---

## Usage

### CLI Version

```
python app.py
```

You'll be prompted to enter a company name, and the roadmap is printed to the terminal.

### Streamlit Web App

```
streamlit run streamlit_app.py
```

Open `http://localhost:8501`

In the web app:

* Enter a company name (e.g. *Amazon*, *Google*, *Adobe*, *Microsoft*)
* Click **Generate Roadmap**
* View the retrieved topics/problems from the graph, followed by an AI-generated roadmap covering important topics, coding problems, a 2-week prep plan, interview tips, and common mistakes to avoid

---

## Project Structure

```
placyy-graphrag/
├── app.py                  ← CLI entry point
├── streamlit_app.py        ← Streamlit web frontend
├── graph_retriever.py      ← Neo4j connection + Cypher query
├── hf_helper.py             ← Hugging Face inference wrapper
├── test.py                  ← Quick LLM sanity check
├── test_connection.py       ← Neo4j connectivity check
└── test_retrival.py         ← Graph retrieval sanity check
```

---

## How the GraphRAG Pipeline Works

```
User enters company name
       │
       ▼
┌──────────────────────────────┐
│  1. Graph Retrieval          │  Cypher query matches Company → Topic → Problem
│     (graph_retriever.py)     │  Returns a list of {topic, problems}
└──────────────────┬───────────┘
                   │  structured graph data
                   ▼
┌──────────────────────────────┐
│  2. Prompt Construction       │  Graph data is injected into a mentor-style prompt
│     (app.py / streamlit_app)  │
└──────────────────┬───────────┘
                   │  prompt
                   ▼
┌──────────────────────────────┐
│  3. LLM Answer Generation     │  Qwen2.5-7B-Instruct generates the roadmap
│     (hf_helper.py)            │  grounded in the retrieved graph context
└──────────────────────────────┘
```

---

## Extending the Project

### Adding more companies

Add `Company`, `Topic`, and `Problem` nodes (and the connecting `ASKS` / `HAS_PROBLEM` relationships) to your Neo4j Aura instance.

### Adding a new node type

1. Extend the Cypher query in `graph_retriever.py` to traverse the new relationship
2. Update the prompt template in `app.py` / `streamlit_app.py` to include the new data

### Switching to a different LLM

Swap the model name in `hf_helper.py`, or replace the `huggingface_hub` client with any OpenAI-compatible SDK (OpenAI, Anthropic, Groq, etc.).

---

## Interesting Graph Queries to Try in Neo4j Browser

```
-- All topics asked by a company
MATCH (c:Company {name:'Amazon'})-[:ASKS]->(t:Topic)
RETURN t.name

-- Topics with the most associated problems
MATCH (t:Topic)-[:HAS_PROBLEM]->(p:Problem)
RETURN t.name, count(p) AS problem_count
ORDER BY problem_count DESC

-- Companies that share a topic
MATCH (c1:Company)-[:ASKS]->(t:Topic)<-[:ASKS]-(c2:Company)
WHERE c1.name < c2.name
RETURN c1.name, c2.name, t.name

-- Full topic + problem breakdown for a company
MATCH (c:Company {name:'Google'})-[:ASKS]->(t:Topic)
OPTIONAL MATCH (t)-[:HAS_PROBLEM]->(p:Problem)
RETURN t.name, collect(p.name) AS problems
```

---
<img width="450" height="250" alt="Screenshot 2026-06-20 233428" src="https://github.com/user-attachments/assets/c2e76f9e-cfe4-45cd-9014-2f4ca53eb678" />


*Built with Neo4j Aura, Hugging Face, and Streamlit.*
