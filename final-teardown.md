# AI Product Teardown: Zomato / Swiggy
**Produced by:** V2 System Prompt + Claude
**Date:** 2026

---

## Layer 1: Data Foundation

**What's happening:**
The platform ingests high-velocity event streams — user clicks, search 
queries, scroll depth, cart abandonment, order history — alongside 
restaurant-side signals: menu updates, prep time changes, inventory 
flags, and merchant status (open/busy/closed). GPS telemetry from 
300K+ delivery partners arrives every 5–10 seconds, requiring 
sub-minute freshness to keep ETA predictions accurate. A critical 
and underappreciated preprocessing challenge is entity resolution: 
the same "Butter Chicken" appears across 500K+ restaurants with 
different spellings, categorizations, and dish descriptions — requiring 
NLP-based menu normalization pipelines to create a consistent dish 
taxonomy for ranking and search.

**Key technologies likely used:**
Apache Kafka (event ingestion), Apache Flink or Spark Structured 
Streaming (real-time processing), Apache Iceberg or Delta Lake on S3 
(data lake with update support), Redis or Aerospike (low-latency 
feature serving for current partner/restaurant state), 
Feast or custom feature store (offline/online feature consistency), 
H3 or S2 geospatial indexing (spatial partitioning of location data), 
Elasticsearch (menu and restaurant text search).

**The core engineering challenge:**
Data freshness at hyperlocal granularity. A restaurant going offline, 
running out of a dish, or changing its prep time mid-dinner-rush must 
propagate through the pipeline in under 60 seconds — otherwise the 
recommendation engine surfaces unavailable options, causing order 
failures and refund costs. At 500K+ restaurants across 500+ cities, 
maintaining sub-minute freshness without overwhelming the serving 
layer is a hard distributed systems problem with direct revenue impact.

**Skill required:**
Data Platform Engineer (Real-time Systems) — Apache Kafka, Flink stream 
processing, geospatial data pipelines, feature store design, 
event-time semantics.

**Honesty check:**
This is the most foundational and resource-heavy layer in the stack. 
Every model above it is only as good as the freshness and accuracy 
of the data flowing through here. Enormous engineering investment.

**Tradeoff:**
Freshness vs infrastructure cost. Processing millions of GPS pings 
per minute to keep ETAs current is expensive. If data is 30 seconds 
stale, the delivery partner has moved 200–400 meters, degrading ETA 
accuracy for every active order downstream.

---

## Layer 2: Statistics & Analysis

**What's happening:**
Swiggy and Zomato run continuous experimentation but face a fundamental 
problem with standard A/B testing: treatment and control users share 
the same pool of delivery partners and restaurants. Giving User Group A 
a discounted restaurant pulls delivery partners toward that area, 
degrading service for User Group B — a direct violation of SUTVA 
(Stable Unit Treatment Value Assumption). To solve this, they use 
Marketplace Switchback Testing: the entire city switches between 
algorithm variants on a 30–60 minute schedule, ensuring no two 
variants compete for the same supply simultaneously. They also use 
Bayesian sequential testing and uplift modeling to measure long-term 
retention effects vs short-term conversion lifts from ranking changes.

