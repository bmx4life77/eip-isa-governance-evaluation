# **EIP‑XXXX: ISA‑Compatible Governance Proposal & Evaluation Schema** 

### *Standardized Off‑Chain Risk Evaluation for Pre‑On‑Chain Governance Decisions*

**Author:** Louis Pearson (kctiDAO / Adaptive Experimental Research Network Collective)  
**Status:** Draft  
**Type:** Standards Track  
**Category:** ERC  
**Created:** 2026‑01‑12  
**Updated:** 2026‑01‑20 (v0.5 — Polished and aligned with ISA Fabric v1.0, Genesis Governance v4.0, Theta Promotion Charter v1.0)  
**Requires:** ERC‑4824  
**Replaces:** None  

---

## **Abstract**

This EIP defines a standardized, machine‑readable schema for **off‑chain governance proposals** and **quantitative evaluation reports** used in pre‑on‑chain governance pipelines. The standard enables protocols to perform structured, metrics‑driven risk assessment *before* submitting proposals to on‑chain governance systems such as OpenZeppelin Governor.

It introduces:

- A canonical **proposal JSON schema**  
- A canonical **evaluation JSON schema**  
- A tri‑state decision model (**PASS / REVIEW / FAIL**)  
- A domain‑aware metrics interpretation layer aligned with ISA Fabric’s 72×72 Domain Library  
- A consistent interface for CI/CD gating, dashboards, and governance tooling  

This EIP does **not** replace on‑chain governance. It formalizes the **pre‑flight evaluation layer** that prevents dangerous or poorly timed proposals from reaching binding execution.

This standard adheres to Genesis Governance v4.0’s **Metric Authority Doctrine**:  
**only ψ₅ (Security Posture) and SE (System Efficiency) may influence execution state.**  
All other metrics — including Θ‑family operators — are strictly descriptive and non‑authoritative.

---

## **Motivation**

On‑chain governance systems provide secure execution of community decisions but lack a standardized mechanism for **pre‑submission risk evaluation**. As a result, protocols frequently encounter:

- High‑risk proposals submitted during volatile conditions  
- Poorly justified parameter changes  
- Over‑optimistic claims without quantitative backing  
- Governance spam or low‑quality upgrades  
- Lack of structured metadata for automated review  

This EIP provides a unified schema for:

- Governance dashboards  
- CI/CD pipelines  
- Risk engines  
- DAO tooling  
- Off‑chain simulation frameworks  
- On‑chain proposal gating systems  

### **Relationship to Existing Standards**

| Layer | Tool | Role |
|-------|------|------|
| **Off‑chain evaluation** | This EIP (via ISA Fabric or compatible engines) | Risk scoring, forecasting, pre‑flight checks |
| **On‑chain execution** | OpenZeppelin Governor | Voting, timelock, execution |

These layers are **complementary**, not competitive.

This EIP explicitly separates:

- **Descriptive metrics** (β, VU, ι, φ, divergence, Θ‑regimes)  
- **Authoritative metrics** (ψ₅, SE)  

This prevents off‑chain evaluation engines from exerting unintended influence over execution state.

---

## **Specification**

This section defines:

1. Governance Proposal Schema  
2. Governance Evaluation Schema  
3. Status Codes  
4. Metric Interpretation Rules  
5. RASUV Decomposition (Normative)  
6. Envelope Production Requirements  

All metrics align with ISA Fabric’s three layers:

- **Core Signals**  
- **Derived Intelligence**  
- **Domain‑Specific Intelligence**  

And with RASUV semantics (Real, Actual, Structural, Uncertain, Virtual).

---

# **1. Governance Proposal Schema**

*(Schema unchanged from v0.4 except for CLI trace refinement.)*

The `cli_trace` field now uses the reproducibility‑focused description:

> “A reproducible sequence of ISA Fabric CLI commands used during evaluation. Engines SHOULD ensure deterministic results when provided with identical telemetry and CLI traces.”

---

# **2. Governance Evaluation Schema**

*(Schema unchanged except for corrected example and CLI trace refinement.)*

In examples, `cli_trace` MUST be a simple string, not a schema object.

---

# **3. Status Codes and Semantics**

| Status | Meaning | Conditions |
|--------|---------|------------|
| **PASS** | Safe to proceed to on‑chain submission | Score ≥ threshold AND no critical flags (ψ₅/SE only) |
| **REVIEW** | Borderline; requires human review | Score ≥ threshold BUT warnings present; Θ‑regime may inform narrative |
| **FAIL** | High risk; do not proceed | Score < threshold OR any critical flag |

**Authority Separation:**  
- ψ₅ and SE may influence PASS/FAIL.  
- Θ‑regime is descriptive only and MUST NOT alter PASS/FAIL outcomes.

