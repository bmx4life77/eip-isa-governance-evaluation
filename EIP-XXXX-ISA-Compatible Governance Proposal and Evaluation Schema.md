---
eip: XXXX
title: ISA-Compatible Governance Proposal & Evaluation Schema
description: Standardized JSON schemas for off-chain governance proposal evaluation and PASS/REVIEW/FAIL decisioning.
author: Louis (@yourGithubHandle)
discussions-to: https://ethereum-magicians.org/t/eip-xxxx-isa-governance-evaluation-schema/XXXXX
status: Draft
type: Standards Track
category: ERC
created: 2026-01-12
requires: ERC-4824
license: CC-BY-4.0
---

## **Abstract**

This EIP defines a standardized, machine‑readable schema for **off‑chain governance proposals** and **quantitative evaluation reports** used in pre-on-chain governance pipelines. The standard enables protocols to perform structured, metrics‑driven risk assessment *before* submitting proposals to on‑chain governance systems such as OpenZeppelin Governor. It introduces:

- A canonical **proposal JSON schema**  
- A canonical **evaluation JSON schema**  
- A tri‑state decision model (**PASS / REVIEW / FAIL**)  
- A domain‑aware metrics interpretation layer  
- A consistent interface for CI/CD gating, dashboards, and governance tooling  

This EIP does **not** replace on‑chain governance. It formalizes the **pre‑flight evaluation layer** that helps prevent dangerous or poorly timed proposals from reaching binding execution.

---

## **Motivation**

On‑chain governance systems (e.g., OpenZeppelin Governor) provide secure, decentralized execution of community decisions. However, they lack a standardized mechanism for **pre‑submission risk evaluation**. As a result, protocols frequently encounter:

- High‑risk proposals submitted during volatile market conditions  
- Poorly justified parameter changes  
- Over‑optimistic claims without quantitative backing  
- Governance spam or low‑quality upgrades  
- Lack of structured metadata for automated review  

This EIP formalizes schemas and semantics for interoperability across:

- Governance dashboards  
- CI/CD pipelines  
- Risk engines  
- DAO tooling  
- Off‑chain simulation frameworks  
- On‑chain proposal gating systems  

### **Relationship to Existing Standards**

| Layer | Tool | Role |
|-------|------|------|
| **Off‑chain evaluation** | This EIP (e.g., via ISA Fabric or compatible tools) | Risk scoring, forecasting, pre‑flight checks |
| **On‑chain execution** | OpenZeppelin Governor | Token‑weighted voting, timelock, execution |

These layers are **complementary**, not competitive.  
This EIP acts as the **diagnostic system**; existing governance contracts serve as the **execution engine**.

---

## **Specification**

This section defines the canonical schemas for:

1. **Governance Proposal JSON**  
2. **Governance Evaluation JSON**  
3. **Status codes and semantics**  
4. **Metric interpretation rules**  

### **1. Governance Proposal Schema**

