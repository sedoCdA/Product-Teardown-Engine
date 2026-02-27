# Reflection — AI Product Teardown Engine

## Q1: Which of the 6 layers surprised you the most in terms of 
complexity for the product you chose? Why?

Layer 2 (Statistics & Analysis) surprised me the most. Going in, I 
assumed this would be the "boring" layer — just some A/B tests and 
dashboards. What I didn't anticipate was how fundamentally broken 
standard A/B testing is for a two-sided marketplace. The moment you 
realize that treatment and control users share the same pool of 
delivery partners and restaurants, you understand why Swiggy can't 
just run a standard experiment. The solution — Marketplace Switchback 
Testing — is a custom methodology that most ML engineers never 
encounter, and it only exists because the product's supply-demand 
coupling makes every standard statistical assumption invalid. This 
layer turned out to be where the most sophisticated statistical 
thinking actually lives in the product.

## Q2: What was the single biggest difference you noticed between 
the LLMs you tested? Not just "one was better" — what specifically 
did one do that the others couldn't?

Claude was the only LLM that identified the closed-loop feedback 
problem in ETA prediction — the insight that the model's own output 
causally changes the system it's predicting. ChatGPT and Gemini both 
described the ETA model accurately in terms of inputs and architecture, 
but neither identified the self-referential feedback loop that makes 
ETA prediction fundamentally harder than a standard regression problem. 
This distinction matters because it's exactly the kind of non-obvious 
systems-thinking insight that separates an engineer who understands 
ML from one who understands production ML. Gemini compensated by 
being the most creative (introducing GNNs and Model Distillation) 
but occasionally included techniques that were hard to verify as 
actually used by these companies — a different failure mode than 
being less insightful.
```

---

## Your GitHub Push Checklist

Create folder `week-01/day-05/teardown-engine/` and push these 8 files:

| File | Status |
|---|---|
| `system-prompt-v1.md` | ✅ From Step 2 earlier |
| `system-prompt-v2.md` | ✅ Above |
| `llm-comparison.md` | ✅ Previous message |
| `best-parts-map.md` | ✅ Above |
| `llm-selection.md` | ✅ Above |
| `final-teardown.md` | ✅ Above |
| `stress-test.md` | ⏳ Run V2 on IRCTC, paste output in |
| `reflection.md` | ✅ Above |

**Commit message:**
```
Day 5: AI Product Teardown Engine - Zomato/Swiggy - Claude
