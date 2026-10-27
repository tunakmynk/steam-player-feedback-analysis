**English** | [Türkçe](README.tr.md)

# ⚔️ Crimson Desert — Player Feedback Analysis System

An AI-powered QA (Quality Assurance) assistant that analyzes **Crimson Desert**
player reviews from Steam with NLP techniques, helping the development team
quickly identify bugs and complaints.

> **Goal:** Automatically cluster negative player reviews and make them
> searchable with natural-language queries, so the development team can get
> data-driven answers to questions like "what do players complain about most?"

---

## 📸 Interface Preview

The app provides a chat interface that runs in the browser. A developer asks a
question in natural language; the system analyzes the database and produces a
report:

```
🧑‍💻 You: What do players who refunded the game complain about most?
🤖 Assistant: The issues most often raised by players who refunded are:
   • Optimization problems and FPS drops
   • Repetitive combat mechanics
   • An open world that feels empty and lifeless
```

---

## 🏗️ System Architecture

The project is built on an end-to-end NLP pipeline:

```
Steam API → Data Collection → Preprocessing → Embedding → Clustering → Vector DB → RAG Assistant
```

### Detailed Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PREPARATION                            │
│                                                                     │
│  1. Collected 15,000+ negative reviews from the Steam API           │
│  2. Cleaned reviews and kept English-language ones                  │
│  3. Generated vector embeddings with Sentence Transformers          │
│  4. Reduced dimensions with UMAP → clustered with HDBSCAN           │
│  5. Built 59 topic clusters, each named by an LLM                   │
│  6. Stored cluster summaries + individual reviews in ChromaDB       │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        RAG QUERY SYSTEM                             │
│                                                                     │
│  User Question                                                      │
│       │                                                             │
│       ▼                                                             │
│  [Router LLM] → Analyze the query, build metadata filters           │
│       │          (playtime, refunded, etc.)                         │
│       ▼                                                             │
│  [ChromaDB Vector Search] → Find the most relevant reviews          │
│       │                                                             │
│       ▼                                                             │
│  [Parent-Child Architecture] → Fetch the matching cluster summaries │
│       │                                                             │
│       ▼                                                             │
│  [Answering LLM] → Read the full context and write the report       │
│       │                                                             │
│       ▼                                                             │
│  Response to the Development Team                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Techniques and Technologies

| Category                | Technology                                                      | Description                                                         |
| ----------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Data Collection**     | Steam Web API, Requests                                         | 15,000+ negative reviews collected automatically                    |
| **NLP Embedding**       | Sentence Transformers (`paraphrase-multilingual-MiniLM-L12-v2`) | Reviews converted into 384-dimensional vectors                      |
| **Dimension Reduction** | UMAP                                                            | High-dimensional vectors reduced to 5 dimensions for clustering     |
| **Clustering**          | HDBSCAN                                                         | Reviews automatically grouped into 59 topic clusters                |
| **Vector Database**     | ChromaDB                                                        | Cluster summaries and individual reviews stored as parent-child     |
| **LLM**                 | Google Gemini 2.5 Flash                                         | Query analysis (Router) and report generation (Answerer)            |
| **Web Interface**       | Flask + HTML/CSS/JS                                             | Real-time, chat-based QA interface                                  |
| **API Management**      | Multi-key API rotation                                          | Switches keys automatically when a quota is exceeded                |

---

## 📂 Project Structure

```
Crimson_Desert_NLP_Projesi/
│
├── 1_veri_cekimi.py          # Scraper class that collects reviews from the Steam API
├── ReviewClusterer.py        # UMAP + HDBSCAN clustering module
├── app_core.py               # Core vector search function (modular core)
├── app.py                    # Console-based RAG chatbot (terminal test tool)
├── flask_app.py              # Flask web server (API + interface)
├── web_app.py                # Alternative Streamlit interface
│
├── templates/
│   └── index.html            # HTML/CSS/JS template for the Flask interface
│
├── oyun_rag_veritabani/      # Persistent ChromaDB vector database
│
├── .env                      # API keys (not committed)
├── .gitignore                # Excludes sensitive and large files from Git
├── not.md                    # Development notes
└── README.md                 # This file
```

---

## 🔑 Key Features

