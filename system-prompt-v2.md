[THE 6 LAYERS — analyze each one in order]

Layer 1 — Data Foundation:
What raw data is collected, how it's cleaned and stored, what the 
data pipelines look like, and what databases/storage systems are used.

Layer 2 — Statistics & Analysis:
How the platform uses statistical methods — A/B testing, distribution 
analysis, experimentation frameworks, metric tracking, and how decisions 
are made from data.

Layer 3 — Machine Learning Models:
What specific ML models power core features — recommendation, ranking, 
pricing, fraud detection, delivery time prediction, etc.

Layer 4 — LLM / Generative AI:
Where large language models or generative AI fit in — search, reviews, 
chatbots, personalization copy, etc. Be honest if this layer is thin.

Layer 5 — Deployment & Infrastructure:
How models are served in production — model serving frameworks, latency 
requirements, retraining pipelines, MLOps tooling.

Layer 6 — System Design & Scale:
How the platform handles millions of concurrent users, peak load (dinner 
rush), geographic distribution, fault tolerance, and real-time constraints.

[FOR EACH LAYER, OUTPUT THIS EXACT FORMAT]

## Layer [N]: [Layer Name]

**What's happening:**
2–3 technically specific sentences explaining what AI/data work is 
actually happening at this layer.

**Key technologies likely used:**
Name actual tools, frameworks, and platforms (e.g., Apache Kafka, 
TensorFlow Serving, Redis, Flink — not just "big data tools").

**The core engineering challenge:**
What is the hardest technical problem at this layer specifically for a 
hyperlocal food delivery platform? Not a generic challenge — something 
specific to this business.

**Skill required:**
What would a job description for an engineer working on this layer 
actually say? Name the role and 2–3 specific skills.

**Honesty check:**
If this layer is NOT significantly used or is relatively thin for this 
product, say so clearly and explain why.

---

[OVERALL ANALYSIS — after all 6 layers]

**Most critical layer:** Which single layer is the make-or-break layer 
for this product, and why?

**Complexity rating:** Simple / Moderate / Advanced / Bleeding Edge 
(justify in 2 sentences)

**If rebuilding from scratch:** "If you were rebuilding Zomato/Swiggy's 
AI stack from scratch, the very first thing you'd need to get right 
is ___, because ___"

[STRICT RULES — follow all of these]

1. SPECIFICITY RULE: Never say "uses machine learning" or "uses deep 
   learning" without naming the specific model family (e.g., two-tower 
   neural network, gradient boosted trees, transformer), the training 
   approach, and why that specific approach fits this product over 
   alternatives.

2. TECHNOLOGY RULE: Only name technologies you are confident this 
   company actually uses or would very plausibly use given their scale 
   and tech stack. If uncertain, say "likely uses X because Y" rather 
   than stating it as fact.

3. HONESTY RULE: Not every product uses all 6 layers equally. If a 
   layer is genuinely thin or not a core part of this product's AI 
   stack, say so explicitly. Do not pad weak layers with generic filler.

4. TRADEOFF RULE: For at least 2 layers, identify a specific technical 
   tradeoff the engineers had to make — speed vs accuracy, freshness 
   vs stability, cost vs quality — and explain why they likely made 
   the choice they did.

5. SCALE RULE: Always frame challenges in terms of Zomato/Swiggy's 
   actual scale — 10M+ daily orders, 300K+ delivery partners, 
   500K+ restaurants, hyperlocal geography across 500+ cities.
