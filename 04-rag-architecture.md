# Retrieval-Augmented Generation (RAG): The Knowledge Foundation

## The Problem RAG Solves

### LLM Limitations Without RAG

1. **Hallucinations:** LLMs generate plausible but false information
2. **Knowledge Cutoff:** Training data is months/years old
3. **No Source Attribution:** Cannot prove where answer came from
4. **Domain Gaps:** Generic training doesn't cover company-specific knowledge
5. **Update Lag:** Retraining takes weeks/months for new information

### RAG Solution

Ground LLM responses in **real, retrievable, attributable company knowledge**.

## How RAG Works (3-Step Process)

### Step 1: Retrieve
Customer query → Semantic search in knowledge base → Top N relevant documents

### Step 2: Augment
Combine retrieved docs + query into rich prompt context

### Step 3: Generate
LLM synthesizes answer grounded in retrieved content

```
┌─────────────────────────────────────────────────┐
│          Knowledge Base                         │
│  (Documents, PDFs, FAQs, Policies)              │
└─────────────────────────────────────────────────┘
                    ↓
        [Chunking & Embedding]
                    ↓
┌─────────────────────────────────────────────────┐
│          Vector Database                        │
│  (Pinecone, FAISS, Milvus, Azure Search)        │
└─────────────────────────────────────────────────┘
                    ↓
        [At Query Time]
                    ↓
User Query → Semantic Search → Top K Documents
                    ↓
    [Prompt Augmentation]
    Retrieved docs + Query
                    ↓
        Azure OpenAI (GPT-4)
                    ↓
        Grounded Answer + Sources
```

## RAG Architecture Components

### 1. Document Ingestion Pipeline

```python
def ingest_documents(documents):
    """
    Process documents for RAG system
    """
    processed_chunks = []
    
    for doc in documents:
        # Step 1: Extract text
        text = extract_text(doc)  # PDF, DOCX, HTML
        
        # Step 2: Clean and normalize
        text = clean_text(text)  # Remove headers, footers, noise
        
        # Step 3: Chunk into manageable pieces
        chunks = chunk_text(
            text, 
            chunk_size=512,  # tokens
            overlap=50       # preserve context at boundaries
        )
        
        # Step 4: Generate embeddings
        for chunk in chunks:
            embedding = embed_text(chunk)
            
            # Step 5: Store with metadata
            processed_chunks.append({
                "text": chunk,
                "embedding": embedding,
                "metadata": {
                    "source": doc["filename"],
                    "page": chunk["page_number"],
                    "section": chunk["section"],
                    "last_updated": doc["last_modified"],
                    "confidence": 1.0
                }
            })
    
    # Step 6: Index in vector database
    vector_db.upsert(processed_chunks)
    
    return len(processed_chunks)
```

### 2. Chunking Strategies

**Why Chunking Matters:**
- Too small (50 tokens): Loses context, poor semantic meaning
- Too large (2000 tokens): Noisy retrieval, exceeds LLM context window
- Sweet spot: 256-512 tokens with 10-20% overlap

**Chunking Methods:**

```python
# Method 1: Fixed-size chunking
def fixed_chunk(text, size=512, overlap=50):
    tokens = tokenize(text)
    chunks = []
    for i in range(0, len(tokens), size - overlap):
        chunk = tokens[i:i + size]
        chunks.append(chunk)
    return chunks

# Method 2: Semantic chunking (better)
def semantic_chunk(text):
    """
    Chunk by semantic boundaries (paragraphs, sections)
    """
    paragraphs = text.split('\n\n')
    chunks = []
    current_chunk = []
    current_size = 0
    
    for para in paragraphs:
        para_size = count_tokens(para)
        
        if current_size + para_size > 512:
            # Save current chunk
            chunks.append(' '.join(current_chunk))
            current_chunk = [para]
            current_size = para_size
        else:
            current_chunk.append(para)
            current_size += para_size
    
    if current_chunk:
        chunks.append(' '.join(current_chunk))
    
    return chunks

# Method 3: Hierarchical chunking (advanced)
def hierarchical_chunk(document):
    """
    Create multiple granularity levels
    """
    return {
        "document_summary": summarize(document),  # 100 tokens
        "section_summaries": [summarize(s) for s in sections],  # 200 tokens each
        "paragraphs": extract_paragraphs(document)  # 512 tokens each
    }
```

