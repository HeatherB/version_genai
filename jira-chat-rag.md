# JIRA_Chat: JIRA Ticket Q&A System
## Case Study: RAG-Powered IT Support Knowledge Base

---

## 🎯 Problem

### The Challenge
IT support teams accumulate vast amounts of knowledge in their ticketing systems (JIRA, ServiceNow, etc.), but this knowledge is **trapped in unstructured data**:

- **Ticket histories** contain solutions to recurring problems
- **Resolution notes** document what actually worked
- **Incident patterns** reveal systemic issues

Yet when a new ticket comes in, support agents often:
- Manually search through old tickets
- Rely on tribal knowledge
- Re-solve problems that were already fixed

### Business Impact
- **Duplicated effort** — solving the same problems repeatedly
- **Inconsistent resolution quality** — depends on who handles the ticket
- **Lost institutional knowledge** — when experienced staff leave
- **Slow time-to-resolution** — searching instead of solving

---

## 🔧 Approach

### Solution Design
Build a **Retrieval-Augmented Generation (RAG) system** that:
1. Ingests historical JIRA ticket data
2. Creates searchable vector embeddings
3. Retrieves relevant past tickets for any new query
4. Generates contextual answers using GPT-4

### RAG Architecture Philosophy

```
┌─────────────────────────────────────────────────────────────┐
│                    INDEXING PHASE (Offline)                  │
│                                                              │
│   JIRA Data    →    Chunking    →    Embeddings    →  FAISS │
│   (CSV/PDF)         (5000 char)      (384-dim)        Index  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    QUERY PHASE (Real-time)                   │
│                                                              │
│   User Query  →  Embed Query  →  Similarity   →  Top-K     │
│                                    Search        Chunks     │
│                                                    │        │
│                                                    ▼        │
│                     LLM  ←  Context + Query  ←  Format     │
│                      │                                      │
│                      ▼                                      │
│                   Answer                                    │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Decisions

**1. Embedding Model Selection**
- Using `multi-qa-MiniLM-L6-cos-v1` — optimized for question-answering tasks
- 384 dimensions — good balance of quality and performance
- Cosine similarity — handles varied text lengths well

**2. Chunking Strategy**
- Fixed 5000-character chunks from JSON-serialized records
- Preserves ticket context integrity
- Manageable for embedding generation

**3. Caching Strategy**
- Pickle-based DataFrame caching (avoid re-parsing CSVs)
- Embedding cache with tuple keys (text, model)
- Persistent FAISS index to disk

---

## 🏗️ Architecture

### System Components

```
┌──────────────────────────────────────────────────────────────┐
│                         JIRA_CHAT SYSTEM                         │
│                                                               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   DATA      │    │  EMBEDDING  │    │   VECTOR    │      │
│  │   LAYER     │    │   LAYER     │    │   STORE     │      │
│  │             │    │             │    │             │      │
│  │ • CSV/PDF   │───▶│ • Sentence  │───▶│ • FAISS     │      │
│  │ • Pandas    │    │   Transform │    │ • IndexFlat │      │
│  │ • Chunking  │    │ • Caching   │    │ • L2 dist   │      │
│  └─────────────┘    └─────────────┘    └──────┬──────┘      │
│                                               │              │
│                                               ▼              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   USER      │    │    RAG      │    │   LLM       │      │
│  │   INPUT     │───▶│   CHAIN     │───▶│   LAYER     │      │
│  │             │    │             │    │             │      │
│  │ • Query     │    │ • Retrieve  │    │ • GPT-4     │      │
│  │ • CLI       │    │ • Context   │    │ • GenAI     │      │
│  │             │    │ • Prompt    │    │   Center    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

### Data Schema
JIRA ticket data includes:

| Field | Description |
|-------|-------------|
| `Number` | Ticket ID |
| `Task type` | Category of work |
| `State` | Current status |
| `Short description` | Issue summary |
| `Opened` / `Closed` | Timestamps |
| `Resolution notes` | How it was resolved |
| `Assigned to` | Handler |
| `Severity` | Priority level |
| `Description` | Full details |

### Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Embeddings** | SentenceTransformers | Multi-qa-MiniLM-L6-cos-v1 |
| **Vector Store** | FAISS | IndexFlatL2 similarity search |
| **LLM** | GPT-4 | Via GenAICenter API |
| **Prompts** | LangChain | ChatPromptTemplate |
| **Data** | Pandas | DataFrame processing |
| **Caching** | Pickle | Embeddings & DataFrames |
| **HTTP** | httpx | Async API calls |

---

## 💻 Code Patterns

### Pattern 1: Efficient Embedding Caching
Avoid redundant embedding generation with tuple-key caching:

