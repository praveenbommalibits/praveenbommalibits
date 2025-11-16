# Azure OpenAI & LLM Integration

## What Is Azure OpenAI Service

Microsoft's managed service for accessing OpenAI models (GPT-4, GPT-4o, GPT-4 Turbo) with **enterprise controls, compliance certifications, and Azure ecosystem integration**.

## Why Azure OpenAI Over Direct OpenAI

| Feature | Azure OpenAI | Direct OpenAI |
|---------|-------------|---------------|
| **Compliance** | HIPAA, FedRAMP, SOC 2, ISO 27001 | Limited certifications |
| **Data Residency** | EU (Frankfurt, Ireland), US, APAC | US-only |
| **Data Privacy** | No training on customer data | Opt-out required |
| **SLA** | 99.9% uptime guarantee | Best effort |
| **Enterprise Support** | Azure support team, TAM | Community + paid tiers |
| **Integration** | Native Azure services (APIM, Search, ML) | API-only |
| **Networking** | Private endpoints, VNet integration | Public internet |
| **Access Control** | Azure AD, RBAC, managed identities | API keys only |
| **Cost Management** | Azure billing, budgets, cost analysis | Separate billing |

### Interview Talking Point
*"Azure OpenAI gives us GPT-4 performance with enterprise compliance. At ADCB, we needed GDPR compliance for EU customers—Azure's Frankfurt deployment solved this. We also use Azure AD for authentication and private endpoints for security. The integration with Azure Search and APIM makes it the obvious choice for enterprise contact centers."*

## Key Capabilities

### 1. GPT-4 Turbo / GPT-4o

**Use Cases in Contact Centers:**
- Complex reasoning (multi-step problem solving)
- Customer context understanding (conversation history)
- Nuanced language generation (empathy, tone)
- Multi-turn dialogue management

**Technical Specs:**
- Context window: 128K tokens (GPT-4 Turbo)
- Response time: 1-3 seconds (typical)
- Cost: $0.01/1K input tokens, $0.03/1K output tokens
- Throughput: 10K tokens/minute (default quota)

**Your Implementation:**
```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_KEY"),
    api_version="2024-02-15-preview",
    azure_endpoint="https://adcb-openai.openai.azure.com/"
)

response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[
        {"role": "system", "content": "You are a banking assistant."},
        {"role": "user", "content": "How do I reset my PIN?"}
    ],
    temperature=0.3,  # Lower for factual responses
    max_tokens=500
)
```

### 2. Embeddings (text-embedding-3-large)

**Use Cases:**
- Semantic search in knowledge bases
- Intent classification
- Similarity matching (find similar past issues)
- Customer query clustering

**Technical Specs:**
- Dimensions: 3072 (large), 1536 (small)
- Cost: $0.00013/1K tokens (large)
- Latency: 50-200ms
- Batch size: Up to 2048 inputs

**Your RAG Implementation:**
```python
# Generate embeddings for knowledge base
def embed_documents(documents):
    embeddings = []
    for doc in documents:
        response = client.embeddings.create(
            model="text-embedding-3-large",
            input=doc["text"]
        )
        embeddings.append(response.data[0].embedding)
    return embeddings

# Query-time embedding
def embed_query(query):
    response = client.embeddings.create(
        model="text-embedding-3-large",
        input=query
    )
    return response.data[0].embedding
```

### 3. Fine-Tuning

**When to Fine-Tune:**
- Domain-specific language (banking terms, product names)
- Consistent formatting requirements
- High-volume, repetitive tasks
- Need to reduce prompt size (cost optimization)

**When NOT to Fine-Tune:**
- Knowledge changes frequently (use RAG instead)
- Low volume (<10K interactions/month)
- Rapid iteration needed (prompt engineering faster)

**Your Experience:**
```
ADCB Mobile Banking Assistant:
- Fine-tuned GPT-3.5 on 50K banking FAQs
- Reduced hallucinations by 35%
- Improved domain terminology accuracy
- Cost: $500 training + $0.012/1K tokens (vs $0.0015 base)
- ROI: Positive after 100K queries

McLaren Trade Finance:
- Fine-tuned for LC (Letter of Credit) document classification
- Training data: 10K annotated trade documents
- Accuracy: 94% (vs 78% with prompt engineering)
- Deployment: 6 weeks (data prep + training + validation)
```

### 4. Prompt Caching

**How It Works:**
- Cache frequently used prompt prefixes
- Reduce token costs by 50% for repeated content
- Automatic in Azure OpenAI (no code changes)