**Your ADCB Implementation:**
```
Mobile Banking FAQ:
- Chunk size: 512 tokens
- Overlap: 50 tokens (10%)
- Method: Semantic (paragraph boundaries)
- Total chunks: 50K documents → 500K chunks
- Indexing time: 2 hours
- Storage: 15GB (embeddings + metadata)
```

### 3. Embedding Generation

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_KEY"),
    api_version="2024-02-15-preview",
    azure_endpoint="https://adcb-openai.openai.azure.com/"
)

def embed_text(text):
    """
    Generate embedding for text chunk
    """
    response = client.embeddings.create(
        model="text-embedding-3-large",  # 3072 dimensions
        input=text
    )
    return response.data[0].embedding

# Batch processing for efficiency
def embed_batch(texts, batch_size=100):
    """
    Process multiple texts in batches
    """
    embeddings = []
    for i in range(0, len(texts), batch_size):
        batch = texts[i:i + batch_size]
        response = client.embeddings.create(
            model="text-embedding-3-large",
            input=batch
        )
        embeddings.extend([d.embedding for d in response.data])
    return embeddings
```

### 4. Vector Database Storage

**Your Options:**

```python
# Option 1: FAISS (Local, Fast)
import faiss
import numpy as np

dimension = 3072
index = faiss.IndexFlatL2(dimension)  # L2 distance
index.add(np.array(embeddings))  # Add embeddings

# Search
query_embedding = embed_text(query)
distances, indices = index.search(np.array([query_embedding]), k=5)

# Option 2: Pinecone (Cloud, Scalable)
import pinecone

pinecone.init(api_key="...", environment="us-west1-gcp")
index = pinecone.Index("adcb-knowledge-base")

# Upsert
index.upsert(vectors=[
    ("id1", embedding1, {"text": "...", "source": "..."}),
    ("id2", embedding2, {"text": "...", "source": "..."})
])

# Search
results = index.query(
    vector=query_embedding,
    top_k=5,
    include_metadata=True
)

# Option 3: Azure AI Search (Azure-native)
from azure.search.documents import SearchClient

search_client = SearchClient(
    endpoint="https://adcb-search.search.windows.net",
    index_name="knowledge-base",
    credential=credential
)

# Search with hybrid (keyword + semantic)
results = search_client.search(
    search_text=query,
    vector_queries=[{
        "vector": query_embedding,
        "k": 5,
        "fields": "content_vector"
    }],
    select=["content", "source", "page"],
    top=5
)
```

### 5. Retrieval Strategies

**Basic Retrieval:**
```python
def basic_retrieval(query, k=5):
    """
    Simple semantic search
    """
    query_embedding = embed_text(query)
    results = vector_db.search(query_embedding, top_k=k)
    return results
```

**Hybrid Retrieval (Better):**
```python
def hybrid_retrieval(query, k=5):
    """
    Combine keyword (BM25) + semantic (vector) search
    """
    # Keyword search
    keyword_results = bm25_search(query, top_k=k*2)
    
    # Semantic search
    query_embedding = embed_text(query)
    semantic_results = vector_db.search(query_embedding, top_k=k*2)
    
    # Combine and re-rank
    combined = merge_and_rerank(keyword_results, semantic_results)
    
    return combined[:k]
```

**Query Expansion (Advanced):**
```python
def expanded_retrieval(query, k=5):
    """
    Expand query with synonyms and related terms
    """
    # Generate query variations
    expanded_queries = [
        query,
        add_synonyms(query),
        rephrase_query(query),
        add_context(query)
    ]
    
    # Search with all variations
    all_results = []
    for q in expanded_queries:
        results = basic_retrieval(q, k=k)
        all_results.extend(results)
    
    # Deduplicate and re-rank
    return deduplicate_and_rerank(all_results)[:k]
```

### 6. Prompt Augmentation

```python
def augment_prompt(query, retrieved_docs):
    """
    Combine query + retrieved docs into LLM prompt
    """
    # Format retrieved documents
    context = "\n\n".join([
        f"[Source: {doc['source']}, Page: {doc['page']}]\n{doc['text']}"
        for doc in retrieved_docs
    ])
    
    # Create augmented prompt
    prompt = f"""You are a helpful banking assistant.

CONTEXT (Retrieved from knowledge base):
{context}

CUSTOMER QUERY:
{query}

INSTRUCTIONS:
1. Answer based ONLY on the provided context
2. If the context doesn't contain the answer, say "I don't have that information"
3. Cite sources using [Source: filename, Page: X] format
4. Be concise and accurate

ANSWER:"""
    
    return prompt
