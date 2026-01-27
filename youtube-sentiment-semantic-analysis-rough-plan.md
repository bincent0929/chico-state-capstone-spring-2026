# YouTube Transcript Sentiment & Semantic Analyzer - MVP Plan

## Project Overview

A CLI tool that analyzes YouTube video transcripts for:
- **Sentiment analysis** (positive/negative/neutral emotional tone)
- **Semantic analysis** (topics, entities, and similarity between videos)
- **LLM-generated summaries** in human-readable prose
- **Persistent storage** for comparing videos across creators

## Tech Stack

| Component | Technology | Justification |
|-----------|------------|---------------|
| Language | Python 3.11+ | Universal for ML/NLP |
| Transcripts | `youtube-transcript-api` | Direct API, no browser |
| Sentiment | VADER | Fast, accurate for social media text |
| NER | spaCy | Industry standard, fast |
| Embeddings | `sentence-transformers` | High-quality semantic vectors |
| Vector DB | ChromaDB | Semantic similarity search |
| Structured DB | SQLite | Simple storage for scores/metadata |
| LLM | LangChain + Ollama (or OpenAI) | Local-first, cost-free dev |
| CLI | Typer + Rich | Clean terminal UX |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CLI Interface (typer)                     │
│  Commands: analyze, compare, similar, list, export          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Orchestrator Service                       │
│  Coordinates all analysis pipelines                         │
└─────────────────────────────────────────────────────────────┘
        │              │              │              │
        ▼              ▼              ▼              ▼
┌───────────┐  ┌────────────┐  ┌───────────┐  ┌────────────┐
│Transcript │  │ Sentiment  │  │ Semantic  │  │    LLM     │
│ Extractor │  │  Analyzer  │  │ Analyzer  │  │ Presenter  │
├───────────┤  ├────────────┤  ├───────────┤  ├────────────┤
│youtube-   │  │VADER scores│  │• spaCy NER│  │LangChain + │
│transcript │  │per segment │  │• Topics   │  │Ollama/API  │
│-api       │  │            │  │• Embeddings│ │            │
└───────────┘  └────────────┘  └───────────┘  └────────────┘
                              │
        ┌─────────────────────┴─────────────────────┐
        ▼                                           ▼
┌───────────────┐                         ┌─────────────────┐
│    SQLite     │                         │    ChromaDB     │
│ (structured)  │                         │   (vectors)     │
├───────────────┤                         ├─────────────────┤
│• Video info   │                         │• Transcript     │
│• Scores       │                         │  embeddings     │
│• Entities     │                         │• Similarity     │
│• Topics       │                         │  search         │
└───────────────┘                         └─────────────────┘
```

## File Structure

```
youtube-sentiment-analyzer/
├── pyproject.toml
├── requirements.txt
├── .env.example
├── README.md
│
├── src/youtube_sentiment/
│   ├── __init__.py
│   ├── __main__.py
│   ├── cli.py                    # Typer commands
│   ├── config.py                 # Settings
│   │
│   ├── models/
│   │   ├── video.py              # VideoInfo, Transcript
│   │   ├── sentiment.py          # SentimentResult, SegmentScore
│   │   ├── semantic.py           # Topics, Entities, Embeddings
│   │   └── report.py             # AnalysisReport
│   │
│   ├── services/
│   │   ├── transcript.py         # TranscriptExtractor
│   │   ├── sentiment.py          # SentimentAnalyzer (VADER)
│   │   ├── semantic.py           # SemanticAnalyzer (spaCy + embeddings)
│   │   ├── llm.py                # LLMPresenter
│   │   └── orchestrator.py       # Main coordinator
│   │
│   ├── storage/
│   │   ├── database.py           # SQLite connection
│   │   ├── vectors.py            # ChromaDB connection
│   │   └── repository.py         # Data access layer
│   │
│   └── utils/
│       └── url_parser.py         # YouTube URL parsing
│
├── tests/
│   ├── test_transcript.py
│   ├── test_sentiment.py
│   ├── test_semantic.py
│   └── fixtures/
│
└── data/                         # gitignored
    ├── sentiment.db
    └── chroma/
