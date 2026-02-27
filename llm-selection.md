# LLM Selection Decision — Zomato / Swiggy Teardown

| Decision Factor | My Choice | Reason |
|---|---|---|
| Which LLM followed prompt structure most faithfully? | ChatGPT (LLM 2) | Maintained consistent headers and parallel structure across all 6 layers without deviation |
| Which LLM was most technically accurate (least hallucination)? | ChatGPT (LLM 2) | Claude was equally accurate; Gemini introduced GNN for flavor graphs and MCP integration — creative but harder to verify |
| Which LLM's output was most readable and well-organized? | ChatGPT (LLM 2) | Most consistent sub-section use and scannable formatting throughout |
| Which LLM handled the "honesty check" best? | Claude (LLM 1) | Explicitly stated Layer 4 is NOT the core of the product and explained why — most direct honesty signal |
| Which LLM produced the single best technical insight? | Claude (LLM 1) | Closed-loop ETA feedback problem is the most non-obvious, correct, and interview-worthy insight |

**Selected LLM for final tool:** Claude (claude.ai)

**Why:** Claude consistently produced the deepest non-obvious insights 
(closed-loop ETA feedback, dish normalization pipeline, two-path LLM 
architecture) and was the most honest about layer weight differences. 
ChatGPT was more structurally consistent, but for a teardown tool where 
insight depth matters more than formatting polish, Claude is the 
stronger engine — especially when the V2 prompt enforces structure 
explicitly, removing Claude's only comparative weakness.
