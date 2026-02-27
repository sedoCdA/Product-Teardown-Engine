# Best-Parts Map — Zomato / Swiggy Teardown

| Layer | Best LLM | What to Extract |
|---|---|---|
| Layer 1: Data Foundation | Claude (LLM 1) | Dish normalization as NLP pipeline insight + entity resolution framing. "The same Butter Chicken appears across 50,000 restaurants" — this is a real, specific problem no other LLM named. |
| Layer 2: Statistics & Analysis | Gemini (LLM 3) | "Marketplace Switchback Testing" — entire city switches between algorithms every 30-60 min. This is the correct solution to A/B interference in two-sided marketplaces and the most technically accurate insight across all 3 outputs. |
| Layer 3: ML Models | Claude (LLM 1) | Closed-loop feedback problem: ETA prediction changes user behavior (cancellations), which changes delivery partner availability, which changes the actual ETA. This self-referential feedback loop causing distributional shift is the most non-obvious and correct insight in the entire comparison. |
| Layer 4: LLM / Generative AI | Claude (LLM 1) | Two-path latency architecture: lightweight intent classifier handles 80% of queries instantly, LLM handles ambiguous edge cases. This is a real production pattern and the most actionable engineering insight for this layer. |
| Layer 5: Deployment & Infrastructure | Gemini (LLM 3) | Model Distillation (Teacher/Student) tradeoff — large teacher model trains a smaller student model for sub-100ms serving. This is the correct answer to the speed vs accuracy tradeoff at inference time. |
| Layer 6: System Design & Scale | Claude (LLM 1) + Gemini (LLM 3) | Claude: H3/S2 geospatial indexing + cell-based city isolation + pre-computation for popular segments. Gemini: "Thundering Herd" problem framing at 8pm dinner rush. Combine both. |
| Overall Analysis / Hardest Problem | Claude (LLM 1) | ETA closed-loop feedback as the hardest problem. Most specific, most correct, most non-obvious. |
| Writing Style / Structure | ChatGPT (LLM 2) | Most consistent use of headers, sub-sections, and parallel structure across all 6 layers. Easiest to scan. |