Proposals MUST conform to the following JSON Schema (Draft-07):

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "title", "description", "impacts", "threshold"],
  "properties": {
    "id": {
      "type": "string",
      "description": "Unique proposal identifier (e.g., PROP-YYYY-NNN)",
      "pattern": "^(PROP|UPG|GOV)-[0-9]{4}-[0-9]{3}$"
    },
    "title": {
      "type": "string",
      "description": "Human-readable title",
      "minLength": 8,
      "maxLength": 120
    },
    "description": {
      "type": "string",
      "description": "Rationale, risks, design details",
      "minLength": 40
    },
    "impacts": {
      "type": "array",
      "description": "Expected metric changes",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["metric", "direction", "magnitude"],
        "properties": {
          "metric": {
            "type": "string",
            "enum": ["beta", "vu", "VU", "iota", "phi", "psi5", "SE", "drift_1h", "volatility_24h"]
          },
          "direction": {
            "type": "string",
            "enum": ["increase", "decrease", "neutral"]
          },
          "magnitude": {
            "type": "number",
            "description": "Relative change (0.0–1.0)",
            "minimum": 0.0,
            "maximum": 1.0
          },
          "confidence": {
            "type": "number",
            "description": "Evidence-based confidence (0.0–1.0)",
            "minimum": 0.0,
            "maximum": 1.0,
            "default": 0.75
          }
        }
      }
    },
    "threshold": {
      "type": "number",
      "description": "Minimum governance score required",
      "minimum": 0.5,
      "maximum": 0.98
    },
    "domain": {
      "type": "string",
      "description": "Optional override (e.g., DEX, LENDING, PERP)",
      "enum": ["DEX", "LENDING", "PERP", "BRIDGE", "YIELD", "NFT", "DAO"]
    },
    "metadata": {
      "type": "object",
      "description": "Arbitrary additional info (e.g., author, audit links)",
      "additionalProperties": true,
      "properties": {
        "version": {
          "type": "string",
          "description": "Schema version for backwards compatibility (e.g., 1.0)",
          "default": "1.0"
        }
      }
    }
  },
  "additionalProperties": false
}
```

**Example Proposal**:
```json
{
  "id": "PROP-2026-042",
  "title": "Batch Processing & Gas Optimization v4.2",
  "description": "Introduce transaction batching at the sequencer level + proxy contract upgrades to reduce average gas per swap by ~18%. Includes new circuit breakers for batch size. Expected to lower volatility during high activity periods while slightly increasing short-term drift during rollout.",
  "impacts": [
    {
      "metric": "beta",
      "direction": "decrease",
      "magnitude": 0.094,
      "confidence": 0.89
    },
    {
      "metric": "volatility_24h",
      "direction": "decrease",
      "magnitude": 0.132,
      "confidence": 0.82
    }
  ],
  "threshold": 0.76,
  "domain": "DEX",
  "metadata": {
    "version": "1.0",
    "author": "core-eng-team",
    "audit": "https://audits.example.com/2026/batch-v4.2",
    "forum": "https://gov.example.com/t/prop-042-batch-optim-v4.2"
  }
}
```

### **2. Governance Evaluation Schema**

Evaluations MUST conform to the following JSON Schema (Draft-07):

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["runId", "timestamp", "domain", "proposal", "status", "governanceScore"],
  "properties": {
    "runId": {
      "type": "string",
      "description": "Unique evaluation identifier"
    },
    "timestamp": {
      "type": "string",
      "description": "ISO 8601 timestamp",
      "format": "date-time"
    },
    "domain": {
      "type": "string",
      "enum": ["DEX", "LENDING", "PERP", "BRIDGE", "YIELD", "NFT", "DAO"]
    },
    "proposal": {
      "type": "object",
      "required": ["id", "title", "threshold"],
      "properties": {
        "id": { "type": "string" },
        "title": { "type": "string" },
        "threshold": { "type": "number" }
      }
    },
    "status": {
      "type": "string",
      "enum": ["PASS", "REVIEW", "FAIL"]
    },
    "governanceScore": {
      "type": "number",
      "description": "Weighted score (0.0–1.0)",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "currentModulatedMetrics": {
      "type": "object",
      "description": "Current modulated metric values",
      "additionalProperties": { "type": "number" }
    },
    "impactAnalysis": {
      "type": "array",
      "description": "Per-metric forecast and quality assessment",
      "items": {
        "type": "object",
        "required": ["metric", "current", "declaredDirection", "declaredMagnitude", "forecasted", "quality", "flag"],
        "properties": {
          "metric": { "type": "string" },
          "current": { "type": "number" },
          "declaredDirection": { "type": "string" },
          "declaredMagnitude": { "type": "number" },
          "forecasted": { "type": "number" },
          "quality": { "type": "string" },
          "flag": { "type": "string" }
        }
      }
    },
    "criticalFlags": {
      "type": "array",
      "items": { "type": "string" }
    },
    "warnings": {
      "type": "array",
      "items": { "type": "string" }
    },
    "recommendation": {
      "type": "string"
    },
    "summary": {
      "type": "string"
    },
    "suggestedNextSteps": {
      "type": "array",
      "items": { "type": "string" }
    },
    "metadata": {
      "type": "object",
      "additionalProperties": true,
      "properties": {
        "version": {
          "type": "string",
          "description": "Schema version for backwards compatibility (e.g., 1.0)",
          "default": "1.0"
        }
      }
    }
  },
  "additionalProperties": false
}
```

**Example Evaluation** (excerpt for FAIL case):
```json
{
  "runId": "gov-eval_20260112_184522_x9f2d1",
  "timestamp": "2026-01-12T18:45:22.317Z",
  "domain": "DEX",
  "proposal": {
    "id": "PROP-2026-089",
    "title": "Risky Gas Upgrade v5",
    "threshold": 0.78
  },
  "status": "FAIL",
  "governanceScore": 0.612,
  "currentModulatedMetrics": {
    "beta_modulated": 2.62,
    "volatility_24h": 0.38,
    "SE": 0.032,
    "drift_1h": 0.092,
    "phi": 0.61,
    "psi5": 0.48
  },
  "impactAnalysis": [
    {
      "metric": "beta_modulated",
      "current": 2.62,
      "declaredDirection": "neutral",
      "declaredMagnitude": 0.0,
      "forecasted": 2.62,
      "quality": "none",
      "flag": "CRITICAL"
    }
  ],
  "criticalFlags": ["high_beta_exposure"],
  "warnings": ["low_phi_efficiency"],
  "recommendation": "REJECT / MAJOR REVISION REQUIRED",
  "summary": "Current baseline already in danger zone...",
  "suggestedNextSteps": ["Wait for calmer macro conditions"],
  "metadata": {
    "version": "1.0"
  }
}
```

### **3. Status Codes and Semantics**

| Status | Meaning | Conditions |
|--------|---------|------------|
| **PASS** | Safe to proceed to on-chain submission | Score ≥ threshold AND no critical flags |
| **REVIEW** | Borderline; requires manual/human review | Score ≥ threshold BUT warnings or minor risks present |
| **FAIL** | Do not proceed; high risk | Score < threshold OR any critical flag |

