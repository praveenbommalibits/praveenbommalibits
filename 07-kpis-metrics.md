# KPIs & Metrics Framework for AI Contact Centers

## Executive Summary

Success in AI-enabled contact centers requires tracking metrics across four dimensions:
1. **Business Impact** (Revenue, Cost, Customer Satisfaction)
2. **Operational Efficiency** (Speed, Quality, Capacity)
3. **AI Quality** (Accuracy, Reliability, Performance)
4. **Compliance & Risk** (Governance, Audit, Fairness)

---

## 1. Business KPIs (C-Suite Cares)

### 1.1 Containment Rate
**Definition:** % of customer interactions resolved by AI without human escalation

**Formula:** `(AI-resolved interactions / Total interactions) × 100`

**Targets:**
- Industry baseline: 45-55%
- Good: 65-70%
- Excellent: 75-80%
- World-class: 85%+

**Your Experience:**
- ADCB Mobile Banking: 73% containment
- JPMorgan ClearTrade: 68% containment (complex domain)
- Hayyak Onboarding: 85% containment (narrow scope)

**Monitoring:**
```python
# Daily containment tracking
def calculate_containment_rate(date):
    total = db.count_interactions(date)
    ai_resolved = db.count_interactions(date, escalated=False)
    return (ai_resolved / total) * 100

# Alert if drops below threshold
if containment_rate < 70:
    alert_team("Containment rate dropped to {containment_rate}%")
```

**Improvement Levers:**
1. Expand AI scope (add more intents)
2. Improve RAG accuracy (better retrieval)
3. Increase confidence threshold (fewer false escalations)
4. Fine-tune on edge cases

---

### 1.2 Customer Satisfaction (CSAT)
**Definition:** Customer rating of interaction quality (1-5 scale)

**Formula:** `Average rating across all interactions`

**Targets:**
- Industry baseline: 3.5-3.8
- Good: 4.0-4.2
- Excellent: 4.3-4.5
- World-class: 4.6+

**Your Experience:**
- ADCB Mobile Banking: 4.3/5
- JPMorgan ClearTrade: 4.3/5
- Hayyak Onboarding: 4.5/5

**Segmentation:**
```python
# CSAT by interaction type
csat_by_intent = {
    "account_inquiry": 4.5,
    "password_reset": 4.6,
    "complaint": 3.2,  # Low—needs improvement
    "product_inquiry": 4.1
}

# CSAT by channel
csat_by_channel = {
    "voice": 4.2,
    "chat": 4.4,
    "email": 4.0
}

# CSAT by customer segment
csat_by_segment = {
    "retail": 4.3,
    "business": 4.1,  # Lower—investigate
    "vip": 4.6
}
```

**Root Cause Analysis:**
```python
# Analyze low-CSAT interactions
low_csat = db.query("""
    SELECT * FROM interactions
    WHERE csat_score < 3
    ORDER BY date DESC
    LIMIT 1000
""")

# Categorize issues
issues = categorize_issues(low_csat)
# Output: {"hallucination": 35%, "wrong_intent": 25%, "slow_response": 20%, ...}
```

---

### 1.3 Cost Per Contact
**Definition:** Total operational cost divided by number of interactions

**Formula:** `(AI costs + Human agent costs + Infrastructure) / Total interactions`

**Breakdown:**
```
Human Agent Cost:
- Average: $2.50/interaction
- Components: Salary ($50K/year), benefits, training, infrastructure
- Handle time: 8-10 minutes

AI Cost:
- Average: $0.05-$0.15/interaction
- Components: LLM tokens, embeddings, infrastructure, monitoring
- Handle time: 1-3 minutes

Hybrid (70% AI, 30% Human):
- Cost: (0.7 × $0.10) + (0.3 × $2.50) = $0.82/interaction
- Savings: 67% vs. human-only
```

**Your Experience:**
```
ADCB Mobile Banking:
- Before AI: $2.50/interaction (100% human)
- After AI: $0.82/interaction (70% AI, 30% human)
- Annual volume: 10M interactions
- Annual savings: $16.8M

JPMorgan ClearTrade:
- Before AI: $150/document (manual processing)
- After AI: $5/document (AI + human review)
- Annual volume: 100K documents
- Annual savings: $14.5M
```

