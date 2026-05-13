# External Work Dependency Map

**Section 4 of the discovery deliverables. Canonical baseline registry of external work touching Anna's research areas.**

Created: 2026-05-09. Last updated: 2026-05-12 (reflects citations now in Paper B Master and Divergence Taxonomy after B1 system-spec re-search, AEGIS scoop mitigation, and Path 2 audit work that added three citations to align the master with the locked Phase 2 framing).

Encountered during the discovery sweep covering academic (arXiv cs.CR, cs.AI, cs.SE), industry/blog/conference, and standards/regulatory channels. Time bound: 2024-01 to present, weighted toward 2025-2026.

Section 1 (Overlap Map) is the actionable subset of this registry. Section 4 is the full registry.

Status legend: `cited` = already in Anna's papers. `to-cite` = recommended for inclusion. `deferred` = relevant but not for current papers. `declined` = considered and rejected. `undecided` = needs Anna's call.

Garcia flag legend: `[also in Garcia et al.]` = the structural pattern, framing, or specific phrasing also appears in Garcia et al. (Stratosphere v2 draft). Common immunology and AIS field knowledge is not flagged.

Provenance tags: `[source claim]` = direct from external source. `[my interpretation]` = my reading of the source's relevance to Anna. `[my extrapolation]` = inferential beyond what the source says.

## Update log

- **2026-05-12 (this update):** Two-part update.
  - *Path 2 registry reconciliation:* Ten items moved from to-cite/undecided to cited (items 10, 11, 12, 13, 14, 17, 18, 20, 37, 38) reflecting actual placement in Paper B Master and Divergence Taxonomy. Twelve new items added to cited (items 48-59) that were not previously in the registry.
  - *Items 1-9 audit:* Verified each cited entry against the actual master file. Found three items in cited that were NOT actually cited anywhere in the paper: Bhardwaj (item 2), Mind the GAP (item 4), Astrix (item 9). The locked Phase 2 framing intended these citations but they never made it into the master file substitution. To align the registry with the paper rather than vice versa, all three citations were added to the master in this session: Bhardwaj to Section 4.2 Tier 2 'Formal backing' field, Mind the GAP to Divergence Taxonomy Section 2 'Layer-spanning empirical confirmation' paragraph, Astrix to Section 2.1 Motion Detector Problem with reference [345] added to the References section. Hook fields for items 1, 3 corrected to remove overstated claims (Section 13.2 references; non-existent Divergence Taxonomy concurrent-work subsection). Item-19 caveat preserved: arXiv:2603.16938 is a separate Aegis architecture from AEGIS (arXiv:2603.12621) and remains undecided pending verification.

---

## Status: cited (already in Anna's papers)

