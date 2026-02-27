# Stress Test — IRCTC Train Booking System

## Did the tool generalize correctly?

**Test product:** IRCTC (Indian Railway Catering and Tourism Corporation) 
train booking and ticketing platform

**Key question:** Does the teardown correctly produce a THINNER output 
at layers that don't apply — or does it pad every layer equally?

## What a correct teardown should show:

| Layer | Expected Weight | Why |
|---|---|---|
| Layer 1: Data Foundation | HEAVY | Massive transaction logs, seat inventory, real-time train position data |
| Layer 2: Statistics & Analysis | MODERATE | Demand forecasting for routes, quota management analysis |
| Layer 3: ML Models | MODERATE | Demand prediction for Tatkal pricing, waitlist prediction, maybe fraud detection |
| Layer 4: LLM / Gen AI | THIN | Minimal LLM usage — core product is transactional, not conversational |
| Layer 5: Deployment | HEAVY | Must survive peak booking windows (10am Tatkal rush = 100K concurrent users) |
| Layer 6: System Design | HEAVY | One of India's highest-traffic government platforms — concurrency and queue management is the core problem |

## Pass/Fail Criteria:
- PASS: Layer 4 honesty check says "LLM usage is minimal — this is 
  a transactional booking system, not a recommendation engine"
- PASS: Layer 6 highlights the 10am Tatkal booking spike as the 
  primary system design challenge
- FAIL: Every layer gets the same depth as the Zomato teardown
- FAIL: Layer 4 invents LLM use cases that don't exist for IRCTC

## Paste your actual V2 output below:
[INSERT OUTPUT FROM YOUR V2 PROMPT ON IRCTC HERE]

## Verdict:
[ ] Tool generalizes correctly — depth varies by layer as expected
[ ] Tool over-fitted — all layers equally dense regardless of product