**Cost Optimization Tracking:**
```python
# Monthly cost breakdown
monthly_costs = {
    "llm_tokens": 45000,      # Azure OpenAI
    "embeddings": 2000,       # Vector generation
    "vector_db": 8000,        # Pinecone/FAISS
    "infrastructure": 15000,  # Compute, storage
    "monitoring": 3000,       # Observability tools
    "human_agents": 180000,   # 30% escalations
    "total": 253000
}

# Cost per interaction
cost_per_interaction = monthly_costs["total"] / monthly_interactions
# Target: <$1.00
```

---

### 1.4 Net Promoter Score (NPS)
**Definition:** Likelihood customer would recommend service (0-10 scale)

**Formula:** `% Promoters (9-10) - % Detractors (0-6)`

**Targets:**
- Industry baseline: 20-30
- Good: 40-50
- Excellent: 50-60
- World-class: 70+

**Your Experience:**
- ADCB Mobile Banking: NPS improved from 35 to 52 after AI deployment

---

### 1.5 Customer Effort Score (CES)
**Definition:** How easy was it to resolve issue (1-7 scale, lower is better)

**Formula:** `Average effort rating`

**Targets:**
- Industry baseline: 5-6
- Good: 3-4
- Excellent: 2-3
- World-class: <2

**Correlation with AI:**
```
AI-resolved interactions: CES 2.8 (easy)
Human-resolved interactions: CES 4.5 (moderate effort)
Multiple transfers: CES 6.2 (high effort)
```

---

## 2. Operational KPIs (COO Cares)

### 2.1 Average Handle Time (AHT)
**Definition:** Average duration of customer interaction

**Targets:**
```
Voice:
- Human: 8-10 minutes
- AI: 2-3 minutes
- Improvement: 70-75%

Chat:
- Human: 12-15 minutes (multi-tasking)
- AI: 1-2 minutes
- Improvement: 85-90%

Email:
- Human: 20-30 minutes
- AI: 5-10 minutes
- Improvement: 60-70%
```

**Your Experience:**
```
ADCB Mobile Banking:
- Before: 8.2 minutes average
- After: 2.1 minutes (AI), 9.5 minutes (human escalations)
- Blended: 4.1 minutes (70% AI, 30% human)
- Improvement: 50%
```

---

### 2.2 First Contact Resolution (FCR)
**Definition:** % of issues resolved in first interaction (no follow-up needed)

**Formula:** `(Resolved in first contact / Total interactions) × 100`

**Targets:**
- Industry baseline: 65-70%
- Good: 75-80%
- Excellent: 85-90%
- World-class: 90%+

**AI Impact:**
```
AI-resolved: 88% FCR (high—AI has full context)
Human-resolved: 72% FCR (lower—may need research)
```

---

### 2.3 Occupancy Rate
**Definition:** % of time agents are actively handling interactions

**Formula:** `(Handle time + After-call work) / Total logged-in time × 100`

**Targets:**
- Industry baseline: 70-75%
- Good: 80-85%
- Excellent: 85-90%
- Optimal: 85% (higher = burnout risk)

**AI Impact:**
```
Before AI: 72% occupancy (agents handle all calls)
After AI: 87% occupancy (agents handle only complex calls)
Benefit: Higher productivity, less idle time
```

---

### 2.4 Abandonment Rate
**Definition:** % of customers who hang up before reaching agent/bot

**Formula:** `(Abandoned calls / Total calls) × 100`

**Targets:**
- Industry baseline: 5-8%
- Good: 3-5%
- Excellent: 1-3%
- World-class: <1%

**AI Impact:**
```
Before AI: 6.5% abandonment (long wait times)
After AI: 1.8% abandonment (instant bot response)
Improvement: 72%
```

---

### 2.5 Service Level Agreement (SLA)
**Definition:** % of interactions answered within target time

**Formula:** `(Answered within X seconds / Total interactions) × 100`

**Targets:**
```
Voice: 80% answered within 20 seconds
Chat: 90% answered within 30 seconds
Email: 95% answered within 4 hours
```

**AI Impact:**
```
Voice: 95% within 20 seconds (AI answers instantly)
Chat: 99% within 30 seconds (no queue)
Email: 98% within 1 hour (AI processes immediately)
```

---

## 3. AI Quality KPIs (Engineering Team Cares)

### 3.1 Hallucination Rate
**Definition:** % of AI responses containing false/ungrounded information

**Formula:** `(Hallucinated responses / Total AI responses) × 100`