**Use Case:**
```python
# System prompt (cached automatically)
system_prompt = """You are a banking assistant for ADCB.
Our products include: [5000 tokens of product details]
Our policies include: [3000 tokens of policies]
"""

# This system prompt is cached after first use
# Subsequent calls only pay for new user messages
```

**Cost Savings:**
- Without caching: 8K tokens × $0.01 = $0.08 per query
- With caching: 8K tokens (cached) + 100 tokens (user) = $0.001 per query
- Savings: 98.75% on repeated system prompts

### 5. Vision (GPT-4o)

**Use Cases in Contact Centers:**
- ID verification (driver's license, passport)
- Document understanding (invoices, contracts)
- Product identification (customer sends photo)
- Damage assessment (insurance claims)

**Implementation:**
```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Extract information from this ID"},
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:image/jpeg;base64,{base64_image}"}
                }
            ]
        }
    ]
)
```

## Integration Patterns

### Pattern 1: Prompt Engineering (Fastest)

**Pros:**
- No training required
- Instant iteration
- Works with latest models
- Low upfront cost

**Cons:**
- Higher per-query cost (longer prompts)
- Less consistent than fine-tuning
- Requires careful prompt design

**Your Approach:**
```python
# Structured prompt template
PROMPT_TEMPLATE = """You are a contact center AI assistant.

CONTEXT:
Customer: {customer_name}
Account Type: {account_type}
Recent Transactions: {transactions}

KNOWLEDGE BASE:
{retrieved_documents}

CUSTOMER QUERY:
{query}

INSTRUCTIONS:
1. Answer based ONLY on provided knowledge base
2. If unsure, say "I need to transfer you to a specialist"
3. Be empathetic and professional
4. Provide specific policy references

RESPONSE:"""
```

### Pattern 2: RAG (Most Common)

**Pros:**
- Grounds LLM in company knowledge
- Easy to update (just update knowledge base)
- Reduces hallucinations by 80%
- Provides source attribution

**Cons:**
- Requires vector database infrastructure
- Retrieval latency (100-500ms)
- Quality depends on chunking strategy

**Your Architecture:**
```
Customer Query
      ↓
[Embed Query] (50ms)
      ↓
[Vector Search] (100ms)
      ↓
[Retrieve Top 5 Docs] (50ms)
      ↓
[Augment Prompt] (10ms)
      ↓
[Azure OpenAI] (1500ms)
      ↓
Response with Sources
```

### Pattern 3: Fine-Tuning (High Volume)

**Pros:**
- Best accuracy for domain-specific tasks
- Shorter prompts (lower cost at scale)
- Consistent formatting
- Faster inference (no retrieval)

**Cons:**
- Upfront training cost ($500-$5000)
- Slower iteration (days to retrain)
- Requires quality training data (10K+ examples)
- Model drift over time

**Your Decision Framework:**
```
Query Volume < 10K/month → Prompt Engineering
Query Volume 10K-100K/month → RAG
Query Volume > 100K/month + Stable Domain → Fine-Tuning
Knowledge Changes Frequently → Always RAG
```

### Pattern 4: Function Calling (Tool Use)

**Use Case:** LLM needs to call external APIs

**Implementation:**
```python
functions = [
    {
        "name": "get_account_balance",
        "description": "Get customer's current account balance",
        "parameters": {
            "type": "object",
            "properties": {
                "customer_id": {"type": "string"}
            },
            "required": ["customer_id"]
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": "What's my balance?"}],
    functions=functions,
    function_call="auto"
)

# LLM decides to call function
if response.choices[0].message.function_call:
    function_name = response.choices[0].message.function_call.name
    arguments = json.loads(response.choices[0].message.function_call.arguments)
    
    # Execute function
    balance = get_account_balance(arguments["customer_id"])
    
    # Send result back to LLM
    final_response = client.chat.completions.create(
        model="gpt-4-turbo",
        messages=[
            {"role": "user", "content": "What's my balance?"},
            response.choices[0].message,
            {"role": "function", "name": function_name, "content": str(balance)}
        ]
    )
```

## Cost Optimization Strategies

### 1. Model Selection
```
Simple queries (FAQ) → GPT-3.5 Turbo ($0.0005/1K tokens)
Complex reasoning → GPT-4 Turbo ($0.01/1K tokens)
Vision tasks → GPT-4o ($0.005/1K tokens)

Savings: 95% by routing appropriately
```

### 2. Prompt Compression
```python
# Before: 2000 tokens
long_prompt = """[Entire policy document]"""

# After: 500 tokens (use RAG to retrieve only relevant sections)
compressed_prompt = """[Top 3 relevant policy sections]"""

# Savings: 75% token reduction
```

### 3. Caching Strategy
```python
# Cache system prompts (8K tokens)
# Only pay for user messages (100-500 tokens)
# Savings: 90% on repeated interactions
```

### 4. Batch Processing
```python
# Process non-urgent queries in batches (50% discount)
# Real-time: $0.01/1K tokens
# Batch: $0.005/1K tokens
```

### 5. Response Streaming
```python
# Stream responses token-by-token
# Improves perceived latency
# No cost difference, better UX

response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=messages,
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

## Security & Compliance

### 1. Private Endpoints
```python
# Deploy Azure OpenAI in VNet
# No public internet access
# Traffic stays within Azure backbone
```

### 2. Managed Identity
```python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = AzureOpenAI(
    azure_ad_token_provider=credential,
    api_version="2024-02-15-preview",
    azure_endpoint="https://adcb-openai.openai.azure.com/"
)
# No API keys in code
```

### 3. Content Filtering
```python
# Azure OpenAI includes built-in content filters
# Blocks: hate, violence, sexual, self-harm
# Configurable severity levels

response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=messages,
    # Content filter automatically applied
)