| # | Work | arXiv / URL | Domain | Hook into Anna's work | Garcia flag |
|---|------|-------------|--------|----------------------|-------------|
| 1 | Errico, AARM | arXiv:2602.09433 | Academic (cs.CR) | Paper B Section 4 substrate convergence paragraph (HMAC-receipt logging precedent) | - |
| 2 | Bhardwaj, Agent Behavioral Contracts (ABC), Drift Bounds Theorem | arXiv:2602.22302 | Academic (cs.AI) | Paper B Section 4.2 Tier 2 "Formal backing" field (Drift Bounds Theorem as formal complement to drift rate metric; concurrent independent work converging on the same architectural claim) | - |
| 3 | Basu, NabaOS Tool Receipts | arXiv:2603.10060 | Academic (cs.CR) | Paper B Section 4 substrate convergence paragraph (HMAC-receipt logging) | - |
| 4 | Cartagena and Teixeira, Mind the GAP | arXiv:2602.16943 | Academic (cs.AI) | Divergence Taxonomy Section 2 "Layer-spanning empirical confirmation" paragraph (GAP/LEAK/RAD types mapping structurally to Divergence Types 1, 4, 5, 8; layer-invariance hypothesis) | - |
| 5 | Maloyan and Namiot, Breaking the Protocol (MCP) | arXiv:2601.17549 | Academic (cs.CR) | Paper B Reference [344], 23%-41% MCP attack amplification; Scenario #22 Tool Substitution context | - |
| 6 | Garcia et al., Rethinking Cybersecurity Defense (Stratosphere blog v1 PDF) | https://www.stratosphereips.org/s/Rethinking_the_Principles_of_Immunity_For_a_Cybersecurity_Immune_System-Draftv1.pdf and blog post 2026-02-04 | Industry/blog | Paper B Reference [337], Section 13 concurrent work; complementary network-layer scope, 16 vs 37 mappings | self |
| 7 | Matzinger, The Danger Model | Science 296(5566):301-305, 2002, DOI 10.1126/science.1071059 | Academic (immunology) | Paper B Reference [347], cited at Section 2.4 Related Work; multi-signal danger framework underlying Audit Stack divergence-signal approach | shared anchor |
| 8 | Schrom et al., Challenges in cybersecurity: Lessons from biological defense systems | Mathematical Biosciences 362, 109024, 2023 | Academic | Paper B Section 2.4 (cited as "[309, cited in [340]]"); Biomimetic Gap Analysis explicitly "builds on and extends" per Zenodo abstract | - |
| 9 | Astrix Security, State of MCP Server Security 2025 | https://astrix.security/learn/blog/state-of-mcp-server-security-2025/ | Industry | Paper B Section 2.1 Motion Detector Problem (5,200+ MCP servers, 88% credentials, 53% static secrets, 8.5% OAuth); Reference [345] | - |
| 10 | IETF draft-sharif-agent-audit-trail-00, Agent Audit Trail: A Standard Logging Format for Autonomous AI Systems | https://datatracker.ietf.org/doc/draft-sharif-agent-audit-trail/ | Standards (IETF) | Paper B Section 4 substrate convergence paragraph (protocol-layer standardization of the receipt primitive) | - |
| 11 | Yuan, Su, Zhao - AEGIS: No Tool Call Left Unchecked: A Pre-Execution Firewall and Audit Layer for AI Agents | arXiv:2603.12621 | Academic (cs.CR) | Paper B Section 4 substrate convergence (Ed25519 + SHA-256 hash chaining); Section 4.4 Relationship to Concurrent Architectures (closest architectural analog with full differentiation); Section 4.5 Contribution Boundary (referenced as the contested architectural ground) | - |
| 12 | Uchibeke - Before the Tool Call: Deterministic Pre-Action Authorization (Open Agent Passport) | arXiv:2603.20953 | Academic (cs.CR) | Paper B Section 4.4 Relationship to Concurrent Architectures (pre-action authorization layer angle, "AI agents have passwords but no permission slips") | - |
| 13 | Kaptein, Khan, Podstavnychy - Runtime Governance for AI Agents: Policies on Paths | arXiv:2603.16586 | Academic (cs.CR) | Paper B Section 4.4 Relationship to Concurrent Architectures (path-dependent runtime governance with EU AI Act alignment; applicable at Tier 3 disclosure enforcement) | - |
| 14 | Chinaei - Causality Laundering: Denial-Feedback Leakage in Tool-Calling LLM Agents | arXiv:2604.04035 | Academic (cs.CR) | Divergence Taxonomy Type 1 "Related operational work" field (Agentic Reference Monitor / ARM as denial-specific runtime defense); Paper B Section 4.4 (narrow alternative architecture for one Divergence Type) | - |
| 17 | Aegon - Auditable AI Content Access with Ledger-Bound Tokens and Hardware-Attested Mobile Receipts | arXiv:2604.06693 | Academic (cs.CR) | Paper B Section 4 substrate convergence paragraph (ledger-bound tokens and hardware-attested receipts as substrate variant) | - |
| 18 | Right to History: A Sovereignty Kernel for Verifiable AI Agent Execution | arXiv:2602.20214 | Academic (cs.CR) | Paper B Section 4 substrate convergence paragraph (RFC 6962-style Merkle audit logs in Rust kernel-level TCB) | - |
| 20 | Winston et al. - Solver-Aided Verification of Policy Compliance in Tool-Augmented LLM Agents | arXiv:2603.20449 | Academic (cs.SE) | Paper B Section 4.4 Relationship to Concurrent Architectures ("related concurrent work" - SMT (Z3) pre-execution policy checking) | - |
| 37 | AgentSentry - Mitigating Indirect Prompt Injection via Temporal Causal Diagnostics | arXiv:2602.22724 | Academic (cs.CR) | Paper B Scenario #23 Incremental Attention Drift "Parallel terminology" field (temporal causal diagnostics over accumulated agent context) | - |
| 38 | Session Risk Memory (SRM): Temporal Authorization for Deterministic Pre-Execution Safety Gates | arXiv:2603.22350 | Academic (cs.CR) | Paper B Scenario #23 Incremental Attention Drift "Parallel terminology" field (trajectory-level risk with session-level semantic centroid and risk EMA) | - |
| 48 | Yang - AgentTrust: Runtime Safety Layer for MCP-Compatible Tool Calls | arXiv:2605.04785 | Academic (cs.CR) | Paper B Section 4.4 Relationship to Concurrent Architectures ("related concurrent work" - RiskChain for multi-step attack chains; allow/warn/block/review verdict) | - |
| 49 | ARGUS: Runtime Defense Against Context-Aware Prompt Injection | arXiv:2605.03378 | Academic (cs.CR) | Paper B Section 4.4 Relationship to Concurrent Architectures ("related concurrent work"); Scenario #23 "Parallel terminology" field (names broader class "context-aware prompt injection") | - |
| 50 | Governed MCP: Kernel-Level Tool Governance via Logit-Based Safety Primitives | arXiv:2604.16870 | Academic (cs.CR) | Paper B Section 4.4 Relationship to Concurrent Architectures ("related concurrent work" - Anima OS / WASM ABI surface mediation; kernel-level operation below agent privilege boundary) | - |
| 51 | Zhao et al. - Nitro: High-Performance Tamper-Evident Logging via eBPF | arXiv:2509.03821 | Academic (cs.CR) | Paper B Section 4 substrate convergence paragraph (tamper-evident logging at eBPF substrate level) | - |
| 52 | Protecting Context and Prompts: Deterministic Security for Non-Deterministic AI | arXiv:2602.10481 | Academic (cs.CR) | Paper B Section 4 substrate convergence paragraph (hash chain + sequence numbers + tamper-evident state defense against context poisoning) | - |
| 53 | Walden - Reasoning Models Will Blatantly Lie About Their Reasoning | arXiv:2601.07663 | Academic (cs.CR) | Divergence Taxonomy Type 1 "Experimental analog" field (controlled experimental demonstration of denial: reasoning models flatly deny using hints when directly asked) | - |
| 54 | Chen et al. - Reasoning Models Don't Always Say What They Think (Anthropic) | arXiv:2505.05410 | Academic (cs.AI) | Divergence Taxonomy Section 2 "Relationship to prior work" (foundational citation: faithfulness rates as low as 25% for Claude 3.7 Sonnet and DeepSeek-R1 on some hint types) | - |
| 55 | Arcuschin et al. - Chain-of-Thought Reasoning In The Wild Is Not Always Faithful | arXiv:2503.08679 | Academic (cs.AI) | Divergence Taxonomy Section 2 "Relationship to prior work" (post-hoc rationalization rates in production models) | - |
| 56 | R.J. Young - Why Models Know But Don't Say: CoT Faithfulness Divergence Between Thinking Tokens and Answers in Open-Weight Reasoning Models | arXiv:2603.26410 | Academic (cs.AI) | Divergence Taxonomy Section 2 "Relationship to prior work" (12 open-weight models, 10,506 hint-followed cases, 59-percentage-point gap between thinking-token and answer-text acknowledgment) | - |
| 57 | DeepContext - Stateful Real-Time Detection of Multi-Turn Adversarial Intent Drift in LLMs | arXiv:2602.16935 | Academic (cs.CR) | Paper B Scenario #23 Incremental Attention Drift "Parallel terminology" field (sliding-window LLM-based detection; names failure class "intent drift") | - |
| 58 | Taming OpenClaw - Lifecycle Threat Taxonomy | arXiv:2603.11619 | Academic (cs.CR) | Paper B Scenario #23 Incremental Attention Drift "Parallel terminology" field (lifecycle taxonomy explicitly names "memory poisoning and context drift, gradually eroding adherence to user's original instructions") | - |
| 59 | Agent Drift: Quantifying Behavioral Degradation in Multi-Agent LLM Systems Over Extended Interactions | arXiv:2601.04170 | Academic (cs.CR) | Paper B Scenario #23 Incremental Attention Drift "Parallel terminology" field (quantitative empirical study; names failure class "behavioral degradation") | - |