**Targets:**
- Acceptable: <2%
- Good: <1%
- Excellent: <0.5%
- Critical systems: <0.1%

**Detection Methods:**
```python
# Method 1: Automated fact-checking
def detect_hallucination(response, retrieved_docs):
    check_prompt = f"""
    Response: {response}
    Sources: {retrieved_docs}
    
    Is every claim in the response supported by sources?
    List any unsupported claims.
    """
    result = llm.check(check_prompt)
    return result.has_unsupported_claims

# Method 2: Human evaluation (sample)
def human_eval_hallucination():
    sample = random.sample(interactions, 1000)
    for interaction in sample:
        human_rating = human_evaluator.rate(interaction)
        if human_rating == "hallucination":
            log_hallucination(interaction)
    
    return hallucination_count / 1000

# Method 3: Customer feedback
def feedback_based_detection():
    negative_feedback = db.query("""
        SELECT * FROM interactions
        WHERE feedback_type = 'incorrect_information'
    """)
    return len(negative_feedback) / total_interactions
```

**Your Experience:**
```
ADCB Mobile Banking: 1.8% hallucination rate
- Detection: Weekly human eval (1000 samples)
- Mitigation: RAG + fact-checking layer
- Improvement: From 4.2% to 1.8% over 6 months

JPMorgan ClearTrade: 0.8% hallucination rate
- Detection: Automated fact-checking (every response)
- Mitigation: Fine-tuned model + strict RAG
- Critical: Trade finance requires high accuracy
```

---

### 3.2 Intent Recognition Accuracy
**Definition:** % of customer intents correctly identified

**Formula:** `(Correct intent classifications / Total interactions) × 100`

**Targets:**
- Acceptable: 85-90%
- Good: 90-95%
- Excellent: 95-98%
- World-class: 98%+

**Measurement:**
```python
# Confusion matrix
from sklearn.metrics import confusion_matrix, classification_report

y_true = [actual_intent for interaction in test_set]
y_pred = [predicted_intent for interaction in test_set]

print(classification_report(y_true, y_pred))

# Output:
#                   precision  recall  f1-score  support
# account_inquiry      0.96     0.94     0.95      500
# password_reset       0.98     0.97     0.98      300
# complaint            0.82     0.79     0.80      200
# product_inquiry      0.91     0.93     0.92      400
```

**Your Experience:**
```
ADCB Mobile Banking: 93% intent accuracy
- Method: Fine-tuned BERT classifier
- Training data: 50K labeled interactions
- Validation: Monthly human eval

JPMorgan ClearTrade: 97% intent accuracy
- Method: Fine-tuned GPT-3.5
- Training data: 10K trade finance documents
- Domain-specific: High accuracy due to narrow scope
```

---

### 3.3 Retrieval Precision & Recall
**Definition:** Quality of RAG retrieval

**Formulas:**
- Precision: `(Relevant retrieved docs / Total retrieved docs) × 100`
- Recall: `(Relevant retrieved docs / Total relevant docs) × 100`

**Targets:**
```
Precision:
- Acceptable: 70-80%
- Good: 80-90%
- Excellent: 90-95%

Recall:
- Acceptable: 70-80%
- Good: 80-90%
- Excellent: 90-95%
```

**Measurement:**
```python
# Evaluate retrieval quality
def evaluate_retrieval(query, retrieved_docs, ground_truth_docs):
    relevant_retrieved = set(retrieved_docs) & set(ground_truth_docs)
    
    precision = len(relevant_retrieved) / len(retrieved_docs)
    recall = len(relevant_retrieved) / len(ground_truth_docs)
    f1 = 2 * (precision * recall) / (precision + recall)
    
    return {
        "precision": precision,
        "recall": recall,
        "f1": f1
    }

# Run on test set
test_results = [
    evaluate_retrieval(q, retrieved, ground_truth)
    for q, retrieved, ground_truth in test_set
]

avg_precision = mean([r["precision"] for r in test_results])
avg_recall = mean([r["recall"] for r in test_results])
```

**Your Experience:**
```
ADCB Mobile Banking:
- Precision: 88%
- Recall: 82%
- Method: Hybrid retrieval (BM25 + semantic)

JPMorgan ClearTrade:
- Precision: 92%
- Recall: 87%
- Method: Graph RAG (entity relationships)
```

---

### 3.4 Latency (P50, P95, P99)
**Definition:** Response time distribution