# Check if filtered
if response.choices[0].finish_reason == "content_filter":
    # Handle filtered content
    return "I cannot respond to that request"
```

### 4. Data Residency
```
EU Customers → Deploy in Azure West Europe (Ireland)
US Customers → Deploy in Azure East US
APAC Customers → Deploy in Azure Southeast Asia (Singapore)

Ensures GDPR compliance
```

## Monitoring & Observability

### Key Metrics
```python
# Track in Azure Monitor
metrics = {
    "token_usage": {
        "prompt_tokens": 1500,
        "completion_tokens": 300,
        "total_tokens": 1800
    },
    "latency_ms": 2100,
    "cost_usd": 0.024,
    "model": "gpt-4-turbo",
    "customer_id": "12345",
    "intent": "account_inquiry",
    "confidence": 0.92
}
```

### Alerting
```
Alert if:
- Latency > 5 seconds (P95)
- Cost > $1000/day
- Error rate > 1%
- Token usage spike > 50% day-over-day
```

## Your Talking Points

### For Technical Deep Dives
*"At ADCB, we use Azure OpenAI for three use cases: (1) Mobile banking FAQ with RAG—GPT-4 Turbo + FAISS, 1.6s P95 latency. (2) Document extraction with GPT-4o Vision—ID verification, 94% accuracy. (3) Fine-tuned GPT-3.5 for intent classification—50K training examples, 97% accuracy. We deploy in Azure West Europe for GDPR compliance and use managed identities for security."*

### For Cost Discussions
*"Token costs are manageable with the right architecture. We use GPT-3.5 for 70% of queries (simple FAQ), GPT-4 for 30% (complex reasoning). Prompt caching saves us 90% on repeated system prompts. Total cost: $0.08 per customer interaction, vs. $2.50 for human agent. ROI is clear."*

### For Compliance Questions
*"Azure OpenAI is HIPAA and SOC 2 certified. We deploy in region-specific instances (EU data in EU, US data in US). Private endpoints ensure traffic never leaves Azure. Content filters block inappropriate content. Every interaction is logged with customer consent. We're audit-ready."*

## Common Interview Questions

**Q: How do you decide between prompt engineering, RAG, and fine-tuning?**

**A:** Decision tree:
1. Knowledge changes frequently? → RAG
2. High volume (>100K/month) + stable domain? → Fine-tuning
3. Need rapid iteration? → Prompt engineering
4. Need source attribution? → RAG
5. Budget constrained? → Start with prompt engineering, scale to RAG

**Q: How do you handle hallucinations?**

**A:** Multi-layer approach:
1. **RAG:** Ground responses in knowledge base
2. **Fact-checking:** LLM validates its own response against sources
3. **Confidence scoring:** Escalate low-confidence responses
4. **Human review:** Sample 5% of interactions monthly
5. **Feedback loop:** Retrain on hallucination cases

**Q: What's your Azure OpenAI cost optimization strategy?**

**A:** Five levers:
1. Model routing (GPT-3.5 vs GPT-4)
2. Prompt caching (90% savings on system prompts)
3. Batch processing (50% discount for non-urgent)
4. Prompt compression (RAG retrieves only relevant content)
5. Reserved capacity (20% discount for committed usage)

Result: $0.08/interaction (vs $0.25 without optimization)