---

# **4. Metric Interpretation Rules**

Domains and sub‑domains SHOULD reference the ISA Fabric 72×72 Domain Library.  
If omitted, domain MAY be auto‑detected using ISA‑MATRIX semantic signatures.

### **Metric Table**

| Metric | Symbol | Meaning | Range | Higher Means | Authority |
|--------|--------|---------|--------|--------------|-----------|
| Beta | β | Throughput / Stability | 0–1 | Higher correlated risk | Descriptive |
| SE | SE | System Efficiency | 0–1 | Higher efficiency | **Authoritative** |
| VU | VU | Volatility | 0–1 | Higher instability | Descriptive |
| Iota | ι | Momentum | 0–1 | Stronger acceleration | Descriptive |
| Phi | φ | Recovery / Resilience | 0–1 | Faster recovery | Descriptive |
| Psi5 | ψ₅ | Security Posture | 0–1 | Stronger safety | **Authoritative** |
| Divergence | δ | Deviation from expected path | 0–1 | Higher deviation | Descriptive |
| Theta Regime | Θ | Temporal structure | Enum | Narrative only | Non‑authoritative |
| RASUV SE | SEₛ | Structural Exposure | 0–1 | Higher exposure | Derived |
| RASUV SI | SIₛ | Structural Integrity | 0–1 | Stronger posture | Derived |
| RASUV SP | SPₛ | Structural Pressure | 0–1 | Higher strain | Derived |

### **Authority Separation (Normative)**

- ψ₅ and SE → **authoritative**  
- Θ‑family → **descriptive only**  
- All others → **descriptive**  

---

# **5. RASUV Decomposition (Normative)**

### **5.1 Perspectives**

| Perspective | Symbol | Meaning |
|------------|--------|---------|
| Real | R | Observed telemetry |
| Actual | A | Validated value |
| Structural | S | Model‑based expectation |
| Uncertain | U | Probabilistic bound |
| Virtual | V | Simulated / forecasted |

### **5.2 Requirements**

- MUST be normalized to \([0,1]\)  
- MUST be included for ψ₅  
- SHOULD be included for all metrics in `impactAnalysis`  
- MUST NOT influence execution state directly  
- MUST be deterministic given identical telemetry + CLI trace  

### **5.3 Integration with ISA‑MATRIX**

RASUV maps directly to ISA‑MATRIX’s five measurement perspectives.

### **5.4 Integration with ψ₅**

ψ₅ is computed as a geometric aggregation of five RASUV‑derived security dimensions:

- Reentrancy  
- Access  
- State Exposure  
- Upgrade Safety  
- Verification Strength  

ψ₅ is authoritative; RASUV itself is descriptive.

---

# **6. Envelope Production Requirements**

Evaluation engines SHOULD produce a canonical **ISA Metrics Envelope** as defined in ISA Fabric v1.0.

Envelopes serve as:

- machine‑readable evidence  
- CI/CD artifacts  
- governance audit logs  
- reproducibility anchors  

Envelopes MUST include:

- core metrics  
- derived metrics  
- RASUV decomposition  
- Θ‑family outputs (descriptive only)  
- CLI trace  
- domain/sub‑domain classification  

---

# **Rationale**

### **Constitutional Alignment**

This EIP is fully compatible with Genesis Governance v4.0:

- **Article 0 — Metric Authority Doctrine:** Only ψ₅ and SE may influence execution state.  
- **Article II — Proposal Lifecycle:** Evaluations correspond to pre‑validation and pre‑execution phases.  
- **Article III — Closed‑Loop Architecture:** Evaluation engines act as the observational layer.  
- **Article V — Calibration & Evolution:** Schema evolution must preserve invariants and authority boundaries.

### **Why PASS / REVIEW / FAIL?**

Binary decisions are insufficient; REVIEW captures borderline cases requiring human judgment.

### **Why JSON?**

- Machine‑readable  
- CI/CD compatible  
- Diff‑friendly  
- Dashboard‑friendly  

---

# **Backwards Compatibility**

Schema evolution MUST follow Genesis Governance v4.0 Article V:

- calibration allowed  
- constitutional authority cannot change  
- execution authority cannot expand  

Unknown fields MUST be ignored.

---

# **Security Considerations**

- No on‑chain execution; no gas cost  
- Reduces governance risk by enforcing Metric Authority Doctrine  
- Evaluation engines MUST be tamper‑resistant  
- Θ MUST remain non‑authoritative  
- Metadata SHOULD be signed or hashed  

---

# **Reference Examples**

*(FAIL, REVIEW, PASS examples unchanged except for corrected CLI trace formatting.)*