**Key technologies likely used:**
Custom experimentation platform (similar to Uber's XP framework), 
Bayesian sequential testing libraries, PyMC or SciPy/StatsModels for 
distribution modeling (log-normal distributions for prep/travel time), 
Apache Airflow (metric computation pipelines), ClickHouse or Druid 
(real-time OLAP on experiment metrics), Grafana dashboards.

**The core engineering challenge:**
Causal interference in a two-sided marketplace. Standard A/B tools 
(Optimizely, etc.) assume independent units — an assumption that breaks 
completely when supply is shared across treatment and control. Building 
a correct experimentation system for a marketplace requires custom 
infrastructure and methodology most companies never need to develop.

**Skill required:**
Experimentation Scientist / Applied Scientist — causal inference, 
Bayesian A/B testing, marketplace interference modeling, 
counterfactual evaluation.

**Honesty check:**
Extremely mature and heavily used. Running a food delivery platform 
without rigorous experimentation would mean flying blind on ranking, 
dispatch, and pricing decisions that directly affect revenue and 
unit economics at 10M+ daily orders.

**Tradeoff:**
Experiment velocity vs statistical validity. Switchback testing reduces 
interference but introduces temporal autocorrelation — dinner rush 
in the treatment period isn't identical to dinner rush in the control 
period. Cycle length must be tuned carefully: too short and the system 
hasn't stabilized; too long and experiment throughput collapses.

---

## Layer 3: Machine Learning Models

**What's happening:**
Three distinct ML systems run in parallel. (1) **Restaurant ranking** 
uses a multi-stage pipeline: a two-tower neural network retrieves the 
top-500 candidate restaurants from 500K+ options by matching user 
embeddings against restaurant embeddings (trained on implicit click/order 
feedback with sampled softmax), then LightGBM re-ranks the top-500 using 
200+ features including real-time signals: current prep time, delivery 
partner density near the restaurant, user's last 5 order categories, 
predicted ETA. (2) **Delivery time prediction (ETA)** uses gradient 
boosted regression (LightGBM/XGBoost) or a Temporal Fusion Transformer 
trained on rider GPS traces, historical traffic, restaurant prep 
distributions, weather, and supply-demand imbalance — updated every 
30 seconds per active order. (3) **Dispatch optimization** assigns 
delivery partners to orders using combinatorial optimization 
(OR-Tools or custom heuristics) balancing travel distance, current 
partner load, and predicted kitchen ready time.

**Key technologies likely used:**
TensorFlow or PyTorch (two-tower training), LightGBM / XGBoost 
(ranking and ETA regression), FAISS or ScaNN (approximate nearest-neighbor 
search in embedding space — faster than exact search at the cost of ~5% 
recall), OR-Tools (dispatch optimization), MLflow or W&B 
(experiment tracking), Feast (feature serving consistency).

**The core engineering challenge:**
The ETA model has a closed-loop feedback problem. Showing a user 
"45 min ETA" causes some users to cancel — removing a delivery partner 
from that area — which changes the actual ETA for the next user. 
The model's output causally affects the system it is predicting. 
Models trained on historical data don't account for this, causing 
systematic distributional shift during high-cancellation periods 
(bad weather, late-night orders) that is invisible in offline evaluation.

**Skill required:**
ML Engineer (Ranking / Logistics) — two-tower retrieval architectures, 
gradient boosted trees, spatiotemporal modeling, offline-to-online 
feature consistency, feedback loop detection.

**Honesty check:**
This is the densest and most mission-critical ML layer. The ranking 
and ETA models are the product — everything else supports them.

**Tradeoff:**
Model accuracy vs inference latency. Temporal Fusion Transformers 
improve ETA accuracy but add inference latency. Many production systems 
favor LightGBM for predictable millisecond-level inference, accepting 
a small accuracy tradeoff for a latency guarantee that can't be 
violated during dinner rush.

---

## Layer 4: LLM / Generative AI

**What's happening:**
LLM usage is real but deliberately scoped to tasks where latency 
constraints are manageable. Three main applications: (1) **Review 
summarization** — batch-processing hundreds of reviews per restaurant 
into 2-3 line summaries run offline, not in the serving path. 
(2) **Semantic search query understanding** — mapping ambiguous queries 
like "something light for the office" or "spicy ramen under ₹500" 
to structured filters (cuisine type, price band, dietary flags). 
(3) **Customer support automation** — LLM-powered agents handling 
order status, refunds, and complaints via RAG over order history 
and policy documents.

**Key technologies likely used:**
GPT-4o or Claude via API (batch review summarization — cost-effective 
offline), fine-tuned Llama 3 or Mistral self-hosted (high-volume 
query understanding — lower latency and token cost), 
Pinecone or Elasticsearch dense retrieval (RAG for support chat), 
LangChain or LlamaIndex (orchestration).

**The core engineering challenge:**
Latency vs quality in search. Full LLM inference takes 1–3 seconds; 
acceptable search latency is under 200ms. This forces a two-path 
architecture: a lightweight intent classifier (fine-tuned BERT or 
DistilBERT) handles 80%+ of queries instantly and maps them to 
structured filters; the LLM handles ambiguous edge cases either 
asynchronously or via aggressive caching of frequent query patterns.

**Skill required:**
Applied LLM Engineer — RAG pipeline design, prompt engineering with 
structured grounding, LLM latency optimization, hallucination 
evaluation.

**Honesty check:**
This layer enhances UX but is NOT the core of the product. The LLM 
does not decide which restaurant ranks first — it interprets user 
intent and passes structured queries to the Layer 3 ranking engine. 
Anyone claiming LLMs are the "main AI" in a food delivery platform 
is misreading the architecture.

**Tradeoff:**
API (GPT-4o) vs self-hosted (Llama 3). API models are higher quality 
but cost more per query and add external latency dependency. 
Self-hosted open-source models have lower per-query cost and full 
latency control but require MLOps investment to maintain. 
At Swiggy's query volume, self-hosted wins on economics for 
high-frequency tasks.

---

## Layer 5: Deployment & Infrastructure

**What's happening:**
The ranking, ETA, and pricing services each run as independent 
microservice endpoints with strict latency budgets: ranking must 
respond in under 100ms in the critical serving path, retrieval in 
under 20ms. Models are pre-loaded into memory (never loaded per-request) 
with features precomputed in Redis rather than computed on-the-fly. 
A key technique is model distillation: a large, high-accuracy "teacher" 
model trains a smaller, faster "student" model for production serving — 
accepting a small accuracy drop for a latency guarantee. 
Retraining pipelines run on different schedules: ETA model hourly 
(high drift sensitivity), ranking model daily (higher compute cost).

**Key technologies likely used:**
TensorFlow Serving or TorchServe / BentoML (model serving), 
Kubernetes with autoscaling (handling dinner rush spikes), 
Feast or Tecton (feature store for offline/online parity), 
MLflow (model registry and versioning), Apache Airflow (retraining 
orchestration), Prometheus + Grafana (model performance and drift 
monitoring), Envoy or Istio (service mesh and traffic management), 
shadow deployment setup (test new models against live traffic 
without affecting users).

**The core engineering challenge:**
Feature distribution skew during peak hours. The feature distributions 
at 8pm (high partner scarcity, many restaurants at capacity, surge demand) 
look fundamentally different from the training data distribution 
(averaged across all hours). A model calibrated on average traffic 
gives systematically wrong ETAs during the 2-hour window when 
accuracy matters most — and when a bad prediction is most costly.

**Skill required:**
MLOps Engineer — Kubernetes, model serving infrastructure, 
feature store design, retraining pipeline orchestration, 
production model monitoring and drift detection.

**Honesty check:**
This layer is deeply engineered and operationally critical. A food 
delivery platform lives and dies by model serving reliability — 
a 200ms latency spike during dinner rush has a direct, measurable 
impact on conversion and user retention.

**Tradeoff:**
Feature freshness vs system stability. More real-time features 
improve model accuracy but increase the failure surface area — each 
additional real-time feature is another service call that can time out 
under load. Engineers must decide which features are worth the 
reliability risk and which can be served from 5-minute-old cache.

---

## Layer 6: System Design & Scale

**What's happening:**
The system uses a cell-based (regional sharding) architecture: each 
city or cluster of cities operates semi-independently, preventing a 
Mumbai infrastructure failure from cascading into Delhi. Since a user 
in Bangalore never orders from a restaurant in Delhi, recommendations 
and dispatch are isolated by city cluster. To handle the 
"Thundering Herd" problem — millions of users opening the app 
simultaneously at 8pm dinner rush — the system pre-computes and 
caches restaurant rankings for active user segments every 30 minutes, 
so most feed loads serve from cache rather than triggering live 
model inference. The real-time geospatial matching engine continuously 
assigns delivery partners to incoming orders using an approximate 
nearest-neighbor index updated every 10 seconds from GPS telemetry.

**Key technologies likely used:**
Multi-region AWS or GCP deployment, Redis Cluster (distributed 
recommendation caching), consistent hashing (geographic request 
routing), Apache Kafka with geo-partitioned topics, H3 or S2 geometry 
library (geospatial indexing for partner-order matching), 
Envoy + Istio (service mesh), circuit breakers via Resilience4j 
(graceful degradation when downstream services fail), 
CockroachDB or TiDB (distributed SQL for transactional data at scale).

**The core engineering challenge:**
Real-time geospatial matching at scale. At peak, Swiggy must 
simultaneously match 100K+ active orders to 300K+ delivery partners — 
each with a continuously updating GPS position — factoring in partner 
load, predicted travel time, and restaurant ready time, all within 
a 500ms assignment window before the request times out. This is a 
large-scale combinatorial optimization problem that must run every 
second, continuously, across hundreds of cities.

**Skill required:**
Distributed Systems Architect — geospatial systems, real-time matching 
algorithms, event-driven architecture, capacity planning, 
designing for fault tolerance at hyperlocal delivery scale.

**Honesty check:**
This layer is extremely mature and deeply engineered. The geospatial 
matching problem alone is a research-grade systems challenge. 
There is no off-the-shelf solution — this is custom infrastructure.

**Tradeoff:**
Optimization quality vs assignment speed. A globally optimal partner 
assignment algorithm would take seconds to compute. In production, 
they use fast approximate algorithms (greedy heuristics + OR-Tools) 
that find a good-enough assignment in milliseconds — because a delayed 
assignment costs more than a slightly suboptimal one.

---

## Overall Analysis

**Most critical layer:**
**Layer 3 — Machine Learning Models.** Specifically, the ETA prediction 
model. Users tolerate mediocre restaurant recommendations — but a 
wrong ETA (shown as 30 min, arrives in 55 min) is the #1 driver of 
negative reviews, refund requests, and long-term churn. Every other 
layer exists to feed this model with accurate data and serve its 
output reliably within
