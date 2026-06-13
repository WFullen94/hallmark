# Research Directions

Open research questions that emerged from designing hallmark. Direct extension of
GitGuardian's "The Bot Fingerprint" work (May 2026) to multi-category attack content.
Each idea is a potential arXiv preprint or workshop paper.

**Overarching claim:** LLM fingerprints are not category-specific — the statistical biases
baked into a model's outputs appear consistently across credential generation, phishing
text, injection payloads, and malicious code. hallmark measures whether those fingerprints
transfer, which classifiers best detect them, and whether they can be evaded.

---

## 1. Cross-Category LLM Fingerprint Transfer

**Hypothesis:** A fingerprint classifier trained on LLM-generated credentials can correctly
attribute phishing emails, payloads, and code from the same model family — because the
underlying statistical biases are properties of the model, not the content type. Provider-
level attribution transfers more reliably than model-level attribution.

**Novel claim:** GitGuardian demonstrated fingerprinting within one category (passwords).
No work has tested whether these fingerprints transfer across attack categories. If they
do, it means a single provider-level fingerprint database generalizes across the attack
surface rather than requiring per-category training data — a significant practical
reduction in the cost of deployment.

**What we need:**
- Phase 1: labeled multi-category dataset across ≥5 models, ≥3 providers
- Phase 5: cross-category transfer experiments
- Metric: provider attribution accuracy when trained on category A, tested on category B;
  compared to within-category baseline and chance

**Potential finding:** Provider-level accuracy transfers at 60–75% across categories
(well above chance), while model-level accuracy drops sharply. The signal is
provider-level regularization patterns (instruction tuning style, safety-induced
vocabulary constraints) rather than model-specific idiosyncrasies. This means
a hallmark deployment needs one provider fingerprint database, not one per attack type.

**Target venue:** USENIX Security 2027, IEEE S&P 2027, or arXiv
**Status:** Blocked on Phase 1 + 5

---

## 2. Which Attack Categories Are Most Fingerprintable?

**Hypothesis:** LLM fingerprints are strongest in highly structured, short-form content
(credentials, payloads) and weaker in longer, less constrained content (phishing emails,
social engineering scripts) — because structural constraints force models into their
characteristic patterns. Feature importance varies by category.

**Novel claim:** GitGuardian only tested one category. No study has systematically
compared fingerprint detectability across attack types. The finding has practical
implications: defenders should prioritize fingerprinting for categories where LLM
attribution is most reliable.

**What we need:**
- Phase 2 + 3: Markov chains and feature engineering across all categories
- Phase 4: classifier comparison per category
- Metric: LLM-vs-human F1 and provider accuracy per category; feature importance ranking

**Potential finding:** Credentials and injection payloads are highly fingerprintable
(F1 > 0.85) because structural constraints are tight. Phishing text is moderately
fingerprintable (F1 ~ 0.70) via stylometric features. Social engineering and longer text
are hardest (F1 ~ 0.60) because more degrees of freedom reduce model-specific biases.
Markov chains dominate for structured content; stylometric features dominate for text.

**Target venue:** arXiv preprint, or workshop at ACL/EMNLP (language + security)
**Status:** Blocked on Phase 2 + 3

---

## 3. Fingerprint Survival Under Adversarial Obfuscation

**Hypothesis:** Simple obfuscation (case changes, synonym substitution, character
substitution in credentials) partially degrades fingerprint detection but does not
eliminate it. Provider-level attribution survives obfuscation better than model-level
attribution. Paraphrase attacks (asking a second LLM to rewrite the output) are more
effective but leave their own fingerprint.

**Novel claim:** The GitGuardian paper does not evaluate robustness. No study has
measured how much adversarial effort is required to evade LLM fingerprinting across
attack categories. The "paraphrase attack" is particularly interesting because it
produces content with two overlapping fingerprints — potentially a detectable signature
of its own.

**What we need:**
- Phase 6: robustness evaluation pipeline
- Obfuscation implementations per category: l33tspeak/case for credentials, synonym
  swap for text, encoding tricks for payloads
- Metric: fingerprint accuracy before vs. after obfuscation; attack success rate at
  different obfuscation intensities

**Potential finding:** Credential fingerprints degrade ~20% under l33tspeak but remain
above baseline; text fingerprints degrade ~15% under synonym swap. Paraphrase attacks
are more effective (~35% degradation) but produce a detectable "two-fingerprint" signature
from the paraphrasing model. An attacker who wants to evade detection needs both
targeted obfuscation and access to an unusual/small model not in the fingerprint database.

**Target venue:** IEEE S&P, USENIX Security, or ACL workshop (adversarial NLP)
**Status:** Blocked on Phase 6

---

## 4. Detecting the Fingerprint vs. Detecting the Pattern — Two Distinct Signals

**Hypothesis:** There are two distinct detection signals in LLM-generated attack content:
(1) the LLM fingerprint (provider/model-specific statistical patterns) and (2) the
attack pattern (structure of a well-formed phishing email vs. a human-written one).
These signals are correlated but independent — a human-written phishing email looks like
an attack but not like an LLM; an LLM writing a benign email looks like an LLM but not
an attack. Combining both signals yields a better triage classifier than either alone.

**Novel claim:** Existing LLM detection work focuses on one signal. Existing phishing
detection focuses on the other. Nobody has studied the interaction. For a SOC analyst,
the useful output is "this is LLM-generated AND it's a phishing attempt" — not either
alone.

**What we need:**
- Phase 3 + 4: feature engineering capturing both signals
- Comparison: fingerprint-only classifier vs. attack-pattern-only vs. combined
- Dataset: 2×2 examples — LLM phishing, human phishing, LLM benign, human benign

**Potential finding:** Combined classifier significantly outperforms either alone at
low false positive rates (< 1% FPR), which is the regime SOC analysts care about.
The joint signal is most useful for borderline cases where one signal alone is uncertain.

**Target venue:** CSET, USENIX Security workshop, or arXiv
**Status:** Blocked on Phase 3 + 4

---

## Dependency map

```
Phase 1 (data generation)       ──► All ideas
Phase 2 (Markov baseline)        ──► Ideas 1, 2
Phase 3 (feature engineering)    ──► Ideas 2, 4
Phase 4 (classifier comparison)  ──► Ideas 2, 4
Phase 5 (cross-category transfer)──► Idea 1
Phase 6 (robustness)             ──► Idea 3
```

**Natural first paper: Ideas 1 + 2** — produced directly from Phases 1–4; the
cross-category transfer and category-level comparability results are the novel
contribution over GitGuardian. Ideas 3 and 4 are natural follow-on work.

**Blog series:**
- Post 1 (after Phase 2): "Extending the Bot Fingerprint: LLM detection across attack
  categories beyond passwords"
- Post 2 (after Phase 5): "Do LLM fingerprints transfer? What cross-category attribution
  tells us about attacker tooling"