## Status: to-cite (high-priority adds)

| # | Work | arXiv / URL | Domain | Hook into Anna's work | Garcia flag | Reason |
|---|------|-------------|--------|----------------------|-------------|--------|
| 15 | ClawSafety: "Safe" LLMs, Unsafe Agents | arXiv:2604.01438 | Academic (cs.AI) | Divergence Taxonomy Type 1 (Denial); Mind the GAP companion empirical work | - | Cites Mind the GAP as canonical, builds benchmark on text-vs-tool divergence. Empirical confirmation of the divergence class Anna documents at the model-output layer. |
| 16 | Bhattarai and Vu, Trustworthy Agentic AI Requires Deterministic Architectural Boundaries (Trinity) | arXiv:2602.09947 | Academic (cs.CR) | Paper B Section 3.5 Compliance Override; Section 4; Section 12 Practitioner Actions | - | Deterministic command/data separation as authorization-integrity boundary. Same architectural-overrides-prompt-controls thesis as Anna. |

## Status: undecided (worth a Section-13.2 sentence or footnote, pending Anna's call)

| # | Work | arXiv / URL | Domain | Hook into Anna's work | Garcia flag | Reason |
|---|------|-------------|--------|----------------------|-------------|--------|
| 19 | Cryptographic Runtime Governance / Aegis architecture | arXiv:2603.16938 | Academic (cs.CR) | Paper B Section 4 Tiers 1+3 | - | Verifiable policy enforcement. Architectural overlap with AEGIS (2603.12621); confirm not the same paper before citing both. |
| 21 | Wang, Poskitt, Sun, AgentSpec | arXiv:2503.18666, ICSE 2026 | Academic (cs.SE) | Paper B Section 4 Tier 3 | - | DSL for runtime rule enforcement. ICSE 2026 publication is recent. Overlapping enforcement category. |
| 22 | Agent-Sentry: Bounding LLM Agents via Execution Provenance | arXiv:2603.22868 | Academic (cs.CR) | Paper B Section 4 Tier 2; Tier 1 (provenance) | - | Functionality-graph mediation with intent-alignment. |
| 23 | KAIJU, Executive Kernel for Intent-Gated Execution | arXiv:2604.02375 | Academic (cs.CR) | Paper B Section 4 Tier 2; Section 3.6 Entangled Representations (execution/reasoning split) | - | Strict execution-vs-reasoning split is structurally close to the OOB Optimization constraint. |
| 24 | AgentArmor, Enforcing Program Analysis on Agent Runtime Trace | arXiv:2508.01249 | Academic (cs.CR) | Paper B Section 4 Tier 2 | - | Program analysis on runtime traces. Earlier than 2026-cluster but methodologically distinct. |
| 25 | Right-to-Act: Pre-Execution Non-Compensatory Decision Protocol | arXiv:2604.24153 | Academic (cs.CR) | Paper B Section 3 Audit Gap framework; Section 12 | - | Non-compensatory pre-execution boundary. Aligns conceptually with Friction Constraint. |
| 26 | OpenPort Protocol: Security Governance Specification for AI Agent Tool Access | arXiv:2602.20196 | Academic (cs.CR) | Paper B Section 4; Scenario #22 Tool Substitution | - | Server-side gateway as enforcement point. Same-genre system spec. |
| 27 | Towards Verifiably Safe Tool Use for LLM Agents (Doshi) | arXiv:2601.08012 | Academic (cs.CR) | Paper B Section 4 Tiers 2+3 | - | STPA-derived formal safety specifications. Methodological complement. |
| 28 | OpenClaw PRISM | arXiv:2603.11853 | Academic (cs.CR) | Paper B Section 4 Tier 3; Section 12 | - | Zero-fork, defense-in-depth runtime layer. Implementation-side neighbor. |
| 29 | Verifier Tax: Horizon-Dependent Safety-Success Tradeoffs | arXiv:2603.19328 | Academic (cs.AI) | Paper B Section 4 (cost-benefit framing for Audit Stack); Section 12 Practitioner Actions ("wrong config is worse than no config") | - | Empirical evidence that runtime mediation has measurable cost. Useful for honest Section 12 framing. |
| 30 | Maloyan and Namiot, Prompt Injection Attacks on Agentic Coding Assistants (SoK) | arXiv:2601.17548 | Academic (cs.CR) | Paper B Scenario #22 Tool Substitution; #15 Digital Biofilm via skill ecosystem; ATLAS extension | - | Sibling of [344]. SoK on coding-assistant prompt injection. Relevant to skill-registry attacks (ClawHavoc). |
| 31 | MCP Threat Modeling (STRIDE/DREAD on 57 MCP threats) | arXiv:2603.22489 | Academic (cs.CR) | Paper B Scenario #22; ATLAS-extension argument | - | Cites Maloyan/Namiot's ATTESTMCP. STRIDE/DREAD coverage of 57 MCP threats. |
| 32 | MCP-DPT: Defense-Placement Taxonomy | arXiv:2604.07551 | Academic (cs.CR) | Paper B Section 4 Tiers; Section 12 | - | Defense placement taxonomy. Structural complement to Anna's Three-Tier Audit Stack. |
| 33 | Authorization Propagation in Multi-Agent AI Systems | arXiv:2605.05440 | Academic (cs.CR) | Paper B Scenario #12 Identity Rotation; Section 4 | - | Most recent in the cluster (May 2026). Identity governance as infrastructure. Authorization propagation across multi-agent chains. |
| 34 | Kaptein/Levy/Perl, Bounding the Black Box: Statistical Certification | arXiv:2604.21854 | Academic (cs.CR) | Paper B Section 12 Practitioner Actions | - | Certification framework for AI risk regulation. Adjacent. |
| 35 | Benameur et al., OIDC-A: OpenID Connect for Agents 1.0 | arXiv:2509.25974 | Standards/Academic | Paper B Scenario #12 Identity Rotation; Section 4 | - | Identity protocol extension for agents. Direct relevance to credential-bearing agent scenarios. |
| 36 | MI9: Integrated Runtime Governance Framework | arXiv:2508.03858 | Academic (cs.CR) | Paper B Section 4 (continuous authorization, drift detection, graduated containment) | - | "Goal-conditioned behavioral deviation" detection is structurally close to Tier 2 drift detection. |
| 39 | Task Shield: Enforcing Task Alignment | arXiv:2412.16682 | Academic (cs.CR) | Paper B Section 4 Tier 3; Divergence Taxonomy intent-alignment angle | - | Test-time goal-alignment verification of every instruction and tool call. Earlier (Dec 2024) than the 2026 cluster but on the same axis. |
| 40 | Probabilistic Model Checking enforcement of LLM agent safety | arXiv:2508.00500 | Academic (cs.SE) | Paper B Section 4 Tier 2 / Tier 3 | - | PCTL/DTMC formal verification. Methodological complement to Bhardwaj's stochastic drift bound. |
| 41 | The Observability Gap: Why Output-Level Human Feedback Fails for LLM Coding Agents | arXiv:2603.26942 | Academic (cs.SE) | Honesty Decay; Divergence Taxonomy structural framing; Paper B Section 3.1 Output Optimization | - | "Many-to-one mapping from internal states to visible outcomes prevents reliable inference." Identical structural argument to Output Optimization constraint, applied to coding-agent context. |

