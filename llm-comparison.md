# LLM Comparison — Product Teardown: Zomato / Swiggy

## Models Used
| # | LLM Name | Mode | Response Time (approx) |
|---|----------|------|----------------------|
| 1 | Claude (claude.ai) | Standard | ~45 sec |
| 2 | ChatGPT (chatgpt.com) | Standard/o1 | ~60 sec |
| 3 | Gemini (gemini.google.com) | Pro/Flash | ~30 sec |

---

## Layer-by-Layer Comparison

### Layer 1: Data Foundation
| Criteria | LLM 1 (Claude) | LLM 2 (ChatGPT) | LLM 3 (Gemini) | Best? |
|---|---|---|---|---|
| Specificity (1-5) | 5 | 4 | 4 | LLM 1 |
| Named real tech? | Y | Y | Y | Tie |
| Identified a real engineering challenge? | Y | Y | Y | Tie |
| Notes | Named dish normalization as NLP problem — unique insight | Covered schema enforcement + GPS drift repair | Named Geohash partitioning — good spatial detail | LLM 1 for depth |

### Layer 2: Statistics & Analysis
| Criteria | LLM 1 (Claude) | LLM 2 (ChatGPT) | LLM 3 (Gemini) | Best? |
|---|---|---|---|---|
| Specificity (1-5) | 4 | 5 | 5 | LLM 2 & 3 |
| Named real tech? | Y | Y | Y | Tie |
| Identified a real engineering challenge? | Y | Y | Y | LLM 2 & 3 |
| Notes | Good on Bayesian + churn analysis | Named counterfactual + uplift modeling — most complete | Named "Switchback Testing" specifically — best single insight | LLM 3 for naming Switchback Testing |

### Layer 3: Machine Learning Models
| Criteria | LLM 1 (Claude) | LLM 2 (ChatGPT) | LLM 3 (Gemini) | Best? |
|---|---|---|---|---|
| Specificity (1-5) | 5 | 5 | 4 | LLM 1 & 2 |
| Named real tech? | Y | Y | Y | Tie |
| Named model family? | Y | Y | Y | Tie |
| Identified a real engineering challenge? | Y | Y | Y | Tie |
| Notes | Best on feedback loop problem (ETA changes behavior) | Added Temporal Fusion Transformer + OR-Tools for dispatch | Named GNN for flavor graphs — creative but less verified | LLM 1 for closed-loop feedback insight |

### Layer 4: LLM / Generative AI
| Criteria | LLM 1 (Claude) | LLM 2 (ChatGPT) | LLM 3 (Gemini) | Best? |
|---|---|---|---|---|
| Specificity (1-5) | 5 | 4 | 4 | LLM 1 |
| Honest if not applicable? | Y | Y | Y | All good |
| Notes | Best 2-path latency solution (classifier + LLM fallback) | Good RAG + grounding challenge | Named MCP protocol — specific but hard to verify | LLM 1 for architecture solution |

### Layer 5: Deployment & Infrastructure
| Criteria | LLM 1 (Claude) | LLM 2 (ChatGPT) | LLM 3 (Gemini) | Best? |
|---|---|---|---|---|
| Specificity (1-5) | 4 | 5 | 5 | LLM 2 & 3 |
| Named real tech? | Y | Y | Y | Tie |
| Notes | Good on dinner rush skew problem | Named Feast + Envoy + shadow testing | Named Model Distillation (Teacher/Student) — best insight here | LLM 3 for distillation tradeoff |

### Layer 6: System Design & Scale
| Criteria | LLM 1 (Claude) | LLM 2 (ChatGPT) | LLM 3 (Gemini) | Best? |
|---|---|---|---|---|
| Specificity (1-5) | 5 | 4 | 4 | LLM 1 |
| Named real tech? | Y | Y | Y | Tie |
| Notes | Named H3/S2 geometry + circuit breakers + pre-computation strategy | Named autoscaling + consistent hashing | Named Thundering Herd problem + pre-ranking at 8pm — great real-world framing | LLM 1 & 3 tie |

---

## Overall Verdict

| Dimension | Winner | Why? |
|---|---|---|
| Most technically specific overall | LLM 1 (Claude) | Named specific tradeoffs per layer with quantified constraints (100ms, 200ms) |
| Best at naming real technologies | LLM 2 (ChatGPT) | Named OR-Tools, TFT, Feast, Envoy consistently |
| Least hallucination / made-up info | LLM 2 (ChatGPT) | Gemini's GNN + MCP claims are plausible but harder to verify |
| Best at "hardest problem" insight | LLM 1 (Claude) | ETA closed-loop feedback problem is the most non-obvious and correct insight |
| Best structured output | LLM 2 (ChatGPT) | Most consistent formatting across all 6 layers |
| Fastest useful response | LLM 3 (Gemini) | Noticeably faster, still high quality |

---

## Key Observation
> All 3 LLMs agreed on the core stack (Kafka, LightGBM, two-tower 
> networks) but diverged sharply in depth of *insight*. Claude 
> identified the closed-loop feedback problem in ETA modeling — a 
> genuinely non-obvious systems issue. ChatGPT was the most 
> consistently structured and technology-specific across all layers. 
> Gemini introduced the most creative ideas (GNNs, MCP, Model 
> Distillation, Switchback Testing) but also had the highest risk 
> of including things that sound right but are hard to verify.