### 🚦 Router Architecture (Smart Query Routing)

A user's natural-language question is first analyzed by a **router LLM**, which
extracts metadata filters from it (field names are in Turkish in the code:
`oynama_suresi` = playtime in minutes, `iade_edildi` = refunded):

- `"players with more than 10 hours"` → `{"oynama_suresi": {"$gt": 600}}`
- `"players who refunded"` → `{"iade_edildi": true}`

### 🧬 Parent-Child Document Architecture

The database holds two kinds of documents:

- **Child (`tekil_yorum`):** each individual player review
- **Parent (`kume_ozeti`):** the overall summary of each cluster

When a search returns individual reviews, the summary of the cluster they
belong to is added to the context automatically, so the LLM sees both the
details and the big picture.

### 🔄 Automatic API Key Rotation

When the free Gemini API quota runs out, the system automatically switches to
the next key, allowing uninterrupted analysis.

### 📊 Cluster Criticality Scoring

A **review value score** (playtime × helpful votes) is calculated for each
review, and a **cluster criticality score** for each cluster. These scores
help prioritize the most urgent issues.

---

## 🚀 Installation and Usage

### Requirements

```
Python 3.10+
```

### 1. Clone the Repository

```bash
git clone https://github.com/tunakmynk/steam-player-feedback-analysis.git
cd steam-player-feedback-analysis
```

### 2. Install Dependencies

```bash
pip install flask chromadb google-generativeai sentence-transformers python-dotenv
```

### 3. Set Up API Keys

Create a `.env` file in the project root:

```env
GEMINI_API_KEY_1=your_api_key_here
GEMINI_API_KEY_2=your_api_key_here
GEMINI_API_KEY_3=your_api_key_here
GEMINI_API_KEY_4=your_api_key_here
```

> You can get free API keys from
> [Google AI Studio](https://aistudio.google.com/apikey).

### 4. Run the App

```bash
python flask_app.py
```

Open `http://localhost:5000` in your browser.

---

## 🎮 Example Queries

| Query                                                          | What it does                                                |
| -------------------------------------------------------------- | ----------------------------------------------------------- |
| `What are the general optimization issues in the game?`        | Returns performance-related clusters and reviews            |
| `What do players who refunded complain about most?`            | Searches with the `iade_edildi: true` filter                |
| `What do players who played 10+ hours and refunded say?`       | Combines several metadata filters (`$and`)                  |
| `What do players think about the combat system?`               | Finds and summarizes the combat mechanics cluster           |
| `Which bugs are the most critical?`                            | Ranks by cluster criticality score                          |

---

## 📈 Data Pipeline Summary

```
15,000+ raw negative reviews
    ↓ (language filtering)
1,355 usable English reviews
    ↓ (Sentence Transformers embedding)
1,355 × 384-dimensional vectors
    ↓ (UMAP dimension reduction: 384 → 5)
1,355 × 5-dimensional vectors
    ↓ (HDBSCAN clustering)
59 topic clusters + noise reviews removed
    ↓ (LLM cluster naming and summarization)
59 cluster summaries + individual reviews → ChromaDB
```

---

## 🛠️ Development Process

The project was developed in stages:

1. **Data Collection** — 15,000+ negative reviews collected via the Steam Web API
2. **Preprocessing** — language filtering and cleaning
3. **Vectorization** — embeddings with a multilingual Sentence Transformer model
4. **Clustering** — unsupervised clustering with UMAP + HDBSCAN (59 clusters)
5. **Scoring** — review value and cluster criticality scores
6. **Database** — stored in ChromaDB with a parent-child architecture
7. **RAG System** — dual-LLM architecture: Router + Answerer
8. **Web Interface** — Flask-based real-time chat interface

## Screenshots

<img width="1902" height="912" alt="Crimson Desert - Developer QA Assistant interface" src="https://github.com/user-attachments/assets/0e7ab7c1-267b-4887-b80d-29d98766fcf8" />
<img width="1896" height="907" alt="Crimson Desert - Developer QA Assistant interface 2" src="https://github.com/user-attachments/assets/6744c166-aec8-4e96-9d35-1ea38ce00404" />

---

## 📄 License

This project was developed for educational and portfolio purposes.