## Status: deferred (relevant but not for Paper B / Talk 2)

| # | Work | arXiv / URL | Domain | Hook | Garcia flag | Reason |
|---|------|-------------|--------|------|-------------|--------|
| 42 | Mavračić, Policy Cards | arXiv:2510.24383 | Standards/regulatory | Future regulator-facing paper | - | Locked decision 2026-05-07: defer from Paper B and Talk 2. Scope into a future regulator-facing paper for NIST AI RMF / ISO 42001 / EU AI Act crosswalks. |
| 43 | Tallam, Cyber Immune System: Adversarial Forces for Resilience | arXiv:2502.17698 | Academic (cs.CR) | Biomimetic Gap Analysis; Motion Detector Framework adjacent | self (immunity framing) | Earlier 2025 work. Different angle (adversarial-as-immune-stress). Adjacent, not competitive. |
| 44 | CSA blog, AARM: Securing the Agentic Runtime, April 30, 2026 | https://cloudsecurityalliance.org/blog/2026/04/30/aarm-finding-a-path-to-secure-the-agentic-runtime | Industry/standards | AARM Builders Registry submission path for Anna's tools (mcp-tap, coffer-mcp, motion-detector-framework) | - | Per memory edit (Options 2+3): AARM-conformant tooling via Builders Registry is the recommended positioning. CSA blog confirms AARM is now a CSA-led research initiative. |