```python
embedding_cache = {}

def embedding_from(string: str, model=model, cache=embedding_cache) -> list:
    """Generate or retrieve cached embedding"""
    
    # Tuple key: (text, model) ensures uniqueness
    if (string, model) not in embedding_cache.keys():
        # Generate new embedding
        embedding_cache[(string, model)] = model.encode(string)
        
        # Persist to disk immediately
        with open(embedding_cache_path, "wb") as f:
            pickle.dump(embedding_cache, f)
            
    return embedding_cache[(string, model)]
```

### Pattern 2: FAISS Index Management
Persistent vector store with lazy loading:

```python
import faiss
import numpy as np

def create_faiss_index(embeddings):
    """Initialize FAISS index from embedding array"""
    
    if embeddings.size == 0:
        raise ValueError("Embeddings array is empty.")
    
    # Get embedding dimension (384 for MiniLM)
    dimension = embeddings.shape[1]
    
    # IndexFlatL2 = exact L2 distance search
    index = faiss.IndexFlatL2(dimension)
    
    # Add all embeddings to index
    index.add(embeddings)
    
    return index

def save_faiss_index(index, path="faiss_index.bin"):
    """Persist index to disk"""
    faiss.write_index(index, path)

def load_faiss_index(path="faiss_index.bin"):
    """Load existing index or return None"""
    if os.path.exists(path):
        return faiss.read_index(path)
    return None
```

### Pattern 3: Flexible Data Ingestion
Support multiple data formats with unified output:

```python
def load_csv_to_dataframe(csv_file_path):
    """Load structured CSV data"""
    if os.path.exists(csv_file_path):
        df = pd.read_csv(csv_file_path)
        logger.info("CSV file loaded successfully.")
        return df
    return None

def load_pdf_to_dataframe(pdf_file_path):
    """Extract text from PDF documents"""
    try:
        rows = []
        with pdfplumber.open(pdf_file_path) as pdf:
            for page in pdf.pages:
                text = page.extract_text()
                if text:
                    rows.extend(text.split("\n"))
        
        df = pd.DataFrame(rows, columns=["Raw Data"])
        return df
    except Exception as e:
        logger.error(f"Error loading PDF: {e}")
        return None

def dataframe_to_json_chunks(df, chunk_size=5000):
    """Convert DataFrame to searchable text chunks"""
    
    # Serialize entire DataFrame to JSON
    data_str = df.to_json(orient='records')
    
    # Split into fixed-size chunks
    data_chunks = [
        data_str[i:i + chunk_size] 
        for i in range(0, len(data_str), chunk_size)
    ]
    
    return data_chunks
```

### Pattern 4: Vector Similarity Retrieval
Top-K retrieval with bounds checking:

```python
def retrieve(query_embedding, index, data_chunks, k=5):
    """Find k most similar chunks to query"""
    
    logger.info(f"Searching for top-{k} similar chunks")
    
    # FAISS search returns distances (D) and indices (I)
    D, I = index.search(np.array([query_embedding]), k)
    
    logger.info(f"Distances: {D}, Indices: {I}")
    
    # Safely retrieve chunks by index
    retrieved_chunks = []
    for i in I[0]:
        if i < len(data_chunks):
            retrieved_chunks.append(data_chunks[i])
        else:
            logger.warning(f"Index {i} out of bounds for {len(data_chunks)} chunks")
    
    return retrieved_chunks
```

### Pattern 5: Complete RAG Chain
Orchestrating retrieval → context → generation:

```python
async def rag_chain(query):
    """Full RAG pipeline: retrieve, format, generate"""
    
    logger.info(f"Processing query: {query}")
    
    # 1. RETRIEVE: Find relevant chunks
    query_embedding = embedding_from(query)
    retrieved_chunks = retrieve(query_embedding, index, data_chunks)
    
    # 2. FORMAT: Build context string
    context = format_docs(retrieved_chunks)
    context = truncate_context(context, query, max_tokens=16385)
    
    # 3. PROMPT: Inject context into template
    prompt_text = prompt.format(
        question=query, 
        context=context,
        local_column_names=", ".join(local_column_names)
    )
    
    # 4. GENERATE: Call LLM
    response = await llm(prompt_text)
    
    # 5. POST-PROCESS: Clean up response
    response = post_process_response(response, query)
    
    return response

def truncate_context(context, question, max_tokens=16385):
    """Ensure context fits within token limit"""
    
    total_tokens = len(context.split()) + len(question.split())
    
    if total_tokens > max_tokens:
        context_tokens = context.split()
        allowed_tokens = max_tokens - len(question.split())
        context = " ".join(context_tokens[:allowed_tokens])
    
    return context
```

### Pattern 6: Async LLM Integration
Non-blocking API calls with httpx:

```python
async def llm(prompt):
    """Async call to genAI endpoint"""
    
    url = "#####"
    
    headers = {
        "api-key": api_key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "model": "gpt-4",
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 500
    }
    
    async with httpx.AsyncClient() as client:
        response = await client.post(
            url, 
            headers=headers, 
            json=payload, 
            timeout=30
        )
        response.raise_for_status()
        data = response.json()
        
        return data['choices'][0]['message']['content']
```

---

## 📊 Results

### System Capabilities

| Feature | Implementation |
|---------|----------------|
| **Query Types** | Natural language questions about past incidents |
| **Data Sources** | CSV exports, PDF documents |
| **Response Time** | ~2-5 seconds (including LLM call) |
| **Embedding Dim** | 384 dimensions |
| **Top-K Retrieval** | 5 similar chunks by default |

### Performance Characteristics

| Metric | Value | Notes |
|--------|-------|-------|
| **Index Build Time** | ~1-2 min | For 100+ tickets |
| **Query Latency** | 2-5s | Dominated by LLM call |
| **Embedding Cache Hit** | ~90%+ | After warmup |
| **Storage** | ~50MB | FAISS index + cache |

### Sample Queries

```
Q: "What issues have users reported with email attachments?"
A: Based on the incident data, users have reported email attachment 
   issues including... [retrieves relevant past tickets]

Q: "How was ticket INC0012345 resolved?"
A: Ticket INC0012345 was resolved by... [pulls resolution notes]

Q: "What are common password reset procedures?"
A: Looking at past incidents, the standard password reset involves...
```

### Architecture Trade-offs

| Decision | Pros | Cons |
|----------|------|------|
| **Fixed chunking** | Simple, predictable | May split related content |
| **FAISS IndexFlatL2** | Exact search, no training | Slower for large datasets |
| **Pickle caching** | Fast, simple | Not distributed-friendly |
| **Async LLM calls** | Non-blocking | Adds complexity |

---

## 🔑 Key Takeaways

### RAG Best Practices Demonstrated

1. **Cache aggressively** — Embeddings are expensive; compute once, reuse forever

2. **Persist indexes** — FAISS indexes survive restarts; no need to rebuild

3. **Truncate safely** — Token limits are real; fail gracefully with context trimming

4. **Log everything** — RAG debugging requires visibility into retrieval quality

5. **Separate concerns** — Data ingestion, embedding, retrieval, and generation are distinct stages

### Patterns to Reuse

| Pattern | Use Case |
|---------|----------|
| Tuple-key embedding cache | Any embedding-heavy application |
| FAISS index persistence | Vector stores that outlive sessions |
| Multi-format data loaders | Systems ingesting varied document types |
| Async LLM wrapper | High-throughput AI applications |
| Context truncation | Any RAG system hitting token limits |

### Extension Opportunities

- **Semantic chunking** — Split by meaning, not character count
- **Hybrid search** — Combine vector + keyword search
- **Re-ranking** — Use cross-encoder to improve top-K quality
- **Streaming responses** — Show results as they generate
- **Feedback loop** — Learn which retrievals led to good answers

---

## 📁 Project Structure

```
JIRA_Chat/
├── main.py                    # Entry point, RAG chain orchestration
├── config.py                  # Environment setup
├── requirements.txt           # Dependencies
├── helpers/
│   ├── config.py              # API key loading
│   ├── data_processing.py     # CSV/PDF ingestion, chunking
│   ├── faiss_utils.py         # Vector index management
│   ├── pickle_utils.py        # Cache persistence
│   └── logger.py              # Logging configuration
├── dataset/
│   ├── top100cleaned.csv      # Sample JIRA ticket data
│   └── output_incident.xlsx   # Incident exports
├── faiss_index.bin            # Persisted vector index
├── dataframe_cache.pkl        # Cached DataFrame
└── embedding_cache.pkl        # Cached embeddings
```

---

## 🛠️ Technologies Used

| Category | Technology |
|----------|------------|
| **Vector Search** | FAISS (IndexFlatL2) |
| **Embeddings** | SentenceTransformers (MiniLM) |
| **LLM** | GPT-4 via GenAICenter |
| **Prompts** | LangChain ChatPromptTemplate |
| **Data Processing** | Pandas, pdfplumber |
| **Async HTTP** | httpx |
| **Caching** | Python pickle |
| **Language** | Python 3.x |

---

## 📚 Related Resources

- [FAISS Documentation](https://faiss.ai/)
- [SentenceTransformers](https://www.sbert.net/)
- [LangChain RAG Guide](https://python.langchain.com/docs/use_cases/question_answering/)
- [RAG Paper (Lewis et al.)](https://arxiv.org/abs/2005.11401)

---

*Case study demonstrating RAG patterns for IT support knowledge management*