```

## Database Schema

### SQLite (structured data)

```sql
CREATE TABLE videos (
    id INTEGER PRIMARY KEY,
    video_id TEXT UNIQUE NOT NULL,
    url TEXT NOT NULL,
    title TEXT,
    channel_name TEXT,
    duration_seconds INTEGER,
    analyzed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE analyses (
    id INTEGER PRIMARY KEY,
    video_id INTEGER REFERENCES videos(id),
    -- Sentiment scores
    compound_score REAL,
    positive REAL,
    negative REAL,
    neutral REAL,
    -- LLM summary
    prose_summary TEXT,
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE entities (
    id INTEGER PRIMARY KEY,
    analysis_id INTEGER REFERENCES analyses(id),
    entity_text TEXT,
    entity_type TEXT,  -- PERSON, ORG, PRODUCT, etc.
    mention_count INTEGER
);

CREATE TABLE topics (
    id INTEGER PRIMARY KEY,
    analysis_id INTEGER REFERENCES analyses(id),
    topic TEXT,
    confidence REAL
);
```

### ChromaDB (vector storage)

- Collection: `video_transcripts`
- Stores: Full transcript embeddings for similarity search
- Metadata: video_id, channel_name, title

## CLI Commands

```bash
# Analyze a single video
youtube-sentiment analyze "https://youtube.com/watch?v=..."

# Compare sentiment across multiple videos
youtube-sentiment compare "URL1" "URL2" "URL3"

# Find semantically similar videos in database
youtube-sentiment similar "URL" --top 5

# List previous analyses
youtube-sentiment list --channel "MrBeast" --limit 10

# Export results
youtube-sentiment export VIDEO_ID --format json
```

## Implementation Phases

### Phase 1: Foundation (Weeks 1-2)
- [ ] Project setup (pyproject.toml, dependencies)
- [ ] URL parser utility
- [ ] TranscriptExtractor service
- [ ] Pydantic models for Video, Transcript
- [ ] Basic CLI skeleton
- [ ] Unit tests for transcript extraction

**Deliverable**: `analyze <url>` prints raw transcript

### Phase 2: Sentiment Analysis (Weeks 3-4)
- [ ] SentimentAnalyzer with VADER
- [ ] Transcript chunking (time-based segments)
- [ ] SentimentResult and SegmentScore models
- [ ] Integrate into CLI with formatted output
- [ ] Unit tests

**Deliverable**: CLI outputs sentiment scores

### Phase 3: Storage Layer (Week 5)
- [ ] SQLite schema and migrations
- [ ] ChromaDB setup for embeddings
- [ ] Repository classes
- [ ] `list` command
- [ ] Skip re-analysis of cached videos

**Deliverable**: Results persist, `list` works

### Phase 4: Semantic Analysis (Week 6)
- [ ] spaCy NER integration
- [ ] Topic extraction via LLM
- [ ] Sentence-transformers embeddings
- [ ] Store in ChromaDB
- [ ] `similar` command for finding related videos

**Deliverable**: Full semantic analysis working

### Phase 5: LLM Summaries (Week 7)
- [ ] Ollama/OpenAI integration via LangChain
- [ ] Prompt engineering for summaries
- [ ] Combine sentiment + semantic in prose
- [ ] Graceful fallback if LLM unavailable

**Deliverable**: Human-readable summaries

### Phase 6: Polish (Week 8)
- [ ] `compare` command with side-by-side output
- [ ] `export` command (JSON/CSV)
- [ ] Rich terminal formatting
- [ ] Error handling
- [ ] Documentation
- [ ] Final testing

**Deliverable**: Complete MVP

## Dependencies

```
# requirements.txt
youtube-transcript-api>=1.0.0
vaderSentiment>=3.3.2
spacy>=3.7.0
sentence-transformers>=2.2.0
chromadb>=0.4.0
langchain>=0.1.0
langchain-community>=0.0.10
ollama>=0.1.0
typer>=0.9.0
rich>=13.0.0
pydantic>=2.0.0
pydantic-settings>=2.0.0
python-dotenv>=1.0.0
```

## Verification Plan

1. **Unit tests**: Each service has isolated tests with fixtures
2. **Integration test**: Full pipeline on a known video
3. **Manual testing**:
   - Analyze 3+ videos from different creators
   - Verify `compare` shows meaningful differences
   - Verify `similar` finds related content
   - Confirm LLM summaries are coherent

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Sentiment engine | VADER | Fast, no training, good for social text |
| Database split | SQLite + ChromaDB | Structured queries + vector similarity |
| LLM default | Ollama (local) | Free, private, works offline |
| NER library | spaCy | Fast, accurate, well-documented |
| Topic extraction | LLM-based | More flexible than LDA for varied content |