### **4. Metric Interpretation Rules**

Metrics provide a multi‑dimensional risk fingerprint. Implementations SHOULD use domain-specific thresholds and weights. Extensions for custom metrics MAY be added via `metadata.customMetrics`.

| Metric | Symbol | Description | Typical Range | Interpretation (Higher Means...) | Reference/Formula Note |
|--------|--------|-------------|---------------|--------------------------|-------------------------|
| Beta | β | Systemic risk exposure | 0.8–3.5 | More correlated risk | Covariance-based; see ISA Fabric docs for formula |
| vu | vu | Volume utilization (short-term) | 0.1–1.2 | More efficient usage | Normalized volume per user |
| VU | VU | Aggregated volume utilization | 0.3–1.5 | Stronger signal | Weighted variant of vu |
| Iota | ι | Innovation/novelty signal | 0.01–0.15 | More unusual behavior | Pattern deviation metric |
| Phi | φ | Capital efficiency | 0.4–1.3 | Better return/risk | Return per unit risk |
| Psi⁵ | ψ⁵ | Fifth-order stability | 0.1–0.7 | More persistent trends | Hurst-like exponent |
| SE | SE | Shannon Entropy (diversity) | 0.01–0.4 | Less centralized (lower = more whale-dominated) | Transaction distribution entropy |
| Drift | drift | Directional momentum | -0.05–+0.12 | Stronger trend | Time-series momentum |
| Volatility | vol | Realized volatility | 0.05–0.45 | Higher instability | Standard deviation of indicators |

Metrics are modulated by macro-state and user roles. For backwards compatibility, unknown metrics SHOULD be ignored.

---

## **Rationale**

### **Why Metrics?**

These metrics (as implemented in frameworks like ISA Fabric) enable data-driven pre-checks. They are extensible for custom use cases.

### **Why PASS / REVIEW / FAIL?**

Binary decisions are insufficient; REVIEW handles salvageable cases.

### **Why JSON?**

- Machine‑readable  
- CI/CD compatible  
- Easy to diff in PRs  
- Works with dashboards and bots  

### **Use Cases**

- **DEX Upgrade**: Evaluate gas optimizations during high volatility.
- **LENDING Parameter Tweak**: Assess interest rate changes against SE and phi for decentralization risks.
- **PERP Leverage Adjustment**: Forecast volatility increases in perpetuals domain.

---

## **Backwards Compatibility**

Schemas include a `metadata.version` field (default "1.0"). Future versions MUST be backwards-compatible or use a new major version. Implementations SHOULD handle unknown fields gracefully.

---

## **Reference Implementation**

A reference implementation is available in ISA Fabric CLI v0.4.1 (open-source at [github.com/isa-fabric/cli](https://github.com/isa-fabric/cli)). It generates compliant proposal and evaluation JSONs via commands like `isa governance run --format json`.

---

## **Security Considerations**

- This EIP does not execute on-chain logic and incurs no gas costs.  
- It reduces governance risk by preventing dangerous proposals from reaching execution.  
- Implementers must ensure evaluation engines are tamper‑resistant (e.g., use signed telemetry sources) and auditable.  
- Risks include: Over-reliance on off-chain scores (mitigate with on-chain overrides); optimistic confidence values (enforce evidence requirements); metadata tampering (validate via hashes or signatures).  

---

## **Reference Examples**

This EIP includes three canonical evaluation examples as normative guidance:

### **1. FAIL Case**  
“Risky Gas Upgrade v5”  
- Score: 0.612 < 0.78  
- 4 critical flags  
- Recommendation: REJECT  

### **2. REVIEW Case**  
“Leverage Cap Adjustment v3.2”  
- Score: 0.749 ≥ 0.74  
- Multiple warnings  
- Recommendation: REVIEW REQUIRED  

### **3. PASS Case**  
“Gas Optimization & Safety Upgrade v5.1”  
- Score: 0.847 > 0.76  
- No critical flags  
- Recommendation: DEPLOY RECOMMENDED  

---

## **Copyright**

This document is licensed under the Creative Commons Attribution 4.0 International License.


FAQ Section: 

Q: Is this replacing on‑chain governance?
No. This standard defines an off‑chain evaluation layer. It complements systems like OpenZeppelin Governor by providing structured risk analysis before proposals reach the chain.

Q: Why JSON instead of Solidity structs?
Because evaluation happens off‑chain, often in CI/CD pipelines, dashboards, or risk engines. JSON is portable, diff‑friendly, and widely supported.

Q: Can protocols extend the schema?
Yes. The metadata field allows arbitrary extensions. Unknown fields must be ignored for forward compatibility.

Q: Are the metrics (β, SE, drift, etc.) required?
No. They are recommended. Implementers may add custom metrics or domain packs.

Q: Why include a REVIEW state?
Binary PASS/FAIL is too coarse. REVIEW captures borderline cases where human judgment is needed.

Q: Does this require ISA Fabric?
No. ISA Fabric is one implementation. Any engine that produces compliant JSON is compatible.