**Targets:**
```
Voice (real-time):
- P50: <1.5s
- P95: <2.5s
- P99: <4.0s

Chat (near real-time):
- P50: <2.0s
- P95: <3.5s
- P99: <5.0s

Email (async):
- P50: <10s
- P95: <30s
- P99: <60s
```

**Breakdown:**
```python
# Latency components
latency_breakdown = {
    "speech_to_text": 100,      # ms
    "intent_recognition": 50,    # ms
    "retrieval": 500,            # ms (bottleneck)
    "llm_generation": 1500,      # ms
    "text_to_speech": 200,       # ms
    "total": 2350                # ms (P50)
}

# Optimization targets
optimized_breakdown = {
    "speech_to_text": 80,        # Streaming
    "intent_recognition": 30,    # Cached model
    "retrieval": 200,            # Multi-index + cache
    "llm_generation": 1200,      # Prompt optimization
    "text_to_speech": 150,       # Streaming
    "total": 1660                # ms (30% improvement)
}
```

**Your Experience:**
```
ADCB Mobile Banking:
- P50: 1.6s
- P95: 2.8s
- P99: 4.2s
- Optimization: Cached embeddings, multi-region deployment

JPMorgan ClearTrade:
- P50: 8.5s (document processing)
- P95: 15.2s
- P99: 25.0s
- Acceptable: Not real-time, batch processing
```

---

### 3.5 Token Cost Per Interaction
**Definition:** Average LLM token cost per customer interaction

**Formula:** `(Total token cost / Total interactions)`

**Targets:**
```
Simple queries (FAQ): $0.01-$0.02
Medium complexity: $0.03-$0.05
Complex reasoning: $0.08-$0.12
```

**Breakdown:**
```python
# Token usage per interaction
token_usage = {
    "system_prompt": 500,        # Cached (90% savings)
    "retrieved_context": 2000,   # RAG docs
    "user_query": 50,            # Customer input
    "conversation_history": 300, # Previous turns
    "llm_response": 200,         # Generated output
    "total_input": 2850,
    "total_output": 200
}

# Cost calculation
cost = (
    (token_usage["total_input"] * 0.00001) +  # $0.01/1K input tokens
    (token_usage["total_output"] * 0.00003)   # $0.03/1K output tokens
)
# = $0.0285 + $0.006 = $0.0345 per interaction
```

**Optimization:**
```python
# With caching (90% system prompt cached)
optimized_cost = (
    (500 * 0.00001 * 0.1) +      # 10% system prompt cost
    (2350 * 0.00001) +            # Other input tokens
    (200 * 0.00003)               # Output tokens
)
# = $0.0005 + $0.0235 + $0.006 = $0.03 per interaction (13% savings)
```

---

### 3.6 Model Drift Detection
**Definition:** Change in model performance over time

**Metrics:**
```python
# Weekly drift analysis
def detect_drift():
    current_week = get_interactions(days=7)
    baseline_week = get_interactions(days=7, weeks_ago=12)
    
    # Compare distributions
    drift_metrics = {
        "intent_distribution": compare_distributions(
            current_week.intents,
            baseline_week.intents
        ),
        "confidence_distribution": compare_distributions(
            current_week.confidences,
            baseline_week.confidences
        ),
        "csat_distribution": compare_distributions(
            current_week.csat,
            baseline_week.csat
        )
    }
    
    # Alert if significant drift
    for metric, drift_score in drift_metrics.items():
        if drift_score > 0.15:  # 15% drift threshold
            alert_team(f"Drift detected in {metric}: {drift_score}")
    
    return drift_metrics
```

**Your Experience:**
```
ADCB Mobile Banking:
- Monitoring: Weekly drift detection
- Alert: If CSAT drops >5% or intent distribution shifts >15%
- Action: Investigate, retrain if needed
- Example: Detected drift when new product launched (new intents)
```

---

## 4. Compliance & Risk KPIs (Legal/Risk Cares)

### 4.1 Audit Trail Completeness
**Definition:** % of interactions with complete audit logs

**Formula:** `(Interactions with full logs / Total interactions) × 100`

**Target:** 100% (non-negotiable)

**Required Fields:**
```python
audit_log = {
    "interaction_id": "uuid",
    "timestamp": "2025-11-16T22:34:58Z",
    "customer_id": "hashed",
    "channel": "voice",
    "intent": "account_inquiry",
    "retrieved_docs": [
        {"source": "policy-123", "version": "v2", "page": 5}
    ],
    "llm_request": {
        "model": "gpt-4-turbo",
        "prompt": "...",
        "temperature": 0.3
    },
    "llm_response": "...",
    "confidence": 0.92,
    "escalated": False,
    "csat": 5,
    "cost": 0.0345,
    "latency_ms": 2100
}
```

