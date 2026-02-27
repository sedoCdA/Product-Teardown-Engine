Layer 1: Data Foundation
What's happening:
Zomato/Swiggy ingest high-velocity, hyperlocal event streams — user clicks, search queries, order history, scroll depth, cart abandonment — alongside restaurant-side data: menu updates, inventory flags, prep time signals, and merchant ratings. This raw event data flows through a streaming pipeline into both a real-time serving store (for live recommendations) and a cold data lake (for batch model training). A critical preprocessing challenge is entity resolution — the same "Butter Chicken" appears across 50,000 restaurants with different names, spellings, and categorizations, requiring NLP-based dish normalization pipelines.
Key technologies likely used:
Apache Kafka for event streaming, Apache Flink or Spark Streaming for real-time processing, Delta Lake or Apache Hudi on S3/GCS for the data lake, Redis for low-latency feature serving, Hive/Presto for batch analytics, and Elasticsearch for menu and restaurant indexing.
The core engineering challenge:
Hyperlocal data freshness. A restaurant going offline, running out of a dish, or changing its ETA mid-dinner-rush must propagate through the pipeline in under 30 seconds — otherwise the recommendation engine serves unavailable options, causing order failures and user churn. At 500K+ restaurants across 500+ cities, maintaining sub-minute data freshness without overwhelming the serving layer is a genuinely hard distributed systems problem.
Skill required:
Data/ML Platform Engineer — skills: Apache Kafka, real-time stream processing (Flink/Spark), feature store design, data pipeline reliability at scale.
Honesty check:
This is the most critical and resource-heavy layer for Swiggy/Zomato. Without clean, fresh, hyperlocal data, every layer above it breaks. Enormous engineering investment here.

Layer 2: Statistics & Analysis
What's happening:
Swiggy and Zomato run continuous A/B and multivariate experiments — testing ranking algorithms, UI layouts, discount mechanisms, and delivery fee thresholds simultaneously across millions of users. They use Bayesian experimentation frameworks (not just classical frequentist t-tests) because they need to make faster shipping decisions without waiting for full statistical significance. They also rely heavily on survival analysis and time-series decomposition for understanding user lifecycle, churn prediction, and seasonal demand patterns (e.g., Sunday lunch spikes, Diwali order surges).
Key technologies likely used:
Custom experimentation platforms built on top of Spark, Statsmodels or PyMC for Bayesian analysis, Grafana + custom dashboards for metric tracking, Apache Airflow for orchestrating analytical pipelines, and Druid or ClickHouse for real-time OLAP queries on order metrics.
The core engineering challenge:
Network interference in experiments. When you discount a restaurant for User Group A, User Group B (the control) also sees changed delivery partner availability because supply is shared. Standard A/B testing assumes independent units — that assumption breaks completely in a two-sided marketplace, making clean causal inference extremely difficult.
Skill required:
Data Scientist / Experimentation Engineer — skills: causal inference, Bayesian A/B testing, marketplace experiment design, SQL + Python for analytical work.
Honesty check:
This layer is genuinely mature and heavily used at both companies. Swiggy in particular has published engineering blog posts on their experimentation infrastructure, suggesting deep investment here.

Layer 3: Machine Learning Models
What's happening:
Three distinct ML problems run in parallel: (1) Restaurant/dish ranking uses a multi-stage retrieval + ranking pipeline — a two-tower neural network retrieves the top-500 candidate restaurants from 50K+ options based on user embeddings and context embeddings, then a gradient boosted model (LightGBM/XGBoost) re-ranks the top-500 using 200+ features including real-time signals like current prep time, delivery partner density near that restaurant, and user's last 5 order categories. (2) Delivery time prediction (ETA) uses a gradient boosted regression model trained on historical order data, real-time traffic signals, restaurant prep time distributions, and delivery partner GPS telemetry — updated every 30 seconds per active order. (3) Surge/dynamic pricing uses a demand-supply imbalance model — likely a regression or lightweight neural network — that predicts delivery partner shortfall in a 1km radius over the next 15 minutes.
Key technologies likely used:
LightGBM and XGBoost for ranking and ETA, TensorFlow or PyTorch for the two-tower retrieval model, FAISS or ScaNN for approximate nearest-neighbor search in embedding space, MLflow or Weights & Biases for experiment tracking, and Feast or a custom feature store for serving precomputed user/restaurant embeddings.
The core engineering challenge:
The ETA model must balance accuracy with the fact that the prediction itself changes user behavior. If Swiggy shows "45 min ETA," some users cancel — removing a delivery partner from that area, which makes the actual ETA for the next user longer. The model's output feeds back into the system it's predicting, creating a closed-loop feedback problem that causes distributional shift if not carefully managed with offline/online evaluation separation.
Skill required:
ML Engineer — skills: two-tower retrieval architectures, gradient boosted trees for ranking, real-time feature engineering, model retraining pipelines, calibration of probabilistic models.
Honesty check:
This is the densest and most sophisticated ML layer. The restaurant ranking and ETA prediction models are almost certainly the most engineered parts of the entire product.