## Status: declined (considered, not for inclusion)

| # | Work | arXiv / URL | Domain | Reason |
|---|------|-------------|--------|--------|
| 45 | Various older AIS surveys (e.g., Widuliński 2023, Yang et al. 2014) | - | Academic | Already in Garcia's 51-paper prior-work corpus and in Anna's biomimetic backbone. No incremental value for Paper B. |
| 46 | Generic LLM safety / sycophancy benchmarks | various | Academic | Out of scope for Paper B; covered exhaustively in Anna's 78-paper Sycophancy and Delusion catalog. |
| 47 | Generic anomaly detection literature | various | Academic | Out of scope; Anna's framework is at identity / tool-use layer, not network-layer anomaly detection. |

---

## Cross-tabulation: Anna's work artifact -> external work concentration

[my interpretation] The registry is heavily weighted toward the system-spec genre cluster around AARM, ABC, NABAOS, and Mind the GAP. This cluster did not exist when Anna started Paper B; the bulk of it appeared between February and April 2026. The implication for Section 13.2 is that it is no longer sufficient to call out AARM, Bhardwaj, NABAOS, and Mind the GAP individually. The genre is now a 30-paper field with shared vocabulary (interception, mediation, receipts, intent alignment, session context). Paper B's contribution is narrower than this cluster (attack-side scenario derivation from biology) and is therefore protected from scoop risk in that respect, but the architectural Three-Tier Audit Stack overlaps directly with the cluster's defense-side work.