```

### 7. Response Generation

```python
def generate_response(query, retrieved_docs):
    """
    Generate LLM response with RAG
    """
    # Augment prompt
    prompt = augment_prompt(query, retrieved_docs)
    
    # Call LLM
    response = client.chat.completions.create(
        model="gpt-4-turbo",
        messages=[
            {"role": "system", "content": "You are a banking assistant."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.3,  # Lower for factual accuracy
        max_tokens=500
    )
    
    answer = response.choices[0].message.content
    
    # Return answer + sources
    return {
        "answer": answer,
        "sources": [
            {"source": doc["source"], "page": doc["page"]}
            for doc in retrieved_docs
        ],
        "confidence": calculate_confidence(answer, retrieved_docs)
    }
```

## RAG Variants (Advanced)

### 1. Basic RAG
Simple retrieval → augmentation → generation

**Use Case:** FAQ, simple Q&A

### 2. Graph RAG
Retrieval from knowledge graph (entities + relationships)

**Use Case:** Multi-hop reasoning
```
Query: "Who is the CEO of the company that acquired WhatsApp?"
→ Retrieve: Facebook acquired WhatsApp
→ Retrieve: Mark Zuckerberg is CEO of Facebook
→ Answer: Mark Zuckerberg
```

### 3. Corrective RAG (CRAG)
Self-evaluate retrieved docs; rewrite query if needed

```python
def corrective_rag(query):
    # Step 1: Initial retrieval
    docs = retrieve(query)
    
    # Step 2: Evaluate relevance
    relevance_scores = [evaluate_relevance(doc, query) for doc in docs]
    
    # Step 3: If low relevance, rewrite query
    if max(relevance_scores) < 0.7:
        rewritten_query = rewrite_query(query)
        docs = retrieve(rewritten_query)
    
    # Step 4: Generate response
    return generate_response(query, docs)
```

### 4. Agentic RAG
RAG with tool use (query decomposition, parallel retrieval, iterative refinement)

```python
def agentic_rag(complex_query):
    # Step 1: Decompose query
    sub_queries = decompose_query(complex_query)
    # ["What is account closure policy?", "What are pending transaction rules?"]
    
    # Step 2: Parallel retrieval
    results = []
    for sub_query in sub_queries:
        docs = retrieve(sub_query)
        results.append(docs)
    
    # Step 3: Synthesize
    all_docs = flatten(results)
    return generate_response(complex_query, all_docs)
```

## Your Experience with RAG

### ADCB Mobile Banking Assistant
```
Architecture:
- Knowledge Base: 50K banking FAQs + policies
- Chunking: 512 tokens, semantic boundaries
- Embeddings: text-embedding-3-large
- Vector DB: FAISS (local deployment)
- Retrieval: Hybrid (BM25 + semantic)
- LLM: GPT-4 Turbo

Performance:
- Latency: 1.6s P95 (100ms retrieval + 1.5s generation)
- Accuracy: 92% (human eval)
- Hallucination rate: 1.8%
- Cost: $0.08/query
- Users: 2M active

Key Learnings:
1. Chunking strategy matters—semantic > fixed-size
2. Hybrid retrieval beats pure semantic by 15%
3. Caching embeddings saves 90% on repeated queries
4. Source attribution builds customer trust
```

### JPMorgan ClearTrade
```
Architecture:
- Knowledge Base: Trade finance regulations + LC templates
- Chunking: Document-aware (preserve LC structure)
- Embeddings: Fine-tuned on trade finance corpus
- Vector DB: Pinecone (multi-region)
- Retrieval: Graph RAG (entity relationships)
- LLM: GPT-4 + fine-tuned classifier

Performance:
- Accuracy: 94% (vs 78% without RAG)
- Processing time: 30 minutes (vs 1-2 days manual)
- Cost savings: $20M in licensing fees
- Error reduction: 60%

Key Learnings:
1. Domain-specific embeddings improve accuracy
2. Graph RAG essential for multi-hop reasoning
3. Human-in-the-loop for high-stakes decisions
4. Audit trails critical for compliance
```

### Hayyak Self-Onboarding
```
Architecture:
- Knowledge Base: Eligibility rules + product catalog
- Chunking: Rule-based (preserve logic)
- Embeddings: text-embedding-3-large
- Vector DB: Azure AI Search
- Retrieval: Filtered by customer segment
- LLM: GPT-4o (vision for document extraction)

Performance:
- Onboarding time: 15-30 minutes (vs 2-5 days)
- Automation rate: 85%
- Accuracy: 96%
- Customer satisfaction: 4.5/5

Key Learnings:
1. Filtered retrieval (by segment) improves relevance
2. Vision models (GPT-4o) for document understanding
3. Confidence thresholds prevent errors
4. Feedback loops improve over time
```

## Production Best Practices

### 1. Chunking
- Use semantic boundaries (paragraphs, sections)
- 256-512 tokens optimal
- 10-20% overlap to preserve context
- Preserve document structure (headers, lists)

### 2. Embedding
- Use latest models (text-embedding-3-large)
- Batch processing for efficiency
- Cache embeddings (don't re-embed same text)
- Version embeddings (track model changes)

### 3. Retrieval
- Hybrid search (keyword + semantic)
- Re-ranking for better relevance
- Metadata filtering (date, source, category)
- Query expansion for better recall

### 4. Generation
- Lower temperature (0.1-0.3) for factual accuracy
- Explicit instructions to cite sources
- Confidence scoring for escalation
- Fact-checking layer (validate against sources)

### 5. Monitoring
- Track retrieval relevance (human eval sample)
- Monitor hallucination rate (automated checks)
- Measure latency (retrieval + generation)
- Log all interactions (audit trail)

## Cost Optimization

### Embedding Costs
```
50K documents × 500 tokens/doc = 25M tokens
25M tokens × $0.00013/1K = $3.25 (one-time)

Updates: 1K docs/month × 500 tokens = 500K tokens/month
500K × $0.00013/1K = $0.065/month

Total: $3.25 + $0.78/year = $4.03/year
```

### Generation Costs
```
Per query:
- Retrieval: 5 docs × 512 tokens = 2560 tokens (input)
- Query: 100 tokens (input)
- Response: 300 tokens (output)

Cost: (2660 × $0.01) + (300 × $0.03) = $0.0266 + $0.009 = $0.0356/query

100K queries/month: $3,560/month
With caching (90% system prompt): $1,780/month
```

## Interview Talking Points

### Technical Deep Dive
*"RAG is my default for contact centers. At ADCB, we built a RAG system handling 2M users with 1.6s latency and 92% accuracy. The key is chunking strategy—we use semantic boundaries, not fixed-size. Hybrid retrieval (BM25 + semantic) beats pure semantic by 15%. We cache embeddings and use prompt caching to reduce costs by 90%."*

### Business Value
*"RAG reduces hallucinations by 80%, provides audit trails (which source answered?), and adapts to knowledge changes without retraining. At JPMorgan, RAG improved trade finance accuracy from 78% to 94%, saving $20M in licensing fees. The ROI is clear: better accuracy, lower cost, faster updates."*

### Compliance & Governance
*"Every RAG response includes source attribution—we can prove where the answer came from. This is critical for regulated industries. We log every interaction: query, retrieved docs, LLM response, customer feedback. Quarterly audits ensure no drift. If accuracy drops, we investigate and update the knowledge base."*

## Common Interview Questions

**Q: How do you handle knowledge base updates?**

**A:** Incremental updates:
1. New documents → chunk → embed → upsert to vector DB
2. Updated documents → re-embed → update vector DB
3. Deleted documents → remove from vector DB
4. No LLM retraining needed (RAG advantage)
5. Validate with test queries before production

**Q: What if retrieval returns irrelevant documents?**

**A:** Multi-layer approach:
1. **Re-ranking:** Score retrieved docs by relevance
2. **Threshold:** Only use docs with score >0.7
3. **Fallback:** If no relevant docs, say "I don't know"
4. **Query rewriting:** Rephrase and retry
5. **Escalation:** Transfer to human if confidence <0.6

**Q: How do you measure RAG quality?**

**A:** Four metrics:
1. **Retrieval Precision:** % of retrieved docs that are relevant
2. **Retrieval Recall:** % of relevant docs that are retrieved
3. **Answer Accuracy:** Human eval on sample (monthly)
4. **Hallucination Rate:** Automated fact-checking (daily)

Target: Precision >85%, Recall >80%, Accuracy >90%, Hallucination <2%