Layer 4: LLM / Generative AI
What's happening:
LLM usage at Zomato/Swiggy is real but relatively targeted compared to the ML layer. The most likely use cases are: (1) Review summarization — taking hundreds of user reviews per restaurant and generating a 2-line summary like "Loved for biryani, portions are large but delivery takes 40+ min." (2) Search query understanding — using a fine-tuned or prompted LLM to handle ambiguous queries like "something light for tonight" or "office lunch for 8 people under ₹2000", mapping them to structured filters. (3) Customer support chatbots — LLM-powered agents handling order status, refund requests, and complaint resolution, reducing human agent load.
Key technologies likely used:
Likely uses GPT-4 or Claude via API for review summarization (cost-effective at batch processing), a fine-tuned smaller model (Llama 3 or Mistral) deployed on-prem for latency-sensitive query understanding, and a RAG pipeline with LangChain or LlamaIndex for customer support, grounding responses in order data and policy documents.
The core engineering challenge:
Latency vs quality tradeoff in search. LLM-based query understanding must return results in under 200ms to not degrade search UX. A full GPT-4 call takes 1–3 seconds. This forces a two-path architecture: lightweight intent classifier handles 80% of queries instantly; LLM handles ambiguous edge cases asynchronously or with aggressive caching of common query patterns.
Honesty check:
This layer is real but NOT the core of Swiggy/Zomato's AI stack — it's additive. The recommmendation and ETA systems (Layer 3) are far more mission-critical. Anyone claiming LLMs are the "main AI" in a food delivery app is misunderstanding the product.

Layer 5: Deployment & Infrastructure
What's happening:
Models are served through a microservices architecture where the recommendation service, ETA service, and pricing service each have independent model endpoints. The ranking model runs synchronously in the critical path of every feed load — meaning it must respond in under 100ms for a good UX. This requires models to be pre-loaded in memory (not loaded per-request), with features precomputed and stored in Redis rather than computed on-the-fly. Continuous training pipelines retrain the ETA model every few hours on fresh order data; the ranking model likely retrains daily due to higher computational cost.
Key technologies likely used:
TensorFlow Serving or Triton Inference Server for model serving, Docker + Kubernetes for containerized deployment, Redis for feature caching, Apache Airflow for retraining pipeline orchestration, Prometheus + Grafana for model performance monitoring, and likely a custom shadow deployment setup for safely testing new model versions against live traffic.
The core engineering challenge:
Model-data skew during the dinner rush. At 8pm, order volume spikes 5–10x. The feature distributions at 8pm (high partner scarcity, many restaurants at capacity) look very different from the training data distribution (averaged across the full day). Models calibrated on average traffic can give systematically wrong ETAs during peak hours — exactly when accuracy matters most.
Skill required:
MLOps Engineer — skills: model serving infrastructure, Kubernetes, retraining pipeline design, monitoring for model drift, latency optimization at inference time.
Honesty check:
Heavy investment here. A food delivery platform lives and dies by model serving reliability — a 200ms latency spike in the recommendation API during dinner rush has a direct, measurable impact on conversion.

Layer 6: System Design & Scale
What's happening:
The platform must handle geographically distributed load — Bangalore's dinner rush peaks at 8pm IST, but that's a single timezone spike, so global CDN strategies are less relevant than city-level capacity planning. Swiggy/Zomato use a cell-based architecture where each city (or cluster of cities) runs semi-independently, preventing a Mumbai outage from cascading into Delhi. The recommendation system uses a pre-computation strategy for popular user segments — the top-50 restaurants for "user in Indiranagar, Bangalore who ordered 3 times last month" are pre-ranked every 30 minutes and served from cache, with real-time re-ranking only happening for fresh signals (e.g., user just searched for "pizza").
Key technologies likely used:
AWS or GCP multi-region deployment, Redis Cluster for distributed caching, consistent hashing for request routing, Apache Kafka with geo-partitioned topics, a custom real-time geospatial index (likely H3 or S2 geometry) for matching delivery partners to orders within a radius, and circuit breakers (Hystrix or Resilience4j) to degrade gracefully when downstream services fail.
The core engineering challenge:
Real-time geospatial matching at scale. At peak, Swiggy must simultaneously match 100K+ active orders to 300K+ delivery partners, factoring in partner location (updated every 10 seconds via GPS ping), partner current load, and predicted travel time — all within a 500ms window before the assignment times out. This is essentially a large-scale combinatorial optimization problem run in near-real-time, every second, across hundreds of cities.
Skill required:
Distributed Systems Engineer — skills: geospatial systems, real-time matching algorithms, event-driven architecture, designing for fault tolerance at scale.
Honesty check:
This layer is extremely mature and deeply engineered. The geospatial matching problem alone is a research-grade systems challenge.

Overall Analysis
Most critical layer: Layer 3 — Machine Learning Models. Specifically, the ETA prediction model. Users tolerate mediocre restaurant recommendations, but a wrong ETA (shown as 30 min, arrives in 55 min) is the #1 driver of negative reviews, refund requests, and churn. Every other layer exists to feed this model with good data and serve its output reliably.
Complexity rating: Bleeding Edge. This is not just "we trained a model" — it's a real-time, two-sided marketplace with closed-loop feedback, hyperlocal geospatial constraints, sub-100ms serving requirements, and models that must stay calibrated across wildly different demand environments (2pm Tuesday vs 8pm Friday). Very few companies in India operate ML infrastructure at this level of sophistication.
If rebuilding from scratch: "If you were rebuilding Swiggy/Zomato's AI stack from scratch, the very first thing you'd need to get right is the real-time feature pipeline — specifically, ensuring restaurant availability, prep time, and delivery partner density are fresh within 60 seconds across all locations — because every model above it (ranking, ETA, pricing) is only as accurate as the real-time features it consumes. Stale features make even a perfect model useless."