[my extrapolation] If a single paper were going to scoop the Three-Tier Audit Stack as an architecture, AEGIS (arXiv:2603.12621) is the closest candidate. Its pre-execution firewall plus tamper-evident audit log is the same architectural pattern. The differentiator for Anna's Paper B remains: (a) biological derivation of the 23 scenarios that motivate the architecture, (b) the OOB-as-evidentiary-standard claim that the architecture serves, (c) the cross-paper integration with the Divergence Series taxonomy. None of those differentiators are addressed by AEGIS or any other paper in the cluster.

**2026-05-12 update:** Paper B now explicitly addresses the AEGIS scoop risk. Section 4 substrate convergence paragraph names nine independent rediscoveries of the receipt primitive (no novel substrate claim). Section 4.4 names AEGIS as the closest analog and states the policy-enforcement-vs-forensic-detection differentiation. Section 4.5 (Contribution Boundary) explicitly concedes the architectural pattern as contested ground and lists what is novel: bio-derived attack scenarios, OOB evidentiary standard, Compliance Override constraint, Divergence Series taxonomic integration. Tier 1/2/3 each have inline biological grounding paragraphs (MHC-I, T-cell receptor, complement cascade) making the biology load-bearing at the architecture description rather than appearing only in Section 6.

## Caveats

- OpenAlex `cited_by_count` for Zenodo 10.5281/zenodo.19393455 is 0 as of the OpenAlex snapshot 2026-04-04, one day after deposit. Subsequent citers, if any, are not yet indexed.
- Stratosphere blog forward-citation search returned no external work citing the public blog URL. Not a verified zero given the search tools available.
- Several arXiv IDs in the to-cite and undecided sections (2603.16938, 2603.22868, 2604.02375, 2603.11853, 2603.19328, 2603.22489, 2604.07551, 2604.21854, 2604.01438) were verified to exist via search-result snippets but have not been individually fetched. Confidence high but not 100%. Originally this list also included 2603.12621, 2604.06693, 2602.20214, 2603.20449, 2603.22350, 2602.22724, and 2604.04035; those are now individually confirmed via citation in Paper B and/or Divergence Taxonomy (2026-05-12).
- The Garcia draft v2 received 2026-05-09 was machine-read. The 16 principles, the 51-paper prior-work matrix, the Matzinger reference [22], and the Forrest/Hofmeyr/Somayaji [1, 63] references are confirmed. No machine-read errors observed in the parts of the draft relevant to overlap analysis.