---

### 4.2 Bias & Fairness Metrics
**Definition:** Ensure AI treats all demographic groups fairly

**Metrics:**
```python
# Quarterly bias audit
def audit_bias():
    # Segment by demographics
    segments = ["age", "gender", "region", "language"]
    
    for segment in segments:
        segment_csat = calculate_csat_by_segment(segment)
        
        # Check variance
        max_csat = max(segment_csat.values())
        min_csat = min(segment_csat.values())
        variance = max_csat - min_csat
        
        if variance > 0.2:  # 0.2 point difference threshold
            alert_compliance(f"Bias detected in {segment}: {variance}")
            investigate_root_cause(segment)
```

**Your Experience:**
```
ADCB Mobile Banking:
- Quarterly bias audits
- Segments: Age, gender, region, language
- Finding: Business customers had 5% lower CSAT
- Root cause: Lack of business-specific training data
- Fix: Added 10K business customer interactions to training
- Result: CSAT equalized within 2 months
```

---

### 4.3 Escalation Accuracy
**Definition:** % of escalations that were correct decisions

**Formula:** `(Correct escalations / Total escalations) × 100`

**Targets:**
- Acceptable: 85-90%
- Good: 90-95%
- Excellent: 95-98%

**Measurement:**
```python
# Human review of escalations
def audit_escalations():
    escalations = db.query("""
        SELECT * FROM interactions
        WHERE escalated = TRUE
        ORDER BY RANDOM()
        LIMIT 100
    """)
    
    correct_escalations = 0
    for escalation in escalations:
        human_review = human_evaluator.review(escalation)
        if human_review == "correct_escalation":
            correct_escalations += 1
    
    accuracy = correct_escalations / 100
    return accuracy
```

---

## 5. Monitoring Dashboard (Real-Time)

### Power BI Dashboard Layout

**Page 1: Executive Summary**
```
┌─────────────────────────────────────────────────┐
│ Today's Metrics                                 │
├─────────────────────────────────────────────────┤
│ Containment: 73% ↑2%    CSAT: 4.3 ↑0.1        │
│ Cost/Contact: $0.82 ↓5%  Latency: 2.1s →      │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ Trend: Last 30 Days                             │
│ [Line chart: Containment, CSAT, Cost]          │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ Alerts                                          │
│ ⚠️  Hallucination rate: 2.1% (threshold: 2%)   │
│ ✅ All other metrics within target              │
└─────────────────────────────────────────────────┘
```

**Page 2: AI Quality**
```
┌─────────────────────────────────────────────────┐
│ Intent Recognition Accuracy: 93%                │
│ [Confusion matrix heatmap]                      │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ Retrieval Quality                               │
│ Precision: 88%  Recall: 82%  F1: 85%           │
│ [Distribution chart]                            │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ Latency Breakdown (P95)                         │
│ [Waterfall chart: STT, Intent, Retrieval, LLM] │
└─────────────────────────────────────────────────┘
```

**Page 3: Cost Analysis**
```
┌─────────────────────────────────────────────────┐
│ Monthly Cost: $253K                             │
│ [Pie chart: LLM, Infrastructure, Human agents] │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ Cost Per Interaction Trend                      │
│ [Line chart: Last 12 months]                    │
│ Target: $0.80  Current: $0.82  ↓15% YoY        │
└─────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Multi-Dimensional Success:** Track business, operational, AI quality, and compliance metrics
2. **Real-Time Monitoring:** Dashboards with alerts for immediate action
3. **Continuous Improvement:** Weekly analysis, monthly retraining, quarterly audits
4. **Stakeholder Alignment:** Different metrics for different audiences (CEO vs. engineer)
5. **Your Experience:** Proven track record of improving all key metrics at ADCB and JPMorgan

**Your Talking Point:**
*"At ADCB, we track 25+ KPIs across four dimensions. The key is real-time monitoring—if CSAT drops 2%, we investigate immediately. This proactive approach improved our CSAT from 3.8 to 4.3, containment from 45% to 73%, and reduced cost per interaction by 67%. Metrics drive decisions, not intuition."*
