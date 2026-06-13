# hallmark

Multi-category LLM-generated attack fingerprinter — detects statistical signatures left by language models in credentials, phishing, payloads, and malicious code.

**GitHub:** `github.com/WFullen94/hallmark`
**Thesis:** LLMs leave detectable fingerprints in everything they generate. GitGuardian proved this for passwords (Markov chains, 2x baseline detection, 65% provider accuracy). hallmark extends the methodology to the full attack surface: phishing emails, social engineering, injection payloads, and malicious code. The research question is whether LLM fingerprints generalize across attack categories — and whether a fingerprint trained on one category can attribute content in another.

**Deliverables:** Research blog series (2 posts) + arXiv preprint. Direct extension of GitGuardian's work with multi-category analysis and cross-category transfer experiments.

---

## What this project does

Attackers are increasingly using LLMs to generate attack content at scale: well-written spear phishing that bypasses keyword filters, credential lists with realistic patterns, injection payloads that look syntactically legitimate, and malicious code that passes style checks. Each LLM leaves a statistical fingerprint in its output — specific n-gram patterns, structural biases, vocabulary preferences — that varies by model and provider.

hallmark builds per-category fingerprint classifiers and answers three questions:
1. **Is this LLM-generated?** (binary: LLM vs. human)
2. **Which provider generated it?** (Anthropic, OpenAI, Google, Meta, Mistral, ...)
3. **Do fingerprints transfer?** (cross-category: does a credential fingerprint identify phishing from the same model?)

Attack categories covered:
- **Credentials** — passwords, API keys, tokens (extending GitGuardian directly)
- **Phishing / spear phishing** — email body text, subject lines
- **Social engineering** — Slack/Teams messages, SMS lures, pretexting scripts
- **Injection payloads** — SQL injection, prompt injection, XSS strings
- **Malicious code snippets** — short scripts, one-liners, obfuscated payloads

**Connection to whodunit:** whodunit probes a model API to infer what's running behind it. hallmark analyzes output content to infer what generated it. Complementary attribution approaches.

**Connection to tracer:** tracer detects agent hijacking at runtime. hallmark flags AI-generated attack content before or during triage. Different layers of the same threat model.

---

## Reading list

1. **GitGuardian blog — "The Bot Fingerprint"** — the direct prior work; understand their Markov chain methodology and dataset construction before writing any code
2. **Stylometry literature** — Koppel et al. on authorship attribution; the same features apply to LLM vs. human attribution
3. **LLM watermarking papers** (Kirchenbauer et al., 2023+) — distinct from hallmark's approach (we detect natural fingerprints, not injected watermarks) but informs what signals are detectable
4. **DetectGPT / RADAR** — zero-shot LLM text detection via perturbation-based scoring; a baseline to compare against

---

## Phased Roadmap

### Phase 1 — Data generation + characterization
- [ ] Generate labeled datasets per category: 1,000+ samples per model per category across Claude, GPT-4o, Gemini 1.5, Llama 3, Mistral (≥5 models, ≥3 providers)
- [ ] Collect human-generated baselines from public sources per category (rockyou/passwords, phishtank/emails, exploit-db/payloads, github/code)
- [ ] `hallmark/data/generator.py`: structured generation prompts per category — consistent enough to compare, varied enough to avoid trivial pattern matching
- [ ] EDA: visualize bigram heatmaps, character class distributions, sentence length distributions per category and model

### Phase 2 — Markov chain baseline (replicating + extending GitGuardian)
- [ ] `hallmark/fingerprinters/markov.py`: character-level Markov chains, configurable n-gram order, per-model and aggregate chains
- [ ] Apply to credentials first — reproduce GitGuardian's methodology, validate approach before expanding
- [ ] Metric: LLM-vs-human accuracy, provider accuracy, model accuracy; compare to GitGuardian's published numbers
- [ ] Apply to all categories; document where Markov chains work well vs. poorly

### Phase 3 — Feature engineering per category
- [ ] `hallmark/features/credentials.py`: bigram frequency, character class sequence patterns (upper/digit/symbol/lower runs), entropy, common substring detection
- [ ] `hallmark/features/text.py`: stylometric features — sentence length distribution, type-token ratio, POS tag distribution, punctuation density, paragraph structure, formality score
- [ ] `hallmark/features/code.py`: AST depth, variable naming entropy, comment-to-code ratio, identifier length distribution, common LLM code patterns
- [ ] `hallmark/features/payloads.py`: token sequence patterns, keyword frequency, structure regularity, encoding patterns

### Phase 4 — Multi-category classifier comparison
- [ ] `hallmark/classifiers/`: sklearn wrappers for SVM, gradient boosting, logistic regression on Phase 3 features
- [ ] `hallmark/fingerprinters/perplexity.py`: reference LM perplexity scoring as a detection signal
- [ ] `hallmark/fingerprinters/embedding.py`: embedding-based similarity to known LLM output distributions
- [ ] Ablation: which feature sets are most discriminative per category? Which classifier generalizes best?

### Phase 5 — Cross-category transfer experiments
- [ ] Train fingerprinter on category A (e.g., credentials), evaluate on category B (e.g., phishing) from the same models
- [ ] Metric: does cross-category provider accuracy exceed chance? Is there a shared "LLM signature" across content types?
- [ ] Result: Pareto chart of cross-category transfer accuracy — the core research finding

### Phase 6 — Robustness + adversarial evaluation
- [ ] Paraphrase attack: does LLM-rewriting-of-LLM-content evade the fingerprinter?
- [ ] Obfuscation: simple transformations (case change, synonym swap, l33tspeak in credentials) — does the fingerprint survive?
- [ ] New model generalization: train on known models, test on a held-out model family
- [ ] Temporal drift: do fingerprints change between model versions?

### Phase 7 — Production interface + distribution
- [ ] `hallmark.detect(content, category)` → `FingerprintResult(is_llm: bool, provider: str | None, confidence: float, features: dict)`
- [ ] CLI: `hallmark scan --file payloads.txt --category payload`
- [ ] PyPI package
- [ ] Report generation: per-category breakdown with confidence distributions

---

## Conventions

- Conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`
- Tag each phase: `v0.1.0`, `v0.2.0`, etc.
- All generated datasets versioned and documented — generation prompts stored alongside data
- Human baseline sources documented with licenses; no scraping without terms compliance
- Classifiers are deterministic: fixed random seed, reproducible splits
- `FingerprintResult` is the single output type across all categories and classifiers

## Related work in this repo

- `~/whodunit/` — model fingerprinting from black-box probing (probes the API; hallmark analyzes the content)
- `~/tracer/` — detects agent hijacking at runtime; hallmark detects AI-generated attack content at triage
- `~/llm-security/` — notebooks 02-04 (injection attacks), 16 (red teaming); attack taxonomy reference
- `~/applied-cybersecurity-ml/` — threat modeling notebooks; provides attack category framing
- `~/ai-offensive-security/` — notebook 09 (threat intel attribution); prior work on attribution methodology

## Current Status

Phase 1 — not started. Start by reading the GitGuardian post in full, then design the generation prompts for each category before writing any classifier code. The quality of the labeled dataset determines everything downstream.
