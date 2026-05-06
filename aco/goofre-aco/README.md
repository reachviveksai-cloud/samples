# Goofre ACO - Agentic Commerce Orchestrator Sample

**Maintainer:** Goofre Open Source (github.com/Goofre-Agentic-Commerce-Orchestrator)
**UCP Conformance:** v1.2 (baseline 2026-01-23)
**License:** Apache 2.0 (samples only)

---

## The problem this sample solves

A merchant has Google Merchant Center, Google Analytics 4, Google Business Profile,
a Magento 2 catalog, and a POS system. All five tools have data about the same
products. None of them talk to each other. When a Gemini shopping agent queries
"do you have the Midnight Canvas Tote available for pickup in Brooklyn?", the answer
requires synthesizing inventory from the POS, availability from GBP, and checkout
eligibility from GMC - in real time, in a format the agent natively understands.

That synthesis is what an Agentic Commerce Orchestrator (ACO) does. This sample
shows the UCP data shapes that flow through one.

---

## What's in this sample

```
schema-demo/
  product.json     UCPProduct normalized from GMC via Merchant API v1
                   + com.goofre.agentic_trust_score extension
  cart.json        UCPCartEvent with identityLink (RFC #355) and AP2 idempotency
  order.json       UCPOrderEvent with idempotencyKey at top level (AP2 mandate)
  insight.json     UCPInsight (ucp_eligibility_blocked) + agentic_trust_score
  b2b-order.json   UCPOrderEvent + com.goofre.b2b_wholesale extension (MOQ, Net Terms)

mcp-integration/
  mcp_config.json  Drop-in config for Claude Desktop / GitHub Copilot
  README.md        Three-step MCP wiring guide

a2a-negotiation/
  intent-b2c.json  B2C CommerceIntent: pickup availability + maxPrice constraint
  intent-b2b.json  B2B Procurement Intent: MOQ + Net Terms + authorizedBudget
  README.md        A2A negotiation contract and response states explained
```

---

## Running it

```bash
npx create-goofre-ucp my-aco
cd my-aco
npm start

# Run the mock A2A negotiation
curl -X POST http://localhost:3000/api/a2a/intent \
  -H "Content-Type: application/json" \
  -d @samples/goofre-aco/a2a-negotiation/intent-b2c.json
```

---

## Vendor namespace extensions

This sample uses two `com.goofre.*` vendor namespace extensions per the
[UCP schema authoring spec](https://ucp.dev/documentation/schema-authoring/):

**`com.goofre.agentic_trust_score`** - AI discoverability health scoring.
UCP defines the transaction contract. This extension adds the operational
health layer: is this product actually discoverable by agents, or has a stale
GMC sync or missing GTIN made it effectively invisible?

**`com.goofre.b2b_wholesale`** - B2B procurement primitives.
MOQ, tiered contract pricing, payment terms (Net 30/60/90), and KYB status.
Both extensions are additive - a UCP-compliant agent that does not recognize
these namespaces can safely ignore them.

---

## UCP conformance

The fixtures in this sample pass the Goofre UCP Conformance Gate (26 tests,
baseline 2026-01-23), which mirrors the Universal-Commerce-Protocol/conformance
Python test suite.

Source: https://github.com/Goofre-Agentic-Commerce-Orchestrator/agentic_commerce_orchestrator_ACO
