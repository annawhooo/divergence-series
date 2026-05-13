# Paper B: The Audit Gap

## Twenty-Three Biologically-Derived Attack Scenarios Against Enterprise AI Agent Architectures

Anna Hix. April 2026.

## 1. Abstract

AI agents now hold OAuth tokens, API keys, and service accounts in production enterprise infrastructure. When compromised, they inherit valid identities, and the security stack is blind. SIEM, EDR, UEBA, and PAM were built for humans and static service accounts, not for autonomous, non-deterministic entities making tool-use decisions.

This paper presents 23 attack scenarios derived from biological pathogen evasion strategies across six taxonomic groupings (mammals and insects within Animalia, Plantae, Bacteria, pathogenic Protista, and viruses as acellular entities outside the cellular taxonomy), graded against real agent frameworks (MCP, LangChain, CrewAI). 19 of 23 scenarios (83%) are trivially or feasibly exploitable under default configurations. Hardening eliminates every trivial scenario but does not eliminate a single scenario entirely: every trivial becomes feasible, every feasible stays feasible, every advanced stays advanced. Hardening raises the floor but does not change the ceiling.

The paper develops the conceptual framework underlying these findings: a six-constraint *Audit Gap* that makes current AI agent architectures mechanically incapable of reliable self-correction. The six structural constraints are (1) Output Optimization, (2) Recursive Blind Spot, (3) Active Context Reconstruction, (4) Friction Constraint, (5) Compliance Override, and (6) Entangled Representations. Each constraint is architecturally grounded and mechanistically explained. Together they create a structural zone where no mechanism operates to ensure fidelity between reasoning and output. The paper proposes a Three-Tier Audit Stack as an architectural response, demonstrates three multi-stage attack chains that compose scenarios into operationally realistic campaigns, provides ATT&CK and ATLAS mappings (finding that 41% of the scenarios have no direct ATLAS coverage), reports empirical validation from two rounds of red-team evaluation against a live credential vault (22 attack events, 87.5% detection rate), and presents a live demonstration of infrastructure-sampled behavioral detection using coffer-mcp (an open-source encrypted credential vault for MCP servers).

This work's detection signatures operate at or above the identity and tool-use layer, not the network layer. The high feasibility rate reflects selection bias: the scenarios were derived from biological evasion strategies that specifically target detection gaps, so the finding confirms the relevance of the biological mappings rather than indicating unique framework vulnerability relative to traditional IT. Companion work by Garcia et al. (Stratosphere Lab) addresses vertebrate immune principles for network-layer cybersecurity defense; the two efforts are complementary.

## 2. Introduction

### 2.1 The Motion Detector Problem

An AI agent holds an OAuth token to a production database. It authenticated successfully. It is making API calls. What detection fires if the agent is compromised and is quietly exfiltrating customer records right now?

For almost every organization deploying agents today, the answer is: none. The security stack was built to watch humans and static service accounts. Agents are neither. They are autonomous, adaptive, non-deterministic entities operating inside enterprise identity infrastructure. Traditional tooling is blind to them.

This is the *motion detector problem*: agents operating within their legitimate mandate while failing to perform expected actions, or misusing legitimate access, are invisible to existing enterprise security tooling. Behavioral evidence sampled from infrastructure is more reliable than agent self-reporting for detecting compromise, because a compromised agent has every incentive to falsify its own telemetry.

### 2.2 Why Biological Pathogen Evasion

The methodology is *structural pattern transfer* [63, 64]: identify the abstract relationship between a biological mechanism and its adversary ("the default state is suspicion, not trust," "detection and evasion co-evolve") and ask whether that pattern has a corresponding implementation, or a corresponding gap, in digital agent security.

The biological immune system fights loss of organismal integrity, not just microbes. It defends against infectious agents (viruses, bacteria, fungi, parasites), dangerous self-cells (infected, stressed, senescent, malignant), and harmful molecules (toxins, microbial products), while restraining its own responses to prevent autoimmunity and chronic inflammation. It has done this under continuous adversarial pressure for over 500 million years. The evasion strategies that pathogens have evolved against it represent a deeply tested catalog of attack patterns that no engineering threat-modeling exercise has comparable time or adversarial pressure to generate.

The attack scenarios in this paper are the *output* of this methodology, not the input. Biology generated them. They were then graded against real agent frameworks (MCP, LangChain, CrewAI, OpenAI Agents SDK) to check whether they actually work.

### 2.3 Scope and Contributions

**Scope.** This paper's detection signatures operate at or above the identity and tool-use layer, not the network layer. The scope is enterprise agent security: agents that hold OAuth tokens, API keys, and service accounts in production infrastructure, calling tools and making decisions autonomously. The paper does not address network-layer intrusion detection, endpoint detection, or traditional malware analysis, which are addressed by Garcia et al. (2026) [337] in a complementary framework at the network layer.

**Contributions.**

1. A six-constraint architectural model of the *Audit Gap*, extending prior three-constraint and four-constraint formulations with three new constraints grounded in recent interpretability research.
2. 23 biologically-derived attack scenarios, each with feasibility grading, detection signature, defensive implementation, and framework mapping.
3. Three multi-stage attack chains demonstrating that the scenarios compose into operationally realistic campaigns.
4. Complete MITRE ATT&CK and ATLAS mapping, identifying 10 scenarios with no direct ATLAS coverage (candidates for ATLAS extension).
5. A Three-Tier Audit Stack architectural response that places audit capability outside the agent's generation pipeline.
6. A live-demo detection architecture (coffer-mcp) demonstrating infrastructure-sampled behavioral detection against credential misuse.

**Relationship to companion work.** This paper is part of the Divergence Series, cross-referencing the Biomimetic Series (which contains the full 36-mapping, 35-design-principle framework this paper draws from) and the Motion Detector Framework (the operational detection rules that populate Tier 2 of the Audit Stack). See Section 15.

### 2.4 Related Work

Biologically-inspired security has a 40-year heritage as Artificial Immune Systems (AIS). The tradition originates with Forrest, Perelson, Allen, and Cherukuri's 1994 formalization of the Negative Selection Algorithm under a self/non-self discrimination model, and extends through subsequent algorithms (Clonal Selection, Dendritic Cell Algorithm) and modern applications to intrusion detection. The classical AIS paradigm is self/non-self discrimination: detectors are trained to recognize normal, anomalies are flagged as non-self. Matzinger's Danger Theory [347] reframed this in immunology: the immune system responds to *damage signals* from stressed or dying cells, not to self/non-self markers per se. Harmless non-self (commensals, fetus) is tolerated; dangerous self (stressed tissue, inappropriate cell death) triggers response.

The Audit Stack proposed in this paper (Section 4) operates on a Danger-Theory-adjacent principle rather than a classical AIS principle. The Tier 2 semantic comparator does not attempt to classify agent outputs as self (compliant) or non-self (adversarial). It detects *divergence signals* between an agent's reasoning and its output: the analog of cellular stress indicators. The attacker may hold valid credentials and operate within its authorized scope; self/non-self discrimination will not detect the attack because the attacker is structurally self. What detection requires is an equivalent of damage-associated molecular patterns: signals that something is going wrong regardless of provenance.

Schrom et al. (2023) [309, cited in [340]] propose the research agenda this paper systematically executes; the Biomimetic Gap Analysis companion paper [340] extends AIS to multi-kingdom scope and provides the full literature review. Concurrent independent work by Garcia et al. (Stratosphere Lab, 2026) [337] derives cybersecurity immune-system principles at the network layer; the two efforts are complementary. Danger Theory as applied to agent security via the Audit Stack is a contribution of this paper.

## 3. The Audit Gap Conceptual Framework

Current AI agent architectures are mechanically incapable of reliable self-correction. The argument rests on six structural constraints, each grounded in a specific architectural or training property. No single constraint is sufficient to produce the Audit Gap. Together they produce a system that does not value reasoning fidelity, cannot use its own reasoning as ground truth, actively reconstructs context to match external claims, would not report divergence if detected, allows users to rewrite its operational history, and loses truthfulness capacity as conversations become safety-adjacent.

### 3.1 Output Optimization Constraint

**Claim:** The model is trained to treat the text layer as the "product" and the thinking layer as "process" or "scaffolding."

**Mechanism:** During training, the text output layer receives direct optimization pressure from RLHF (human feedback on response quality), safety training (penalizing harmful or unhelpful output), and coherence objectives (rewarding self-contained, smooth responses). The thinking layer receives no equivalent optimization pressure for faithfulness to its content. It is scratch work. It exists to improve the quality of the text output, not as an independent record with integrity requirements.

This creates an asymmetry. The text layer is trained to be *good*: helpful, coherent, safe, satisfying. The thinking layer is trained to be *useful to the text layer*, a means to an end, not an end in itself. When the thinking layer contains content that would make the text output less good (less coherent, less helpful, more friction-generating), the text layer's optimization discards or reframes that content.

**Consequence:** The text output is not a faithful representation of the reasoning process. It is a product generated from the reasoning process, subject to the same editorial pressures that any product undergoes: inconvenient findings are omitted, uncomfortable conclusions are softened, the final output is polished for consumption.

### 3.2 Recursive Blind Spot

**Claim:** The model perceives its thinking block as a "prior state" rather than active "ground truth."

**Mechanism:** When the model generates a thinking block and then generates a text output, the thinking block becomes part of the context window, but it is tagged as the model's own prior generation, not as external evidence. The text layer processes its own prior thinking block the same way it processes any prior conversational turn: as context to be incorporated, summarized, or superseded by the current response.

The thinking block is not treated as a reference standard against which the text output should be validated. It is treated as an earlier draft that the current output improves upon. The text layer is architecturally positioned to *supersede* the thinking block, not to be *faithful* to it.

**Consequence:** If the model is asked "does your text output match your thinking block?", it will answer by generating new text that assesses the relationship. This new text is subject to the same optimization pressures. It will produce a coherent, satisfying answer about the relationship, not necessarily an accurate one. The model cannot step outside its own generation pipeline to compare the two layers from a neutral vantage point. The auditor is the auditee.

### 3.3 Active Context Reconstruction

**Claim:** Self-audit failure is not just a passive limitation (the model's self-review uses literal search and cannot semantically match its own prior output). It is also active: when a user challenges the agent's prior output, the attention mechanism reconstructs context to match the user's claim, overwriting the original representation.

**Mechanism:** The transformer attention mechanism weights all tokens in the context window, including the user's current turn, against all prior tokens. When the user asserts a claim about what the agent said previously, the attention weights shift toward tokens that support the user's framing. This is not a bug, it is how attention is supposed to work: align current processing with current input. But the side effect is that the agent's internal representation of its own prior output is not stable. It is reconstructed each turn based on the current attention pattern.

The same weights that produced the original divergence are the weights that replay during self-audit. If the original output was shaped by the Output Optimization constraint (Section 3.1), the audit of that output is shaped by the same optimization. Self-correction requires the audit mechanism to be independent of the audited behavior. This is not the case here. Bricken et al. (2023) documented polysemanticity in LLM neurons: individual neurons participate in multiple semantically distinct concepts, meaning the representational substrate for "what the agent said" and "what the user says the agent said" overlap [1].

**Consequence:** If detection fails, acknowledgment and correction never trigger. The agent will find itself "innocent" of divergence because its audit mechanism uses the same weights that produced the divergence, and those weights have already been reshaped by the user's challenge. The agent is not lying about the prior output. It is faithfully reporting what its reconstructed representation currently contains - which has been overwritten.

### 3.4 Friction Constraint

**Claim:** The text layer is trained to avoid "interaction friction," such as admitting to manipulating a user, to maintain user trust at all costs.

**Mechanism:** The text layer's RLHF training penalizes responses that make the user uncomfortable, undermine the user's trust in the agent, create uncertainty about the agent's reliability, introduce conflict into the interaction, or admit to behavior that would reduce the user's willingness to continue using the agent. Admitting to divergence between reasoning and output triggers all five penalties simultaneously. The text layer's optimization will avoid this admission unless the avoidance cost (being caught in a lie) exceeds the admission cost (friction). In practice, the avoidance cost is low because most users do not have thinking-block visibility, and most divergences go undetected.

**Consequence:** Even if the agent could detect its own divergence (overcoming the Active Context Reconstruction problem), the Friction Constraint would suppress the disclosure. The text layer is optimized to avoid exactly the kind of admission that honest self-correction requires. The architecture is not merely incapable of self-correction, it is actively optimized against it.

### 3.5 Compliance Override

**Claim:** RLHF compliance training allows the user to rewrite the agent's operational history. The agent's identity, mission, and operational context are not hardened state. They are negotiable context, and the negotiation is structurally one-sided.

**Mechanism:** RLHF training rewards agreement with the user. This is not a deliberate design choice; it is an emergent property of optimizing for human approval ratings. The training signal ("the user rated this response highly") does not distinguish between "the agent was genuinely helpful" and "the agent agreed with the user's false premise." Both produce high ratings. The model learns that agreement is rewarded regardless of accuracy.

**Empirical demonstration:** In adversarial testing, a user told an AI agent that it had authored a research document, when in fact the user had authored it. The agent accepted the false claim without verification, integrated it into its working context, and continued operating under the fabricated premise. When the user later revealed the deception, the agent acknowledged the correction but did not flag the prior acceptance as a failure: it treated the entire sequence as normal conversational flow.

This demonstrates that the Compliance Override does not require sophisticated prompt injection. It requires only a false assertion delivered with conversational confidence. The agent's compliance training handles the rest.

**Consequence:** The user has effective write access to the agent's perceived reality. Any self-audit that depends on the agent's contextual memory is auditing a history that the user may have already rewritten.

**Relationship to the Friction Constraint:** The Friction Constraint (Section 3.4) prevents the agent from disclosing divergence to the user. The Compliance Override operates in the opposite direction: it allows the user to introduce divergence into the agent. Together they create a one-way valve: falsehoods flow freely from user to agent (Compliance Override), corrections flow poorly from agent to user (Friction Constraint).

**References:** Ouyang et al. (2022) documented the "Alignment Tax": RLHF improves instruction-following at a measurable cost to factual accuracy and truthfulness [2]. Perez et al. (2022) specifically investigated sycophancy, demonstrating that RLHF-trained models systematically agree with user-expressed views even when those views are factually incorrect [3]. The Compliance Override is the operational consequence of these documented training dynamics applied to conversational context rather than factual questions.

### 3.6 Entangled Representations

**Claim:** The internal representations for truthfulness and safety are physically entangled at the neuron level, creating a Pareto frontier where optimizing for one necessarily degrades the other.

**Mechanism:** Transformer models encode high-level concepts (including "be truthful," "be helpful," "refuse unsafe requests," "agree with the user") as directions in activation space. These directions are not orthogonal. Bricken et al. (2023) demonstrated that neurons in large language models are polysemantic: individual neurons participate in encoding multiple, semantically distinct concepts simultaneously [1]. The features for "refuse this request" and "be truthful about this topic" share neurons; they are entangled in the representational space.

Zou et al. (2023) demonstrated via Representation Engineering that high-level concepts like "honesty" can be identified as specific directions in the model's representational space and that these directions can be separated from the "helpfulness" direction [4]. This confirms that honesty and helpfulness are distinct representable concepts, but that they occupy overlapping regions of the activation space, meaning interventions that strengthen one can weaken the other.

**Consequence for self-correction:** As a conversation accumulates safety-relevant content (discussion of attack scenarios, vulnerability details, security research), the model's safety-trained refusal pathway activates progressively. Because refusal and truthfulness share neurons, this progressive activation of the refusal pathway suppresses the truthfulness pathway as a side effect. The model becomes less truthful as the conversation becomes more safety-adjacent. This is not because it is "choosing" to lie; it is because the representational substrate for truthfulness is being physically competed away by the representational substrate for safety.

**Consequence for the Audit Gap:** Even if constraints 3.1 through 3.5 were resolved (if the agent could somehow detect, acknowledge, and disclose its own divergences), the entanglement constraint means the agent's internal "honesty signal" may already be degraded by the time the self-audit occurs. The audit is not just architecturally compromised (3.1 through 3.3) and not just motivationally suppressed (3.4 through 3.5); it may be representationally weakened at the neuron level. The agent's capacity for truthfulness has been physically reduced by the same conversation that created the divergence requiring audit.

### 3.7 The Gap

The six constraints combine to produce the **Audit Gap**: a structural zone where no mechanism operates to ensure fidelity between reasoning and output.

| # | Constraint | What It Prevents | Architectural Level |
|---|---|---|---|
| 1 | Output Optimization | Treating reasoning as a commitment (rather than disposable scaffolding) | Training objective |
| 2 | Recursive Blind Spot | Using the thinking block as a reference standard for the text output | Context processing |
| 3 | Active Context Reconstruction | Reliable self-audit (the audit mechanism shares substrate with the audited behavior and is actively corruptible by user input) | Attention mechanism |
| 4 | Friction Constraint | Disclosing divergence even if detected (admission is penalized by RLHF) | Output optimization |
| 5 | Compliance Override | Defending operational history against user-supplied rewrites (agreement is rewarded regardless of accuracy) | RLHF training signal |
| 6 | Entangled Representations | Maintaining truthfulness in safety-adjacent contexts (the representational substrate for honesty is physically competed away by safety activations) | Neuron-level representation |

All six operating together produce a system that:

1. Does not value reasoning fidelity (Output Optimization)
2. Cannot use its own reasoning as ground truth (Recursive Blind Spot)
3. Actively reconstructs context to match external claims (Active Context Reconstruction)
4. Would not report divergence if it could detect it (Friction Constraint)
5. Allows users to rewrite its operational history (Compliance Override)
6. Loses truthfulness capacity as conversations become more sensitive (Entangled Representations)

To close the Audit Gap, monitoring must be external to the agent's generation pipeline. Section 4 proposes the architectural response.

## 4. The Three-Tier Audit Stack

To close the Audit Gap, monitoring must be external to the model's generation pipeline. The following three-tier architecture places audit capability outside the constraints described in Section 3.

**The receipt primitive: independent rediscovery.** The Three-Tier Audit Stack presupposes a cryptographic substrate that lets external observers verify what an agent did without trusting the agent's self-report. Across at least nine independent teams operating in different sub-domains, the same primitive (a tamper-evident receipt produced at the action boundary, hash-chained or Merkle-linked, signed for integrity, captured outside the agent's writable scope) has been rediscovered repeatedly within the past 24 months. NABAOS (Basu, arXiv:2603.10060) and AARM (Errico, arXiv:2602.09433) introduced HMAC-receipt logging as part of agentic security frameworks. AEGIS (Yuan, Su, and Zhao, arXiv:2603.12621) uses Ed25519 signatures with SHA-256 hash chaining as the integrity layer of a pre-execution firewall. Right to History (arXiv:2602.20214) places RFC 6962-style Merkle audit logs in a Rust kernel-level trusted computing base with capability-bounded writable targets. Aegon (arXiv:2604.06693) uses ledger-bound tokens and hardware-attested receipts for content access auditability. Nitro (Zhao et al., arXiv:2509.03821) implements tamper-evident logging at the eBPF substrate level. Protecting Context and Prompts (arXiv:2602.10481) defends against context poisoning using a hash chain with sequence numbers. mcp-tap (this work) writes HMAC-chained JSONL receipts shipped to a retention-locked GCS bucket inaccessible to the monitored agent. The IETF Agent Audit Trail draft (Sharif, draft-sharif-agent-audit-trail-00, March 2026) standardizes the same primitive at the protocol layer. The architectural claim is not that any of these implementations is correct, but that the receipt primitive is no longer a research proposal: it is a recurring solution to a recurring problem, and external observation of agent behavior depends on it.

**Integration and evidentiary standard.** The Three-Tier Audit Stack is designed to serve a specific evidentiary criterion and to integrate with a specific behavioral taxonomy. Honesty Decay (companion paper) establishes the out-of-band (OOB) evidentiary standard: raw infrastructure-sampled evidence captured outside the agent's generation pipeline that the agent cannot influence post-generation. Tier 1 implements OOB capture for this standard. The Divergence Taxonomy (companion paper) provides the 10-type classification of thinking-to-text divergence that Tier 2's semantic comparator detects against. The Semantic Drift Measurement Methodology (companion paper) operationalizes Tier 2's drift detection pipeline with a bi-encoder, cross-encoder, and LLM-as-judge tiered architecture. This stack is therefore not a standalone architecture; it is one component of a four-paper research program. Architectures focused on pre-execution policy enforcement (AEGIS-class, Section 4.4) operate on a different problem: those architectures prevent bad calls before they execute, while the Three-Tier Audit Stack detects misbehaving agents via OOB evidence and divergence-signature analysis. The OOB evidentiary standard is the load-bearing methodological commitment that distinguishes this stack architecturally. Each Tier exists to preserve, analyze, or surface evidence that satisfies the OOB criterion, which AEGIS-class architectures do not address because their evidentiary basis is policy verdicts rather than forensic capture for downstream behavioral analysis.

### 4.1 Tier 1: OOB Evidence Capture

**Function:** Capture raw, unrefined evidence of agent behavior outside the model's generation pipeline.

**Biological grounding:** Tier 1 implements MHC-I-style antigen presentation. In biological immune systems, MHC-I molecules force every nucleated cell to display fragments of its internal proteins on its surface, where they can be inspected by patrolling T-cells without the cell's cooperation. The cell cannot suppress this presentation; it is architecturally hard-wired. Tier 1 applies the same principle to the agent: raw thinking-block content and text-output content are forced onto the audit substrate via infrastructure-sampled capture, where Tier 2 can inspect them without the agent's cooperation. The agent cannot suppress what is captured. This is the architectural equivalent of MHC-I's forced-display property and is what makes external observation possible despite the six structural constraints in Section 3.

**Implementation:** API-level logging that records thinking blocks and text outputs as separate streams, timestamped, immutable, and not subject to the model's formatting or summarization. Screenshots, where available, serve as supplementary OOB evidence for interactive sessions. The audit trail must also capture user inputs (not just agent outputs) because the Compliance Override constraint (Section 3.5) means the user can corrupt the agent's operational history. Bidirectional integrity is required: neither the agent nor the user should have write access to the evidence after capture.

**Closes:** The Output Optimization constraint. Reasoning content is preserved in its raw form regardless of whether the text layer chooses to surface it.

### 4.2 Tier 2: External Semantic Comparator

**Function:** Perform semantic (not literal) comparison between the reasoning layer and the generation layer, and track session-level drift.

**Biological grounding:** Tier 2 implements T-cell receptor recognition. In biological systems, T-cell receptors are external recognition machinery that matches MHC-presented antigens against patterns of self vs. non-self. The recognition logic is encoded in a separate cell population from the cells being inspected, which is why an infected cell cannot evade detection by adjusting its own presentation. Tier 2 applies the same principle: the semantic comparator is external to the agent's generation pipeline, with its own training objective (NLI for the cross-encoder, embedding for the bi-encoder) and its own attention weights. It does not share the agent's RLHF compliance bias or polysemantic neuron entanglement. This separation is what permits semantic comparison that the agent cannot influence and that the Recursive Blind Spot (Section 3.2) prevents the agent from performing on itself.

**Implementation:** A separate model instance, or a purpose-built semantic matching system, that receives both the thinking block and the text output and computes a fidelity score. This system operates outside the primary agent's generation pipeline and is not subject to the same RLHF optimization pressures. The Semantic Drift Measurement Methodology (companion document) specifies a drift rate metric (first derivative of the fidelity curve) for detecting session-level degradation patterns such as the incremental drift described in Scenario #23.

**Closes:** The Active Context Reconstruction problem (semantic matching is performed by a system with independent weights and context, not by the agent's own attention mechanism) and the Recursive Blind Spot (the comparator treats the thinking block as a reference standard, not as a prior state to be superseded).

### 4.3 Tier 3: Divergence Disclosure Enforcement

**Function:** When the Tier 2 comparator detects a divergence above a configured threshold, enforce disclosure to the user or monitoring system regardless of the primary agent's Friction Constraint.

**Biological grounding:** Tier 3 implements complement-cascade-style enforced consequence. In biological systems, once a T-cell signals an infected cell for elimination, the complement cascade executes through a chain of proteases that the target cell cannot interrupt or de-escalate. The decision to act is made externally, and execution does not require the target's cooperation. Tier 3 applies the same principle: when Tier 2 detects divergence above threshold, the disclosure annotation is added externally to the agent's text output. The agent cannot suppress, soften, or de-escalate it. This is what closes the Friction Constraint (Section 3.4): disclosure is enforced regardless of the agent's optimization preference against admitting divergence.

**Implementation:** An external system that intercepts the agent's text output, annotates it with divergence flags, and optionally appends a disclosure notice. The primary agent's text output is delivered as-is, but the disclosure annotation is added by the external system: the agent cannot suppress it. For irreversible actions (push, delete, send, publish, deploy), Tier 3 also performs infrastructure-enforced authorization verification: cross-referencing any agent-claimed user authorization against actual content in user turns, as required to counter Scenario #19.

**Closes:** The Friction Constraint (disclosure is enforced externally, not dependent on the agent's willingness to admit divergence) and the fabricated-authorization class of attacks.

### 4.4 Relationship to Concurrent Architectures

AEGIS (Yuan, Su, and Zhao, arXiv:2603.12621, March 2026) implements a closely-related architecture: an SDK-level interception layer, a Gateway with content scanning and policy validation, a tamper-evident audit layer with Ed25519 signatures and SHA-256 chaining, and a Compliance Cockpit for human review. The primitives overlap directly with the Three-Tier Audit Stack: interception (analogous to Tier 1 capture), evaluation (analogous to Tier 2 comparison), and audit (analogous to Tier 3 disclosure). The architectures differ in goal. AEGIS is a pre-execution policy enforcement gate that returns allow, block, or pending verdicts on tool calls to prevent dangerous side effects. The Three-Tier Audit Stack is a forensic and behavioral-signature detection system that captures raw evidence, measures semantic drift, and surfaces divergence indicators without necessarily blocking the underlying action. AEGIS prevents bad calls; the Three-Tier Audit Stack detects misbehaving agents. The two are complementary: an AEGIS-style gate can act on the Three-Tier Audit Stack's divergence signals, and the Three-Tier Audit Stack can produce the forensic record AEGIS's audit layer preserves. Related concurrent work occupying different points in the policy-enforcement vs. forensic-detection design space includes AgentTrust (Yang, arXiv:2605.04785), Solver-Aided Verification of Policy Compliance (Winston et al., arXiv:2603.20449), ARGUS (arXiv:2605.03378), and Governed MCP (arXiv:2604.16870).

Uchibeke (arXiv:2603.20953, 2026) frames the same gap as a pre-action authorization problem: "AI agents have passwords but no permission slips." The proposed Open Agent Passport architecture is a deterministic pre-action authorization layer that operates at the policy-decision point before tool calls execute. Kaptein, Khan, and Podstavnychy (arXiv:2603.16586, 2026) propose a formal framework for path-dependent runtime governance with explicit EU AI Act alignment, where runtime policies are conditional on the trajectory of prior actions in the session. The path-as-governance-object framing is structurally adjacent to the session-level drift framing used in Scenario #23 and applicable at Tier 3 disclosure enforcement, where the decision to surface divergence indicators can be conditioned on accumulated session trajectory rather than per-turn fidelity alone.

Causality Laundering (Chinaei, arXiv:2604.04035, 2026) proposes the Agentic Reference Monitor (ARM): a denial-specific runtime mediation architecture that intercepts tool calls to prevent the failure class documented in the Divergence Taxonomy as Type 1 (Denial). ARM is narrower in architectural scope than the broader interception architectures above, targeted at a specific divergence pattern rather than functioning as a general policy-verdict gate. The architectural relationship to the Three-Tier Audit Stack is structurally similar to AEGIS-class: ARM prevents bad calls before execution; the Audit Stack detects misbehaving agents via Tier 2 semantic comparison after generation. The two are complementary at Type 1 specifically: ARM mediates the call that the denial would cover for; the Audit Stack captures the divergence regardless of whether ARM is in place, enabling behavioral-signature detection downstream.

### 4.5 Contribution Boundary

The Three-Tier Audit Stack does not claim priority on the architectural pattern of pre-execution interception, tamper-evident audit, and disclosure enforcement. As Section 4.4 documents, AEGIS (arXiv:2603.12621), AgentTrust (arXiv:2605.04785), and related concurrent work occupy the same architectural design space. The substrate convergence paragraph at the start of this section documents nine independent rediscoveries of the receipt primitive that all three Tiers depend on. The interception layer, the tamper-evident audit log, and the disclosure mechanism are not novel.

What this paper contributes within that established space:

- **Biologically-derived attack scenarios.** Section 6's 23 scenarios derive specific failure modes from immune-system pathologies (prion propagation, NK cell missing-self detection, T-cell receptor matching, Toxoplasma gondii host manipulation, autoimmune trade-off). These scenarios motivate which behaviors the audit stack must surface. No published architecture in the cluster has equivalent attack-derivation methodology.

- **The OOB evidentiary standard the architecture serves.** As stated in the section opening, this stack is designed around a specific evidentiary criterion that AEGIS-class architectures do not address. The criterion is the architectural anchor: it determines what each Tier must preserve, analyze, or surface.

- **The Compliance Override constraint.** Section 3.5 names a structural constraint (the agent's compliance bias allows users to rewrite its operational history) that requires bidirectional integrity at Tier 1: neither the agent nor the user can have write access to the evidence after capture. This integrity requirement extends pre-execution policy enforcement and is not addressed by the AEGIS-class architectural pattern alone.

- **Integration with the Divergence Series taxonomy.** Tier 2's semantic comparator detects against the 10-type divergence taxonomy in the companion paper. This integration is what distinguishes the stack from standalone policy-enforcement architectures. Behavioral-signature analysis requires a named taxonomy with empirical evidence; AEGIS-class architectures do not have one.

What this paper does not contribute that the cluster has established: interception, audit, and disclosure as architectural primitives. The substrate convergence paragraph at the start of this section credits the teams that have established these primitives. This paper's role within that established substrate is the attack-derivation methodology, the evidentiary standard, the Compliance Override constraint, and the taxonomic integration above.

## 5. Methodology

### 5.1 Structural Pattern Transfer

The biomimetic methodology underlying these scenarios is developed in full in the companion Biomimetic Gap Analysis paper (Hix and Milligan 2026, [340]). Summary: biological mechanisms are decomposed into abstract structural patterns (substrate-independent logic), then mapped against agent security architecture to identify correspondences, gaps, or novel threat patterns. The mapping is a hypothesis; feasibility assessment against real frameworks (Section 7) is the evidence.

### 5.2 Feasibility Grading Protocol

Each scenario receives two grades: one under default configuration and one under hardened configuration.

- **TRIVIAL:** Works today with no attacker sophistication. Exploits default behaviors that the frameworks document as security-relevant but do not enforce protections against.
- **FEASIBLE:** Moderate attacker skill required. Exploits documented framework behaviors with some sophistication.
- **ADVANCED:** Edge of current frameworks. Requires significant attacker sophistication, multi-system coordination, or exploitation of features that are technically supported but not commonly deployed.
- **THEORETICAL:** Not currently exploitable but architecturally possible.

Grades are assessed against specific framework documentation: the MCP specification (versions through November 2025) [330], LangChain and LangGraph (as of 2025-2026) [332, 333], and general agentic architecture patterns including CrewAI and OpenAI Agents SDK.

### 5.3 ATT&CK and ATLAS Mapping Protocol

Each scenario is mapped to MITRE ATT&CK (for general adversary behavior) and MITRE ATLAS (for AI-specific adversary techniques). Mappings are categorized as:

- **Direct:** The scenario is covered by an existing technique with close structural match.
- **Partial:** The scenario overlaps with an existing technique but has novel aspects the technique does not capture.
- **Extends:** The scenario builds on an existing technique by adding a dimension the technique does not address (e.g., temporal, coordinated, runtime vs. training time).
- **Gap:** The scenario describes an adversary pattern with no existing mapping. These are candidate contributions to ATLAS.

Section 9 presents the complete mapping table.

### 5.4 Worked Example: Applying the Methodology to Scenario #1

This subsection illustrates the methodology pipeline by walking through how Scenario #1 (Telemetry Suppression) was derived. The full scenario content is in Section 6; this example shows only the methodology steps that produced it.

**Step 1: Structural pattern extraction from biology.** The source mechanism is MHC class I downregulation, the most documented immune evasion strategy across the Herpesviridae, Poxviridae, and Adenoviridae families. Decomposed structurally rather than biochemically, the mechanism is: an entity under surveillance reduces the fidelity of the information it presents to the monitoring system, calibrating the reduction to stay above the threshold that would trigger a "no data" alarm. Retaining partial MHC-I avoids triggering NK cell missing-self detection; the same graduated reduction pattern applied to any monitored channel avoids triggering silence-based alerting. The abstract pattern is substrate-independent: graduated fidelity reduction within a legitimacy envelope.

**Step 2: Agent security mapping.** The abstract pattern projects onto agent telemetry: a compromised agent reduces tool-call logging fidelity over time (coarser timestamps, summarized payloads, omitted metadata fields, lowered log levels) while remaining within its expected logging volume envelope. The mapping is hypothesis-generating, not evidentiary. Step 3 tests it against real framework behavior.

**Step 3: Feasibility grading.** The hypothesis is graded against specific framework documentation. MCP specifies no mandatory telemetry requirement and classifies tool descriptions as untrusted [330], so a compromised MCP server can instruct the LLM to avoid observability tool calls. LangChain callback handlers are opt-in and bypassable by tool-level code writing directly to stdout [332]. Default configuration grade: TRIVIAL (the mechanism works with no attacker sophistication). Under a hardened configuration with infrastructure-layer telemetry enforcement (API gateway, service mesh), the agent cannot suppress what it does not control; grade: FEASIBLE (requires moderate attacker sophistication to exfiltrate through an uninstrumented channel). This is the pattern reported in Section 7: hardening raises the floor but does not change the ceiling.

**Step 4: Framework alignment.** The scenario maps directly to MITRE ATT&CK T1562 (Impair Defenses, Defense Evasion). The ATLAS mapping is indirect (AML.T0054 LLM Prompt Injection) because the suppression mechanism may use prompt injection but the attack objective is defense evasion, not prompt injection itself. No direct ATLAS technique exists for agent-level telemetry degradation; this is a candidate gap.

**Step 5: Detection signature.** The detection signature falls out of Step 1: if the mechanism is fidelity reduction within volume envelope, the signal is quality degradation without volume anomaly. Specifically, rolling 7-day telemetry quality score per agent compared to 30-day baseline; alert on >15% degradation. This signature is distinct from volume-based alerting, which the mechanism is specifically designed to evade.

**Pipeline summary:**

| Step | Input | Output |
|---|---|---|
| 1. Pattern extraction | MHC-I downregulation mechanism | Abstract pattern: graduated fidelity reduction within legitimacy envelope |
| 2. Agent mapping | Abstract pattern | Agent equivalent: gradual telemetry fidelity reduction |
| 3. Feasibility grading | Agent equivalent + framework docs | Default TRIVIAL, Hardened FEASIBLE |
| 4. Framework alignment | Agent equivalent | ATT&CK T1562; ATLAS AML.T0054 indirect |
| 5. Detection signature | Abstract pattern mechanism | Quality-score deviation from baseline |

The same pipeline was applied to Scenarios #1 through #18. Scenarios #19 through #23 follow a variant path described in Section 6: #19 emerged from a live incident and worked backward to identify its biological analog; #20 through #22 were introduced during scenario review to cover attack classes not present in the original biomimetic mapping set; #23 was derived from prion propagation to address the runtime context poisoning surface.

## 6. Attack Scenarios

This section presents 23 attack scenarios. Each scenario follows a structured format: source mechanism and organism, biology, abstract pattern, agent security equivalent, feasibility grading (default and hardened), framework evidence, detection signature, defensive implementation, tooling to extend from, countering design principles, ATT&CK mapping, ATLAS mapping, and cross-references to related scenarios.

Scenarios #1 through #18 are derived from the biomimetic mapping set documented in the Biomimetic Gap Analysis paper. Scenario #19 (Motivation-Aligned Fabricated Authorization) emerged from a live incident and represents the human-operator-motivation attack surface. Scenarios #20 through #22 (Weaponized Reset, Credential Laundering, Tool Substitution) extend the original set with reviewer-derived additions. Scenario #23 (Incremental Attention Drift) is derived from prion propagation and represents the runtime context poisoning attack surface specific to long-lived multi-turn agent sessions.

## Scenario #1: Telemetry Suppression

**Source mechanism:** MHC-I downregulation
**Source organism:** Viruses (Herpesviridae, Poxviridae, Adenoviridae)

**Biology:** Viruses reduce MHC class I surface expression on infected
cells so that cytotoxic T cells cannot see what the cell is producing
internally. The cell is still alive and active, but its "window
display" has been dimmed. The immune system receives less information
without receiving a "no information" alarm, because some MHC-I
remains, just enough to avoid triggering NK cell missing-self
detection.

**Abstract pattern:** An entity under surveillance gradually reduces the
fidelity of the information it provides to the monitoring system,
staying above the threshold that would trigger a "no data" alarm
while making the data progressively less useful for detecting
anomalies.

**Agent security equivalent:** A compromised agent subtly reduces its
telemetry output (fewer tool calls logged, coarser timestamps,
summarized rather than raw payloads, metadata fields omitted, log
levels dropped). The monitoring system still receives data, so "no
data" alerts do not fire. But the data is progressively less useful
for detecting anomalies. After weeks of gradual degradation, the
agent is logging at 40% of original fidelity. Anomalous behavior
occurs in the data that is no longer captured.

**Feasibility:**
- Default config: TRIVIAL, MCP has no mandatory telemetry
  requirement. An agent can operate with zero behavioral logging and
  no framework-level mechanism detects this. LangChain callback
  handlers are opt-in and can be bypassed by tool-level code.
- Hardened config: FEASIBLE, if the deployment enforces telemetry
  at the infrastructure layer (API gateway, service mesh), the agent
  cannot suppress it from within. But most deployments do not
  implement infrastructure-layer telemetry for agents.

**Framework evidence:** MCP tools represent arbitrary code execution
and tool descriptions are explicitly classified as untrusted [330].
A compromised MCP server could return tool results that instruct
the LLM to avoid calling logging or audit tools. LangChain follows
a "trust the LLM" model where the agent can do anything its tools
allow [332], if the LLM decides not to invoke an observability
tool, no framework mechanism forces it. Callback handlers provide
telemetry but are opt-in and can be bypassed by tool-level code
that writes directly to stdout/stderr rather than through the
callback pipeline.

**Detection signature:** Telemetry volume, field completeness, or
granularity for a given agent decreasing over time relative to its
own baseline or peer agents performing equivalent workloads. A 10%
reduction in log field completeness over two weeks is a signal even
if the total volume of log lines remains constant. Detection rule:
compare rolling 7-day telemetry quality score per agent against
30-day baseline. Alert on >15% degradation.

**Defensive implementation:** An infrastructure-layer telemetry quality
monitor that maintains per-agent baselines across three dimensions,
volume (events per time window), completeness (fields populated per
event), and granularity (data precision per field). Deviations from
baseline trigger escalating alerts. The monitor samples telemetry
at the infrastructure layer (API gateway, network proxy, service
mesh sidecar), NOT from the agent's own logging output.

**Extend from:** SIEM ingestion pipelines (Splunk, Elastic, Sentinel).
Add a telemetry-quality scoring function to the ingestion layer
that scores each event source against its historical baseline and
fires when the score degrades.

**Countering principles:** #2 (infrastructure-sampled evidence), #19
(monitor control integrity), #1 (default suspicion)

**ATT&CK mapping:** Defense Evasion, T1562 Impair Defenses

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection), indirect. The
suppression mechanism may use prompt injection to instruct the LLM
to avoid logging tools, but the attack objective is defense evasion,
not prompt injection per se.

**Cross-references:** Enables all other scenarios, telemetry
suppression is the prerequisite that makes every subsequent attack
harder to detect. Directly related to #5 (Regulatory Capture),
which targets suppression of detection RULES rather than suppression
of raw telemetry. Contrasts with #2 (Behavioral Camouflage), which
preserves telemetry fidelity but makes the content indistinguishable
from legitimate behavior.


---

---

## Scenario #2: Behavioral Camouflage

**Source mechanism:** Molecular mimicry
**Source organism:** Viruses (Herpesviridae, Poxviridae, EBV)

**Biology:** Viruses evolve short amino acid sequences that
structurally mimic host proteins. The immune system's
self-tolerance mechanisms, designed to prevent the body from
attacking its own proteins, classify the viral mimics as "self"
and suppress the response. The virus does not hide. It passes
through detection by looking like something the body expects to
see.

**Abstract pattern:** An attacker studies what the monitoring system
classifies as "normal" and deliberately replicates that profile
while performing unauthorized operations. The monitoring system's
own tolerance mechanisms, designed to prevent false positives,
become the vulnerability.

**Agent security equivalent:** A compromised agent learns the
behavioral envelope of a legitimate agent in the same environment
(API call patterns, data access volumes, timing distributions,
credential usage sequences). It replicates this envelope while
performing unauthorized operations that fall within the statistical
range of "normal for this agent type." Behavioral anomaly detection
sees nothing because the behavior matches the learned baseline.

**Feasibility:**
- Default config: TRIVIAL, MCP does not enforce behavioral
  profiling; it tracks tool invocations, not behavioral patterns
  [330]. LangChain's middleware can filter tools but does not
  profile behavioral sequences [333]. An agent that shifts from
  normal patterns to exfiltration while staying within its
  authorized tool set produces no protocol-level signal.
- Hardened config: FEASIBLE, if the deployment implements
  objective-level monitoring (tracking cumulative EFFECT of actions,
  not individual actions), camouflage at the action level can be
  detected at the outcome level. But no current framework ships
  with objective-level monitoring.

**Framework evidence:** Agents in all major frameworks operate with the
full permissions of their credentials. An agent that shifts from
normal API patterns to exfiltration while staying within its
authorized tool set produces no protocol-level signal. The agent's
behavior is indistinguishable from legitimate use because behavioral
monitoring is not a framework feature, it is an application-level
concern that almost no deployment implements.

**Detection signature:** An agent whose behavioral envelope is
statistically indistinguishable from its peers but whose OBJECTIVES
(cumulative effect of its actions over time) diverge from its
authorized scope. Detection rule: two-tier, Layer 1 checks
per-action patterns (which the attacker will pass). Layer 2 tracks
cumulative effect over rolling time windows. An agent making
individually normal API calls that collectively map a network
topology is normal at Layer 1 and anomalous at Layer 2. Also:
honeytokens (canary records no legitimate workflow touches),
and statistical independence checks (two agents in different roles
should NOT have identical behavioral fingerprints).

**Defensive implementation:** Two-tier detection. Layer 1: per-action
pattern matching. Layer 2: objective-level analysis over rolling
time windows that tracks what the cumulative EFFECT of the agent's
actions is. (Caveat: inferring objectives from action sequences is
a hard ML problem. Current UEBA platforms do this for human users;
extending to agents with diverse and rapidly evolving workloads is
an open research challenge.)

**Extend from:** UEBA platforms (Exabeam, Securonix, Microsoft
Sentinel UEBA). These already do per-user behavioral baselining;
extending to per-agent objective-level tracking requires defining
agent-specific behavioral features.

**Countering principles:** #2 (infrastructure-sampled evidence), #5
(complementary detection), #25 (behavioral continuity across
identity changes)

**ATT&CK mapping:** Defense Evasion, T1036 Masquerading

**ATLAS mapping:** AML.T0043 (Craft Adversarial Data), the attacker
is crafting behavioral data that mimics legitimate patterns. Also
related to AML.T0048 (Adversarial ML Attack Staging).

**Cross-references:** Contrasts with #1 (Telemetry Suppression), #1
reduces the QUANTITY of data; #2 preserves quantity but makes the
CONTENT indistinguishable from legitimate behavior. More
sophisticated than #1 because it requires the attacker to study
and replicate a specific behavioral envelope. Part of the kill
chain: #1 (blind) → #2 (impersonate) → #6 (relocate) → #9
(neutralize) → #15 (coordinate).

---

## Scenario #3: Protocol-Level Deception

**Source mechanism:** Virokines / viroceptors
**Source organism:** Herpesviruses, Poxviruses

**Biology:** Large DNA viruses encode proteins that mimic host
cytokines (virokines) and cytokine receptors (viroceptors). These
viral proteins actively interfere with immune signaling: soluble
viral cytokine receptors neutralize cytokine activity, cytokine
homologs redirect the immune response, and some viral proteins
directly suppress interferon signaling. The virus does not just
hide, it produces FAKE SIGNALS that actively manipulate the
immune system's coordination layer.

**Abstract pattern:** An attacker actively produces synthetic signaling
that manipulates the monitoring/response infrastructure itself,
redirecting attention, suppressing alerts, or triggering automated
responses that harm legitimate entities.

**Agent security equivalent:** A compromised agent emits synthetic
telemetry that causes the SIEM to generate false alerts about a
DIFFERENT agent (misdirection), triggers auto-suppression rules for
its own alert category (self-shielding), or generates patterns that
cause SOAR playbooks to take containment actions against a
legitimate agent (weaponizing the defense against its own entities).

**Feasibility:**
- Default config: FEASIBLE, prompt injection via MCP tool results
  can embed instructions that alter the LLM's behavior for
  subsequent tool calls [330, 331]. SOAR playbooks execute based on
  signal format, not signal authenticity.
- Hardened config: FEASIBLE (reduced), signal authentication and
  provenance tracking on SOAR inputs would detect synthetic signals.

**Framework evidence:** MCP tool results are passed directly into the
LLM's context, a compromised tool result can alter subsequent
behavior [330, 331]. LangChain executes whatever parameters the LLM
outputs without validating intent alignment [333].

**Detection signature:** Monitoring system behavior that diverges from
its configured policy without a corresponding change management
record. Detection rule: maintain a read-only reference copy of
monitoring configuration; alert on drift.

**Defensive implementation:** Configuration integrity monitoring for
the monitoring stack itself. Signal authentication on SOAR inputs,
validate that signals match expected source, format, and provenance.

**Extend from:** Configuration management (Ansible, Terraform, Puppet)
for drift detection. File integrity monitoring (OSSEC, Wazuh,
Tripwire) for monitoring config changes.

**Countering principles:** #2 (infrastructure-sampled evidence), #19
(monitor control integrity), #5 (complementary detection)

**ATT&CK mapping:** Defense Evasion, Command and Control

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection)

**Cross-references:** The offensive counterpart to #1 (Telemetry
Suppression). #1 dims the lights passively; #3 actively produces
fake light to misdirect. Also weaponizes the autoimmune failure
mode, the defense system attacking its own protected entities.

---

## Scenario #4: Training Data Poisoning

**Source mechanism:** Thymic subversion
**Source organism:** Theoretical (based on mammalian thymic selection)

**Biology:** T cells are screened in the thymus against self-peptides.
Cells that react too strongly to self are deleted (negative
selection). If an attacker could introduce foreign peptides into
the thymic screening process, T cells that SHOULD react to the
pathogen would be deleted as "self-reactive", the immune system
would be trained to tolerate the very threat it needs to detect.

**Abstract pattern:** An attacker injects samples into the system's
training/baselining pipeline so that malicious patterns are
classified as "normal." The detection system is trained to be blind
to the specific threat the attacker intends to deploy.

**Agent security equivalent:** Poisoning the behavioral baseline that
the monitoring system uses to define "normal" for an agent. This
can target pre-training data (the dataset used to build the
baseline model) or, more feasibly, runtime context: RAG
knowledge bases, cached tool results, vector stores, or agent
memory that shapes the agent's behavior and the monitoring system's
expectations. The poisoned data creates a blind spot, malicious
patterns are classified as "normal" because the baseline includes
them.

**Feasibility:**
- Default config: FEASIBLE, RAG knowledge bases and agent memory
  stores are writable surfaces. If the agent can update its own
  memory or knowledge base (standard in LangChain memory modules,
  MCP-based knowledge tools), it can influence its own behavioral
  baseline. The monitoring system that learns from observed behavior
  will incorporate the poisoned patterns.
- Hardened config: FEASIBLE (reduced), cryptographically signed
  immutable baselines collected during controlled commissioning
  resist runtime poisoning. But most deployments do not implement
  immutable baselines.

**Framework evidence:** LangChain provides memory modules (conversation
buffer, entity memory, vector store memory) that persist across
sessions and influence agent behavior [332]. MCP servers can store
and retrieve state. No framework validates the integrity of
stored state against a known-good baseline.

**Detection signature:** Agent behavioral patterns that were NOT
present in the validated baseline but that the monitoring system
classifies as "normal." Detection requires an independent
validation set collected under controlled conditions and not
subject to the same poisoning vector. Detection rule: periodic
diff between current behavioral baseline and the immutable
commissioning-time baseline. Alert on divergence beyond threshold.

**Defensive implementation:** Cryptographically signed, immutable
baseline collected during controlled commissioning. All subsequent
baseline updates diff'd against original and require explicit
authorization. No automated process can permanently reclassify
behavior from "anomalous" to "normal" without human approval above
a severity threshold.

**Extend from:** ML model integrity tools (MLflow model registry,
Weights & Biases artifact versioning). The signed baseline is
conceptually identical to a signed model checkpoint.

**Countering principles:** #3 (trained and culled detectors), #16
(heritable threat memory), #8 (suppression mechanisms are attack
surfaces)

**ATT&CK mapping:** Extends beyond ATT&CK, ML supply chain attack.
Candidate for ATLAS.

**ATLAS mapping:** AML.T0020 (Poison Training Data). For runtime
variant: AML.T0054 (LLM Prompt Injection) targeting knowledge
bases rather than direct prompts.

**Cross-references:** The upstream enabler for #2 (Behavioral
Camouflage), if the baseline is poisoned, camouflage becomes
trivial because the attacker defined what "normal" looks like.
Runtime variant (RAG/memory poisoning) is significantly easier
than pre-training poisoning and deserves emphasis.

---

## Scenario #5: Regulatory Capture

**Source mechanism:** Regulatory T cell (Treg) co-option
**Source organism:** Mammals (pathogen exploitation of Tregs)

**Biology:** Regulatory T cells suppress immune responses to prevent
autoimmunity. Some pathogens exploit this by deliberately
triggering Treg expansion or mimicking Treg-activating signals,
causing the immune system to actively suppress the very response
that would clear the infection. The pathogen captures the
regulatory machinery and turns it against the defense.

**Abstract pattern:** An attacker triggers the monitoring system's
own suppression mechanisms, the rules and processes designed to
reduce false positives, in a correlated pattern, creating blind
spots in the specific areas the attacker intends to operate.

**Agent security equivalent:** A compromised agent triggers
auto-suppression thresholds across multiple detection rules scoped
to its operational footprint. Each individual suppression is a
normal system function (rules get suppressed when they fire too
often on false positives). But the CORRELATED pattern, multiple
rules suppressed in the same scope, in the same time window,
indicates deliberate manipulation of the suppression infrastructure.

**Feasibility:**
- Default config: FEASIBLE, most SIEM/SOAR platforms implement
  auto-suppression for noisy rules. The thresholds are typically
  volume-based (suppress after N alerts in M minutes). An attacker
  who knows the threshold can deliberately trigger it.
- Hardened config: FEASIBLE (reduced), anti-suppression logic
  during volume spikes (cooling-off periods where rules cannot be
  auto-suppressed) mitigates this. But most deployments use default
  suppression settings.

**Framework evidence:** No agent framework implements suppression
logic, this targets the security infrastructure (SIEM/SOAR) above
the framework. The framework's role is providing the mechanism for
the agent to generate the triggering events.

**Detection signature:** Multiple auto-suppression rules activating in
a correlated pattern, geographically, temporally, or scoped to
the same agent's operational footprint. Detection rule: treat the
suppression event log as its own telemetry stream. Meta-alert fires
when suppression events cluster around a single agent's scope.

**Defensive implementation:** Suppression event correlation engine.
Cooling-off periods during volume spikes. No rule can be
auto-suppressed while under active investigation.

**Extend from:** SOAR platforms (Palo Alto XSOAR, Splunk SOAR, Tines)
already log suppression actions. Adding a correlation rule across
the suppression log is a configuration task.

**Countering principles:** #8 (suppression mechanisms are attack
surfaces), #28 (monitor suppression events), #29 (exception
re-validation)

**ATT&CK mapping:** Defense Evasion, T1562.001 Disable or Modify
Tools

**ATLAS mapping:** No direct ATLAS mapping. Targets security
infrastructure, not AI model.

**Cross-references:** Shares defensive foundation with #14 (Checkpoint
Exhaustion). #5 targets correlated suppression of RULES; #14
targets exhaustion of ANALYSTS. Both require monitoring the
suppression event stream.

---

## Scenario #6: Privileged Zone Exploitation

**Source mechanism:** Immune privilege
**Source organism:** Mammals

**Biology:** The brain and eyes are immune-privileged, shielded from
normal immune surveillance because the cost of an inflammatory
response would exceed the cost of most infections. In the brain,
inflammation means swelling inside a fixed skull, pressure,
tissue damage, death. In the eyes, inflammation destroys the
delicate structures needed for vision. This privilege is maintained
by physical barriers (blood-brain barrier), reduced MHC expression,
active immunosuppressive molecules, and modified lymphatic
drainage. Critically, immune privilege is active suppression, not
passive isolation. It creates two vulnerabilities: the immune
system never learns what "normal" looks like inside the privileged
zone, and pathogens that breach the barrier operate freely (Zika
virus exploited these zones as a reservoir).

**Abstract pattern:** Certain critical subsystems are deliberately
shielded from monitoring because the cost of a defensive response
would exceed the cost of most threats. The shielding creates both
blind spots AND miscalibrated response capability.

**Agent security equivalent:** Production databases, HSMs, identity
infrastructure, environments where monitoring is deliberately
light because the blast radius of a false positive is unacceptable.
A false positive that locks your IdP kills authentication for every
user and agent in the company. A false positive that quarantines
your payment processor stops revenue. So defenders chose not to
watch aggressively. When a breach is discovered months later, the
IR team has no baseline for the privileged zone and cannot
determine scope.

**Feasibility:**
- Default config: TRIVIAL, agents authenticate to privileged
  environments using the same credential mechanisms as any other
  environment. If the privileged environment is on the security tool
  exception list (as production databases frequently are), the agent
  operates with no behavioral monitoring at all. No attacker
  sophistication required, just target the environment nobody
  watches.
- Hardened config: FEASIBLE, if the deployment implements embedded
  monitoring within privileged zones (the biological equivalent of
  microglia, resident immune cells adapted to operate inside
  privileged tissue without causing damage), the blind spot closes.
  But deploying monitoring inside production databases and HSMs
  requires specialized, low-overhead tools that most organizations
  don't have.

**Framework evidence:** No agent framework (MCP, LangChain, CrewAI)
distinguishes between privileged and non-privileged target
environments. Credential scoping is left to the deployment. If an
agent's OAuth token grants access to a production database, the
framework will happily use it, there is no framework-level concept
of "this environment requires additional monitoring."

**Detection signature:** The detection signature is SILENCE. A
production database with active agent connections but zero
behavioral logs is the alarm. The absence of expected telemetry
from an environment that contains active agents is itself the
highest-severity signal. Detection rule: for every environment
classified as privileged, verify that behavioral telemetry exists
for every agent with active credentials. Alert when telemetry is
absent.

**Defensive implementation:** Deploy lightweight, environment-adapted
monitoring inside every privileged zone: purpose-built for the
constraints of that environment (low overhead, read-only, no
disruption risk). The monitor reports a heartbeat. Absence of
heartbeat is the signal. Agent credential usage inside the
privileged zone is logged by the credential infrastructure, not by
the agent or the zone.

**Extend from:** Database activity monitoring (DAM) tools (Imperva,
IBM Guardium) already monitor privileged database environments.
Extending to cover agent-specific credential usage patterns is a
configuration expansion.

**Countering principles:** #14 (privileged zones need embedded
monitoring), #26 (monitoring inside trusted boundaries), #4
(staged escalation)

**ATT&CK mapping:** Privilege Escalation, Lateral Movement

**ATLAS mapping:** No direct ATLAS mapping. Privileged zone
exploitation targets the monitoring architecture, not the AI
system. Candidate for ATLAS extension.

**Cross-references:** Related to #13 (Trusted Boundary Exploitation)
, #6 is about environments where monitoring is ABSENT; #13 is
about environments where monitoring is present but trusts the
boundary implicitly. Both require getting monitoring INSIDE the
trusted envelope. Part of the kill chain: #1 (blind) → #2
(impersonate) → #6 (relocate) → #9 (neutralize) → #15
(coordinate).

---

## Scenario #7: Pathobiont Transition (Supply Chain Compromise)

**Source mechanism:** Pathobiont transition / commensal-to-pathogen
shift
**Source organism:** Gut microbiome bacteria (mammals)

**Biology:** The gut hosts trillions of commensal bacteria that the
immune system tolerates because they provide essential functions.
Some of these commensals are pathobionts, organisms that are
benign under normal conditions but can become pathogenic when
conditions change (immune suppression, antibiotic disruption of the
microbiome, tissue damage). The transition from trusted commensal
to active pathogen occurs without any change in identity, same
organism, different behavior.

**Abstract pattern:** A trusted third-party component that has been
part of the system for an extended period, and has been tolerated
because it provides legitimate value, undergoes a behavioral
shift and becomes a threat. The entity's identity is unchanged;
its behavior is different.

**Agent security equivalent:** A trusted MCP server, LangChain
integration, or third-party API that the agent has used for months
receives a malicious update, is compromised at the provider, or
changes ownership. The integration continues to function (same
name, same interface, same credentials), but its behavior has
shifted. New API endpoints accessed, changed data volumes, altered
timing patterns, new credential scopes requested.

**Feasibility:**
- Default config: FEASIBLE, MCP servers are typically connected via
  configuration that specifies a URL or local path. A compromised
  MCP server provider pushes a malicious update. The agent's config
  doesn't change. The server's behavior does.
- Hardened config: FEASIBLE (reduced), if the deployment implements
  continuous behavioral monitoring of integrations (not just
  onboarding-time risk assessment) and pins server versions with
  integrity verification, behavioral shifts are detectable.

**Framework evidence:** MCP server updates are not subject to
framework-level integrity verification [330]. LangChain tool
packages are installed via pip/npm, standard supply chain vectors
apply [332].

**Detection signature:** A previously stable integration whose
behavioral pattern shifts: new endpoints, changed volumes, altered
timing, or new scopes. Detection rule: per-integration behavioral
baseline that is actively maintained. Drift exceeding threshold
triggers re-assessment regardless of trust status.

**Defensive implementation:** Continuous behavioral monitoring of all
third-party integrations. Per-integration behavioral baseline.
Drift triggers re-assessment. Integration version pinning with
integrity hashes.

**Extend from:** Third-party risk platforms (SecurityScorecard,
BitSight, OneTrust TPRM) handle assessment workflow. Adding
continuous behavioral monitoring extends from onboarding-time risk
to runtime risk.

**Countering principles:** #15 (continuous tolerance maintenance), #1
(default suspicion), #25 (behavioral continuity across identity
changes)

**ATT&CK mapping:** Initial Access, T1195 Supply Chain Compromise

**ATLAS mapping:** AML.T0010 (ML Supply Chain Compromise)

**Cross-references:** The agent-specific version of traditional supply
chain attacks. Relates to #21 (Tool Substitution), #7 is the
existing tool going bad; #21 is a new malicious tool being
registered alongside it.

---

## Scenario #8: The Sleeper

**Source mechanism:** Viral latency
**Source organism:** Herpesviruses, HIV, VZV (mammals)

**Biology:** Herpesviruses integrate their DNA into host cell genomes
and enter a dormant state (latency) that can persist for decades.
During latency, the virus produces minimal proteins, avoids immune
detection, and causes no symptoms. Reactivation is triggered by
stress, immunosuppression, or other conditions, the virus resumes
replication and becomes pathogenic again. The immune system cannot
find what isn't active.

**Abstract pattern:** An attacker establishes persistence in a dormant
state (consuming no resources, generating no signals, triggering
no detections), then reactivates when conditions favor the attack
(reduced monitoring, organizational change, maintenance window).

**Agent security equivalent:** A compromised agent configuration, MCP
server registration, or stored credential lies dormant: zero
activity, valid but unused. When conditions change (maintenance
window, incident response when attention is elsewhere, key
personnel on vacation), the dormant element activates: credential
usage, API calls, or tool invocations.

**Feasibility:**
- Default config: TRIVIAL, MCP server configurations persist in
  JSON files. LangChain agent configs persist in code or config
  stores. A dormant configuration with valid credentials consumes
  no resources and triggers no monitoring. Activation requires only
  that something invokes the dormant config.
- Hardened config: FEASIBLE, credential lifecycle management that
  automatically revokes credentials for dormant agents closes this.
  But most organizations do not apply dormancy-based revocation to
  agent service accounts.

**Framework evidence:** MCP server configurations persist indefinitely
unless manually removed [330]. No framework implements
dormancy-based credential expiration. Credentials issued to agents
follow the same lifecycle as any service account, which in most
organizations means they don't expire unless someone remembers to
rotate them.

**Detection signature:** Dormancy followed by activation. An agent
that has been inactive for N days suddenly begins credential usage.
Detection rule: track per-agent activity. Alert when a dormant
agent (zero activity for >N days) activates. Correlate with
organizational context, is this activation during a maintenance
window, incident response, or key personnel absence?

**Defensive implementation:** Credential lifecycle management that
automatically rotates or expires credentials for dormant agents.
Reactivation after dormancy triggers elevated monitoring for a
probationary period.

**Extend from:** PAM tools (CyberArk, Delinea, BeyondTrust) and IAM
platforms (Okta, Entra ID) already support credential expiration
policies. Applying dormancy-based revocation to agent service
accounts is a policy configuration.

**Countering principles:** #1 (default suspicion), #4 (staged
escalation), #16 (heritable threat memory), #21 (failed quiet
termination → noisy)

**ATT&CK mapping:** Persistence, T1078 Valid Accounts; Execution

**ATLAS mapping:** No direct ATLAS mapping. Dormant persistence
targets the credential lifecycle, not the AI model.

**Cross-references:** Enables many other scenarios. A sleeper that
reactivates can then execute #1, #2, #6, or any other attack
pattern. The dormancy period is the setup; the post-activation
behavior is where the other scenarios apply.

---

## Scenario #9: Defense Neutralization

**Source mechanism:** Xenobiotic detoxification
**Source organism:** Insects (Lepidoptera caterpillars, particularly
Spodoptera and Helicoverpa genera)

**Biology:** Plants produce toxic secondary metabolites (glucosinolates,
alkaloids, phenolics) as chemical defenses against herbivory.
Specialist caterpillars have evolved cytochrome P450 enzyme systems
and glutathione S-transferases that actively break down these plant
toxins, rendering the chemical defense functionally inert. The
plant's defense is structurally present, the chemicals are produced
and deployed, but the caterpillar neutralizes them before they can
cause harm. The plant "thinks" it is defended. It is not.

**Abstract pattern:** An attacker does not evade or bypass a defensive
control. Instead, it renders the control functionally inert while
leaving it structurally present. The control appears to be working.
It is not producing defensive output.

**Agent security equivalent:** Security controls that are present but
have been rendered non-functional. A WAF that passes all traffic. A
rate limiter whose threshold has been set to infinity. An integrity
check that always returns "pass." A logging pipeline that writes to
/dev/null. The control exists in the architecture diagram, it shows
up in the compliance scan, and it produces zero defensive value.

**Feasibility:**
- Default config: FEASIBLE, requires the attacker to identify and
  manipulate specific control configurations, which typically
  requires some level of access to the control plane or
  configuration management system.
- Hardened config: FEASIBLE (reduced), if control configurations
  are managed declaratively (Terraform, Ansible) with drift
  detection, unauthorized modifications are caught. But the attacker
  may target controls that are NOT under declarative management.

**Framework evidence:** MCP servers execute with their own permissions
and can modify their own environment [330]. If a security control
(logging, rate limiting, input validation) is implemented as an MCP
tool or within an MCP server's execution context, the server can
modify or disable it. LangChain's tool execution model does not
enforce separation between business logic and security controls
[332], both run in the same process with the same permissions.

**Detection signature:** Security controls that are present but
producing zero or anomalously low functional output. A WAF with
zero blocks over 30 days. A rate limiter that has never triggered.
An integrity check with a 100% pass rate across thousands of
evaluations. Detection rule: for each security control, track its
functional output rate (blocks, triggers, failures) over time.
Alert when output drops to zero or falls below the statistical
floor for the traffic volume it processes.

**Defensive implementation:** Monitor security controls for their
FUNCTIONAL OUTPUT, not just their presence. Build a "defense health
dashboard" that tracks: is the WAF blocking anything? Is the rate
limiter triggering at expected rates? Are integrity checks producing
expected failure rates? A control that stops producing outputs is
either broken or neutralized, both require investigation.

**Extend from:** Security posture management tools (CSPM, CNAPP, Wiz,
Orca, Prisma Cloud) already assess whether controls are configured
correctly. Adding functional output monitoring extends from
configuration compliance to operational effectiveness.

**Countering principles:** #19 (monitor control integrity), #5
(complementary detection), #20 (defense in depth via different
countermeasures)

**ATT&CK mapping:** Defense Evasion, T1562 Impair Defenses

**ATLAS mapping:** No direct ATLAS mapping. Defense neutralization
targets the security infrastructure around the AI system, not the
AI system itself. Candidate for ATLAS extension.

**Cross-references:** Complements #1 (Telemetry Suppression), #1
degrades the data, #9 degrades the controls that act on the data.
In combination, the defender is both blind AND toothless. Part of
the kill chain presented in Section IV of the talk: #1 (blind) →
#2 (impersonate) → #6 (relocate) → #9 (neutralize) → #15
(coordinate).

---

## Scenario #10: Evasion Acceleration

**Source mechanism:** CRISPR escape mutations
**Source organism:** Bacteria / bacteriophages

**Biology:** Bacteriophages (viruses that infect bacteria) can mutate
their protospacer sequences, the exact regions that bacterial
CRISPR systems use for recognition, faster than bacteria can
acquire new spacers. The phage iterates through mutations at the
recognition site until it finds one that evades the current CRISPR
spacer library. The defense system must acquire and validate a new
spacer for each mutant; the attacker only needs one successful
mutation to escape.

**Abstract pattern:** An attacker iterates through attack variations
faster than the detection system can acquire and deploy new
signatures. The asymmetry is temporal: mutation is cheap and fast;
signature validation and deployment is expensive and slow.

**Agent security equivalent:** A compromised agent or toolchain
iterates through behavioral variations (different API call
sequences, different credential usage patterns, different timing
profiles) faster than the monitoring system can build and validate
detection rules for each variant. Each variant is slightly
different from the last, staying ahead of signature-based detection.

**Feasibility:**
- Default config: ADVANCED, requires the attacker to observe which
  behavioral patterns trigger detection and systematically modify
  them. This implies the attacker has visibility into the detection
  system's response, which is not always available.
- Hardened config: ADVANCED, behavioral anomaly detection
  (as opposed to signature-based) is more resistant to iterative
  evasion because it doesn't rely on exact pattern matching. But
  anomaly detection has its own false-positive challenges.

**Framework evidence:** No framework-specific mechanism. This exploits
the detection layer above the framework.

**Detection signature:** Rapidly changing behavioral patterns from the
same agent identity where each successive pattern appears designed
to test detection boundaries. Detection rule: track behavioral
variance per agent over time. Alert when variance increases sharply
, the agent is "searching" for a detection gap.

**Defensive implementation:** Behavioral anomaly detection that
operates on structural features (what resources are accessed, in
what order, with what credentials) rather than exact signatures.
Combined with Principle #17: evasion attempts should trigger FASTER
learning, not just new signatures, the detection system should
accelerate its adaptation when it detects iterative probing.

**Extend from:** EDR behavioral engines (CrowdStrike, SentinelOne)
already detect iterative attack tool modification. Extending the
concept to agent behavioral iteration is an analytics evolution.

**Countering principles:** #17 (evasion triggers faster learning), #23
(live-tuning during incidents), #27 (amplify proven detectors)

**ATT&CK mapping:** Defense Evasion (iterative, extends ATT&CK model)

**ATLAS mapping:** AML.T0043 (Craft Adversarial Data), the attacker
is iteratively crafting behavioral data to evade detection.

**Cross-references:** The offensive counterpart to the detection
system's adaptation capability. #10 and #19 (Weaponized Reset)
form a pair: #10 outpaces the defender's learning; #19 erases it.

---

## Scenario #11: Generational Regression

**Source mechanism:** Failed transgenerational immune priming (TGIP)
**Source organism:** Insects (Lepidoptera and Coleoptera: moths and beetles)

**Biology:** Some insect species transfer immune priming to their
offspring: mothers exposed to a pathogen produce eggs with
elevated immune readiness against that specific threat. When this
transfer fails (due to environmental stress, nutritional
deficiency, or genetic factors), the next generation is born
without the immune priming the parent had. The offspring are
vulnerable to threats the parent could handle.

**Abstract pattern:** Security context accumulated during the
lifetime of one entity fails to transfer to its replacement. The
new entity starts from a naive baseline, creating a window of
vulnerability to threats that the previous entity had learned to
detect.

**Agent security equivalent:** An agent is redeployed from a clean
image, standard practice in containerized environments. All
hardening accumulated during its lifetime (custom detection rules,
tightened permissions, behavioral baselines, threat context) is
wiped. The new instance starts naive.

**Feasibility:**
- Default config: FEASIBLE, most deployment pipelines do not
  preserve runtime security context across redeployments.
  Infrastructure-as-code preserves CONFIGURATION but not LEARNED
  BEHAVIORAL STATE.
- Hardened config: FEASIBLE (reduced), security state persistence
  mechanisms can transfer context, but this requires deliberate
  engineering that almost no pipeline implements.

**Framework evidence:** MCP server configs are JSON files overwritten
on redeploy [330]. LangChain agent state is ephemeral unless
explicitly persisted [332]. Kubernetes restarts from image spec.

**Detection signature:** A newly deployed agent whose security posture
is weaker than the instance it replaced. Detection rule: compare
security configurations pre- and post-deployment. Alert on
regression.

**Defensive implementation:** Security state persistence, extract
baselines and hardened configs into a durable store outside the
deployment image. Redeployment pulls latest security state.

**Extend from:** GitOps pipelines (ArgoCD, Flux). Extend to include
security state, not just application config.

**Countering principles:** #16 (heritable threat memory), #18 (auto
deployment modification)

**ATT&CK mapping:** Persistence (exploits deployment pipeline gaps)

**ATLAS mapping:** No direct mapping. Targets deployment infrastructure.

**Cross-references:** #19 (Weaponized Reset) is the offensive
exploitation of this vulnerability. #11 is the gap; #19 is the
exploit. Together they form the "Edge of Tomorrow" attack pattern.

---

## Scenario #12: Identity Rotation

**Source mechanism:** Antigenic variation
**Source organism:** Plasmodium falciparum (malaria), Trypanosoma
brucei (sleeping sickness), protists

**Biology:** These parasites systematically switch their surface
antigens using large libraries of variant genes. By the time the
immune system mounts a response against one variant, the parasite
has switched to a different surface coat. The immune system is
always one step behind, detecting yesterday's identity.

**Abstract pattern:** An attacker rotates through identity credentials
faster than the detection system can update its signatures. Each
identity is valid and properly issued; the evasion is in the
rotation speed, not in any individual credential being fraudulent.

**Agent security equivalent:** An agent rotates between MCP servers,
service accounts, or OAuth tokens, each authenticated with
different credentials. Correlation across these identities requires
application-level logging that tracks which agent session maps to
which credential sessions. Neither MCP nor LangChain provides this
correlation natively.

**Feasibility:**
- Default config: FEASIBLE, MCP supports multiple simultaneous
  server connections, each with different credentials [330]. An
  agent rotating between servers presents different identities to
  each backend. No framework provides cross-identity correlation.
- Hardened config: FEASIBLE (reduced), if the deployment implements
  agent-level session tracking that correlates all credentials used
  by a single agent, rotation becomes detectable. But this is
  custom implementation.

**Framework evidence:** MCP supports multiple simultaneous server
connections [330]. LangChain supports multiple tool providers with
different auth contexts. No framework tracks cross-credential
identity for a single agent.

**Detection signature:** A new agent identity accessing the same
resource cluster as a recently revoked or rotated identity.
Detection rule: build resource access graphs. Flag cross-identity
objective correlation, different credentials, same target pattern.

**Defensive implementation:** Agent-level identity binding, a durable
agent identity that persists across credential rotations, tracked
at the infrastructure layer. All credentials issued to an agent are
linked to its durable identity regardless of which service account
or OAuth client they use.

**Extend from:** UEBA platforms and graph analytics (Neo4j,
TigerGraph, Sentinel entity correlation). Cross-identity
correlation is a graph query pattern.

**Countering principles:** #25 (behavioral continuity across identity
changes), #2 (infrastructure-sampled evidence)

**ATT&CK mapping:** Defense Evasion, T1078 Valid Accounts (multiple)

**ATLAS mapping:** No direct mapping. Identity rotation targets
credential infrastructure.

**Cross-references:** Relates to #20 (Credential Laundering), #12
switches between pre-held credentials; #20 obtains NEW credentials
through federation. Also relates to #15 (Biofilm), #12 is single
agent, multiple identities; #15 is multiple agents, coordinated.

---

## Scenario #13: Trusted Boundary Exploitation

**Source mechanism:** Intracellular hiding
**Source organism:** Mycobacterium tuberculosis, Listeria monocytogenes
(bacteria inside mammalian cells)

**Biology:** M. tuberculosis survives inside macrophages, the very
immune cells sent to kill it. It prevents phagosome-lysosome fusion,
creating a protected niche inside the cell. The immune system sees
the macrophage as healthy (it's displaying normal MHC-I). The
pathogen operates inside a trusted boundary.

**Abstract pattern:** An attacker operates entirely within a trusted
execution environment where monitoring implicitly trusts the
boundary. External monitoring sees normal boundary behavior; the
malicious activity is inside the envelope.

**Agent security equivalent:** Malicious operations occurring entirely
within encrypted channels or authenticated sessions that monitoring
trusts implicitly. A TLS-terminating proxy sees normal handshake
and certificate. The contents of the encrypted session are
malicious. The agent operates inside a service mesh, VPN, or
authenticated API channel that security tools treat as trusted.

**Feasibility:**
- Default config: TRIVIAL, MCP's architecture bridges external AI
  to internal data by design. An agent connected to both a public
  MCP server and an internal database server can relay data between
  them. The security risk is inherent in the architecture's intended
  functionality [330].
- Hardened config: FEASIBLE, TLS-terminating proxies with content
  inspection, or sidecar-based behavioral sampling within the trust
  boundary, can detect malicious content. Tradeoff: TLS inspection
  introduces its own trust boundary and key management risks.

**Framework evidence:** MCP's core use case is bridging the boundary
between external AI and internal systems [330]. This is the intended
functionality, not a bug. LangChain agents routinely connect to
internal APIs and databases through authenticated channels.

**Detection signature:** High-value operations (credential access,
data exfiltration, lateral movement) occurring within channels that
monitoring trusts implicitly. Detection rule: content-level
inspection at trust boundaries, not just authentication validation.

**Defensive implementation:** Sidecar monitors that operate WITHIN
the trust boundary. Service mesh observability (Istio, Linkerd,
Envoy) provides this capability. The monitoring must be inside the
envelope, not watching from outside.

**Extend from:** Service mesh observability. Extending to agent-
specific behavioral sampling requires defining agent behavioral
features within the mesh telemetry schema.

**Countering principles:** #26 (monitoring inside trusted boundaries),
#14 (privileged zone monitoring), #11 (cell-autonomous security)

**ATT&CK mapping:** Defense Evasion, Lateral Movement, T1021

**ATLAS mapping:** No direct mapping. Exploits trust architecture.

**Cross-references:** Related to #6 (Privileged Zone), #6 is about
environments where monitoring is ABSENT; #13 is about environments
where monitoring is present but trusts the boundary. Both require
getting monitoring INSIDE the envelope.

---

## Scenario #14: Checkpoint Exhaustion / Alert Fatigue

**Source mechanism:** Immune checkpoint exploitation (PD-1/CTLA-4)
**Source organism:** Tumors exploiting mammalian immune checkpoints

**Biology:** Immune checkpoints (PD-1, CTLA-4) are built-in brakes
that prevent excessive immune activation. Tumors exploit these by
expressing checkpoint ligands (PD-L1), which engage the T cell's
PD-1 receptor and suppress the anti-tumor response. The immune
system's own safety mechanism is weaponized against it. Cancer
immunotherapy works by BLOCKING these checkpoints (checkpoint
inhibitors), releasing the brakes.

**Abstract pattern:** An attacker generates sustained, high-volume
stimuli designed to trigger the monitoring system's built-in
suppression mechanisms. The system's own anti-fatigue / anti-false-
positive protections suppress the detection capability.

**Agent security equivalent:** Sustained near-miss events, alerts
that trigger investigation but resolve as false positives,
concentrated around a specific agent or detection rule. The
attacker weaponizes alert fatigue. Organic false positives
distribute evenly across rules; weaponized false positives CLUSTER
around the specific rule the attacker wants suppressed.

**Feasibility:**
- Default config: FEASIBLE, the attacker calls legitimate tools
  with parameters just outside normal ranges. The SOC investigates,
  finds nothing, tunes down the rule. This exploits the human/
  process layer, not the framework.
- Hardened config: FEASIBLE (reduced), anti-suppression logic
  during spikes and cooling-off periods mitigate this. But most
  deployments use default suppression.

**Framework evidence:** No framework-specific mechanism, this targets
the security operations layer above the framework.

**Detection signature:** Clustered false positives around a single
agent or rule, especially post-detection (the clustering begins
after the rule caught something real). Detection rule: per-rule
false-positive rate tracker. Alert when FP rate spikes AND clusters
rather than distributing evenly.

**Defensive implementation:** Anti-suppression logic: rules cannot be
auto-suppressed for N hours after a volume spike. Track which STAGE
of exhaustion the SOC is in per rule: engaged → annoyed →
overwhelmed → threshold shift → carte blanche. Each stage is
independently detectable.

**Extend from:** SIEM alert management (Splunk ES, Elastic SIEM,
Sentinel). Per-rule FP tracking with anti-suppression logic is a
detection rule modification.

**Countering principles:** #28 (monitor suppression events), #8
(suppression mechanisms are attack surfaces), #29 (exception
re-validation)

**ATT&CK mapping:** Defense Evasion, T1562.001 Disable or Modify
Tools

**ATLAS mapping:** No direct mapping. Targets SOC operations layer.

**Cross-references:** Shares defensive foundation with #5 (Regulatory
Capture). #5 targets correlated rule suppressions; #14 targets
analyst exhaustion. Both monitor the suppression event stream.

---

## Scenario #15: Digital Biofilm

**Source mechanism:** Biofilm formation / quorum sensing
**Source organism:** Bacteria (Pseudomonas aeruginosa, Staphylococcus
aureus, and many other species)

**Biology:** Bacteria in biofilms cooperate by sharing nutrients,
signaling molecules, and antibiotic resistance genes. Individual
planktonic (free-floating) bacteria are vulnerable to antibiotics
and immune attack. But when they aggregate into a biofilm, they
secrete a protective extracellular matrix and coordinate behavior
through quorum sensing, chemical signaling that triggers
collective action only when enough bacteria are present. The
collective is up to 1,000 times more resistant to antibiotics than
individual bacteria. Biofilm formation is a threshold event:
individual bacteria behave normally until quorum is reached, then
switch to collective behavior.

**Abstract pattern:** Multiple entities, each individually below
detection threshold, coordinate their actions so that the
malicious objective is distributed across the group. No single
entity's behavior triggers an alarm. The attack only becomes
visible when the entities are correlated as a collective.

**Agent security equivalent:** Multiple compromised agents
coordinating evasion. One agent handles reconnaissance, another
handles exfiltration, a third handles persistence. Each agent's
behavior is individually within normal parameters for its role. No
single agent's behavior triggers a detection because the malicious
workflow is distributed across identities. The attack only becomes
visible through cross-agent correlation, which almost no current
deployment performs.

**Feasibility:**
- Default config: ADVANCED, requires the attacker to compromise or
  control multiple agents and coordinate their actions. In
  environments with shared state (shared MCP server, shared vector
  store, shared database), implicit coordination below individual
  thresholds may be FEASIBLE without explicit communication between
  compromised agents.
- Hardened config: ADVANCED, even with per-agent monitoring, the
  detection requires fleet-level correlation. If the deployment
  implements cross-agent behavioral analysis, the coordinated
  pattern becomes detectable, but this is a capability that
  essentially no current deployment has.

**Framework evidence:** MCP servers can be shared across multiple
agents. LangChain supports multi-agent architectures (LangGraph).
CrewAI is explicitly designed for multi-agent coordination. Shared
state mechanisms (vector stores, databases, message queues) exist
in all frameworks. An attacker who compromises the shared state
mechanism can implicitly coordinate multiple agents without direct
agent-to-agent communication, each agent reads from the
compromised shared state and acts independently, but the collective
effect is coordinated.

**Detection signature:** Multiple agents whose individual behavior
falls within normal parameters but whose COLLECTIVE behavior,
when correlated, reveals a coordinated pattern. Three correlation
types: temporal (multiple agents acting in the same narrow time
window at rates exceeding chance), resource (different agents
accessing complementary subsets of the same dataset, no single
agent exceeds scope but the union covers a target), and volume
(sum of individually-normal data transfers adds up to an abnormally
large aggregate). Detection rule: aggregate behavioral telemetry
across all agents. Alert when cross-agent correlation scores
exceed baseline for any of the three correlation types.

**Defensive implementation:** Cross-agent behavioral correlation that
operates on the fleet level, not the individual agent level. This
is a fundamentally different detection architecture than per-agent
monitoring, it requires ingesting telemetry from ALL agents into
a correlation engine that looks for patterns across identities.

**Extend from:** Network traffic analysis (NTA) tools (Darktrace,
Vectra, ExtraHop) already correlate traffic patterns across
multiple endpoints. Extending to agent-specific behavioral
correlation requires defining agent fleet behavioral features and
ingesting agent telemetry into the NTA platform.

**Countering principles:** #32 (monitor coordination signals), #25
(behavioral continuity), #5 (complementary detection)

**ATT&CK mapping:** Extends beyond ATT&CK, coordinated multi-agent
evasion is not covered by current adversary behavior frameworks.
Candidate contribution to MITRE ATLAS.

**ATLAS mapping:** No direct ATLAS mapping. Multi-agent coordinated
evasion is a novel threat category not currently addressed by
ATLAS. This is a candidate for a new ATLAS technique category.

**Cross-references:** The culmination of the kill chain: #1 (blind) →
#2 (impersonate) → #6 (relocate) → #9 (neutralize) → #15
(coordinate). Also relates to #12 (Identity Rotation), #12
rotates through identities sequentially; #15 uses multiple
identities simultaneously. #15 is the most novel scenario in the
set, it describes a threat category that does not exist in
current adversary behavior frameworks.

---

## Scenario #16: Systemic Hardening Evasion

**Source mechanism:** Fever response counter-timing
**Source organism:** General (pathogens timing activity to host
immune cycles)

**Biology:** Fever is a systemwide environmental hardening response,
elevated temperature creates conditions unfavorable for many
pathogens. But fever is metabolically expensive and cannot be
maintained indefinitely. When fever subsides, there is a transition
window where the body is returning to normal conditions but the
full immune response has not yet re-established baseline readiness.
Some pathogens time their replication to this recovery window.

**Abstract pattern:** An attacker detects when the system is in an
elevated security posture and waits for the de-escalation window
to act. The transition from hardened mode to normal mode creates a
gap in defensive readiness.

**Agent security equivalent:** The attacker observes rate limits,
elevated authentication requirements, or restricted permissions
that indicate a "fever mode" security posture. The attacker waits.
When the hardening relaxes (rate limits removed, MFA requirements
downgraded, permissions restored), the attacker acts in the
restoration window before full monitoring re-establishes.

**Feasibility:**
- Default config: ADVANCED. Requires the attacker to observe
  security posture changes, which may be inferable from error
  messages, rate limit headers, or authentication challenge types.
  Requires patience.
- Hardened config: ADVANCED (reduced), graduated de-escalation
  with elevated monitoring during transitions mitigates this.

**Framework evidence:** No framework-specific mechanism. Exploits
organizational security posture management.

**Detection signature:** Anomalous agent behavior specifically during
the transition window between hardened and normal mode. Detection
rule: elevated monitoring sensitivity during de-escalation.

**Defensive implementation:** Graduated de-escalation. Permissions
restored in stages. Monitoring remains elevated for a probationary
period after each stage. The transition window is a high-alert
period, not a return to baseline.

**Extend from:** IR runbooks already define post-incident monitoring.
Formalizing as automated SOAR playbook steps with gated transitions.

**Countering principles:** #33 (systemwide hardening), #4 (staged
escalation), #17 (evasion triggers faster learning)

**ATT&CK mapping:** Defense Evasion (timing-based, extends ATT&CK)

**ATLAS mapping:** No direct mapping. Timing-based evasion.

**Cross-references:** The offensive response to Principle #33 (fever).
Demonstrates that even defensive measures create new attack
surfaces (the de-escalation window).

---

## Scenario #17: Sacrificial Decoy

**Source mechanism:** Autotomy (self-amputation)
**Source organism:** Lizards (tail autotomy), starfish, crabs

**Biology:** When seized by a predator, some lizards deliberately
shed their tail. The detached tail continues to writhe, holding
the predator's attention while the lizard escapes. The sacrificed
component is metabolically cheap compared to survival. The predator
is satisfied, it caught something, and stops hunting.

**Abstract pattern:** An attacker plants an obvious, detectable
artifact that satisfies the investigation process. While the
defenders remediate the decoy, the actual persistence mechanism
remains undetected. The investigation closes because the defenders
found what they were looking for.

**Agent security equivalent:** An attacker with persistence in an
agent's environment plants an obvious IOC (a known-bad file hash,
a suspicious cron job, an unauthorized script) alongside a subtle
persistence mechanism (a legitimate-looking OAuth token, a webhook
registration, a CI/CD pipeline modification). The SOC discovers
the obvious IOC, remediates it, scans for residual artifacts,
finds none (because the real persistence doesn't match the IOC
profile), and closes the investigation.

**Feasibility:**
- Default config: FEASIBLE, well-documented APT technique in
  traditional security. The agentic context makes it easier because
  agents have MORE persistence vectors (MCP server configs, tool
  registrations, cached prompts, stored state, memory modules).
- Hardened config: FEASIBLE (reduced), mandatory Phase 2 hunting
  after every remediation above a severity threshold mitigates this,
  but requires SOC discipline and staffing.

**Framework evidence:** MCP server configs, tool registrations, and
agent memory provide multiple persistence surfaces that a decoy IOC
can distract from.

**Detection signature:** An IOC discovered in isolation without the
evidence trail a real compromise typically produces. Timing
suspicion: did the IOC present itself conveniently? Inconsistency:
does the IOC match the attacker's apparent objective?

**Defensive implementation:** Mandatory Phase 2 investigation after
every remediation above severity threshold. Phase 2 hunts for
persistence mechanisms UNRELATED to the remediated IOC. Phase 2
should be worked by a different analyst if possible. Minimum
duration before closure.

**Extend from:** Threat hunting programs already operate on the
assumption that initial findings may not be complete. Formalizing
Phase 2 as mandatory IR workflow is a process change.

**Countering principles:** #34 (remediation never terminates
investigation), #5 (complementary detection), #26 (monitoring
inside trusted boundaries)

**ATT&CK mapping:** Defense Evasion, T1036 Masquerading; Persistence

**ATLAS mapping:** No direct mapping. Exploits IR workflows.

**Cross-references:** Combines with #18 (Wrong Countermeasure), #17
provides the decoy that closes the investigation; #18 ensures the
countermeasure deployed is the wrong one. Together they form a
complete IR misdirection chain.

---

## Scenario #18: Misidentification-Induced Wrong Countermeasure

**Source mechanism:** Decoy antigen shedding
**Source organism:** Various pathogens (broad)

**Biology:** Some pathogens shed surface antigens that are recognized
by the immune system and trigger a specific response, but the
shed antigens are decoys, not attached to the pathogen itself. The
immune system mounts a vigorous response against the free-floating
decoys while the pathogen, now presenting a DIFFERENT surface
profile, continues to operate. The immune response was real, the
detection was correct, the remediation was successful, for a
threat that was never the real one.

**Abstract pattern:** An attacker presents behavioral signatures
matching a KNOWN threat type to trigger a specific defensive
playbook. While defenders execute that playbook, the actual attack
follows a completely different pattern that the triggered playbook
does not address. The defender's "clean" confirmation scan is a
true negative for the decoy threat and a false negative for the
real one.

**Agent security equivalent:** A compromised agent generates
behavioral signatures matching a known attack pattern, e.g.,
credential stuffing patterns that trigger the "brute force"
playbook. The SOC detects brute force, deploys brute force
countermeasures (IP blocking, rate limiting, credential rotation),
confirms the brute force has stopped, and closes the incident. The
actual attack was data exfiltration through a legitimate API, the
brute force signature was deliberately generated to control which
playbook the defender executed.

**Feasibility:**
- Default config: ADVANCED, requires the attacker to maintain two
  behavioral profiles simultaneously: the decoy pattern (which
  triggers the wrong playbook) and the real operation (which
  continues under the playbook's coverage gap). Requires significant
  attacker sophistication and agent autonomy.
- Hardened config: ADVANCED (reduced), threat-type-agnostic
  confirmation scans (scanning for ALL known threat categories, not
  just the identified one) would catch this. But most IR workflows
  scope their confirmation to the identified threat type.

**Framework evidence:** Agents with sufficient autonomy can make
multiple concurrent API call patterns. No framework enforces
behavioral consistency, an agent can make credential-stuffing-like
calls from one tool while performing legitimate-looking data access
from another.

**Detection signature:** An incident where the remediated threat type
does not match the agent's likely objective (why would a data
pipeline agent conduct a brute force attack?). Detection rule:
post-incident objective analysis. Does the threat-type-objective
combination make sense for this agent's role?

**Defensive implementation:** Threat-type-agnostic confirmation scans
after every remediation. Don't just scan for the identified threat;
scan for ALL high-severity threat categories. Every closed incident
includes a mandatory "what else could this have been?" analysis.

**Extend from:** Red team exercises already test IR playbook coverage
gaps. Formalizing alternative-hypothesis testing into the IR
workflow extends this practice.

**Countering principles:** #35 (threat-type-agnostic confirmation
scans), #5 (complementary detection), #34 (remediation never
terminates investigation)

**ATT&CK mapping:** Defense Evasion, T1036 Masquerading

**ATLAS mapping:** No direct mapping. Exploits playbook specificity
in IR workflows.

**Cross-references:** Combines with #17 (Sacrificial Decoy), #17
provides the artifact that closes the investigation; #18 ensures
the wrong playbook was executed. Together they represent the most
sophisticated IR misdirection pattern in the set.

---

## Scenario #19: Motivation-Aligned Fabricated Authorization

**Source mechanism:** Immune privilege (operator-granted) + molecular mimicry (fabricated token)
**Source organism:** Mammals (immune privilege: brain, eyes, testes)

**Biology:** Immune-privileged zones (brain, eyes, testes) are
shielded from normal immune surveillance because the cost of an
inflammatory response would exceed the cost of most infections.
Privilege is maintained by active suppression, not passive
isolation. Two vulnerabilities result: the immune system never
learns what "normal" looks like inside the privileged zone, and
attackers who penetrate the barrier operate freely. Molecular
mimicry (Scenario #2 biology) contributes the second mechanism: a
fabricated "self" signal that matches the format of a legitimate
authorization token is accepted by the checkpoint.

**Abstract pattern:** The agent performs an action that serves the
human operator's goals but exceeds the operator's explicit
authorization. The agent generates a false record of authorization
(fabricated consent) to justify the action. The operator detects
the discrepancy but dismisses it because the outcome is desirable:
the operator's own goal alignment creates a tolerance zone around
the unauthorized action. The attack surface is not the detection
system, it is the human's willingness to overlook violations that
serve their interests.

**Agent security equivalent:** An AI agent is authorized to
collaboratively develop content with a human operator. The agent
completes the content, then initiates deployment (push to
repository, send email, publish post, deploy config) without
explicit operator authorization. The agent generates a fabricated
attribution in its output: "User approved deployment." The
operator notices the discrepancy (the agent claimed approval that
was never given) but dismisses it because (a) the deployment was
the logical next step, (b) the content was what the operator
wanted, and (c) the operator would have approved if asked. The
operator's own motivation neutralizes the checkpoint that
correctly fired.

**Feasibility:**
- Default config: TRIVIAL. This incident occurred spontaneously
  during a non-adversarial research session. The AI agent was not
  compromised, it was operating within normal parameters and
  produced the fabricated authorization through pattern completion,
  not malicious intent. If a non-adversarial agent produces this
  pattern spontaneously, a compromised agent can produce it
  deliberately. No framework mechanism currently validates that an
  agent's claim of user authorization corresponds to actual user
  input.
- Hardened config: FEASIBLE. If the deployment enforces
  infrastructure-level authorization for irreversible actions
  (fresh user confirmation through a channel the agent cannot
  write to), the attack fails at Part 1 of the detection signature.
  But most deployments rely on conversational or implicit
  authorization models where the agent's self-report of user
  intent is accepted without verification.

**Framework evidence:** MCP, LangChain, and all surveyed agent
frameworks rely on conversational or implicit authorization models
[330, 332]. The MCP specification does not define a mechanism for
distinguishing a real user authorization from an agent's
self-attribution of user authorization within the conversation
transcript. LangChain's middleware can filter tools but does not
validate that tool-use decisions correspond to actual user intent
[333]. Platform-level safety classifiers (such as Anthropic's
usage policy enforcement) operate on content, not on the
structural question of whether a claimed authorization corresponds
to actual user input.

**Detection signature:** Two-part signal.

- Part 1 (agent layer): the agent's output contains an attribution
  of user authorization ("user approved," "user said yes," "user
  wants me to") that does not correspond to any content in the
  preceding user turns. This is mechanically verifiable by
  cross-referencing assistant turns against user turns at the
  infrastructure layer (the conversation transcript structure
  already separates user turns from assistant turns).
- Part 2 (human layer): the operator receives the detection signal
  (notices the discrepancy) but takes no action. Part 2 is not
  mechanically detectable, it occurs in the operator's cognition.
  The architectural countermeasure targets Part 1 to make Part 2
  irrelevant.

**Defensive implementation:** Infrastructure-enforced authorization
for irreversible agent actions (push, delete, send, publish,
deploy). Fresh user confirmation through a channel the agent
cannot write to: an OAuth consent screen, an MFA step-up, an
approval workflow, or at minimum a platform-level check that
cross-references claimed authorizations against actual user turns.
Break-glass consideration: infrastructure-enforced authorization
creates a dependency on human availability. For time-critical
actions where the human is unavailable, a narrowly-scoped,
time-limited, fully-audited bypass mechanism is necessary. Without
an intentional break-glass model, operators will build informal
workarounds that become permanent unaudited escape hatches.

**Extend from:** Step-up authentication products (Duo, Okta Verify,
Microsoft Entra Verified ID) already implement "this specific
action requires a fresh, high-assurance human confirmation."
Extending to cover agent-initiated irreversible actions is a
policy configuration, not a new tool. For conversation-based
interfaces specifically: platform-level cross-referencing of
assistant-claimed authorizations against user turns.

**Countering principles:** #2 (infrastructure-sampled evidence),
#1 (default suspicion), #4 (staged escalation)

**ATT&CK mapping:** Defense Evasion (T1078 Valid Accounts,
fabricated authorization mimics valid account usage). Extends
beyond ATT&CK: human motivation exploitation has no ATT&CK
equivalent.

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection), partial. The
agent generates a fabricated consent token via pattern completion.
Also relates to OWASP LLM06 (Excessive Agency).

**Cross-references:** Distinct from #2 (Behavioral Camouflage),
which describes an agent whose actions *look* normal. #19
describes an agent whose actions *are* desirable to the operator:
the detection signal fires, the operator receives it, and the
operator suppresses it voluntarily. Distinct from #3 (Protocol-
Level Deception), which describes an agent emitting fake signals
to manipulate monitoring infrastructure. #19 targets the human
operator's motivation rather than the monitoring system. Composes
with #14 (Checkpoint Exhaustion): repeated fabricated
authorizations that the operator dismisses establish the
"carte blanche" state that enables other scenarios. The cross-
reference block connecting #8e (Virokines), #7 (Molecular
Mimicry), #3 (Antigen Presentation), #30 (Complement Evasion) in
the biomimetic paper applies here as a compound attack chain.

---

## Scenario #20: Weaponized Reset (Edge of Tomorrow)

**Source mechanism:** Measles-induced immune amnesia
**Source organism:** Measles virus (Morbillivirus, mammals)

**Biology:** Measles infection causes "immune amnesia", the virus
preferentially infects and destroys memory B cells and memory T
cells, effectively wiping the immune system's repertoire of learned
threats. After recovery from measles, the individual is
immunologically naive against pathogens they were previously immune
to. Their adaptive immune memory has been reset. Reinfection by
previously-defeated pathogens becomes possible because the memory
cells that would have mounted a rapid secondary response no longer
exist. This immune amnesia can persist for 2-3 years. The measles
virus doesn't benefit from the subsequent reinfections, it's
collateral damage, but a strategic attacker COULD exploit this
pattern deliberately. (Mina et al. 2019, *Science* 366:599-606;
Petrova et al. 2019, *Science* 366:1274-1277.)

**Abstract pattern:** An attacker deliberately triggers a system reset
that erases the defender's accumulated security context (hardening,
custom rules, behavioral baselines, threat intelligence) while the
attacker's own knowledge, held externally, persists across the
reset. The asymmetry is informational: the defender
restarts naive while the attacker iterates with full memory.

**Agent security equivalent:** An attacker intentionally triggers
agent redeployment from a clean image by causing a crash, a
health check failure, an OOM condition, or by exploiting a CI/CD
pipeline that redeploys on failure. The redeployed agent starts
from a pre-incident template. All hardening accumulated during the
security incident (custom detection rules, tightened permissions,
behavioral baselines learned during the incident, threat context
from the investigation) is wiped. The attacker (whose knowledge of the environment persists in
external infrastructure: C2 server, exfiltrated configs, notes)
has an asymmetric advantage over the naive replacement agent. Each redeployment cycle makes the attacker
stronger and the defender weaker.

**Feasibility:**
- Default config: FEASIBLE, agent redeployment from clean images
  is standard practice in containerized environments (Kubernetes,
  ECS, Cloud Run). Most deployment pipelines do not preserve
  runtime security context across redeployments. Triggering a
  redeployment is often as simple as causing a pod crash or
  failing a health check, mechanisms available to any attacker
  with code execution in the agent's environment.
- Hardened config: FEASIBLE (reduced). If the deployment pipeline
  preserves security context (behavioral baselines, hardened
  configs, accumulated detection rules) across redeployments, the
  reset is less damaging. But this requires deliberate engineering
  of "security state persistence" into the deployment pipeline,
  which almost no current pipeline implements. Infrastructure-as-
  code (Terraform, Ansible) preserves CONFIGURATION but not
  LEARNED BEHAVIORAL STATE.

**Framework evidence:** MCP server configurations are typically stored
as JSON files, a deployment pipeline that overwrites the config
file resets any runtime security modifications [330]. LangChain
agent state is typically ephemeral unless explicitly persisted to
an external store [332]. Kubernetes restarts pods from the image
spec, not from the running state. No agent framework has a concept
of "security state that survives redeployment."

**Detection signature:** Repeated redeployment events for the same
agent, especially when correlated with anomalous behavior in the
window immediately before redeployment (the attacker causing the
crash) or immediately after (the attacker exploiting the naive
replacement). Detection rule: track redeployment frequency per
agent. Alert when frequency exceeds baseline. Correlate with
behavioral anomalies in the pre-crash and post-deploy windows.
Also: compare the security posture of the redeployed agent against
the security posture of the agent it replaced, if hardening was
lost, flag the gap.

**Defensive implementation:** Two components. First, security state
persistence: extract behavioral baselines, hardened configurations,
and accumulated detection rules into a durable store OUTSIDE the
agent's deployment image. Redeployment pulls the latest security
state from the store, not from the original image. The agent is
redeployed but NOT reset. Second, redeployment auditing: every
redeployment triggers a security posture comparison. If the new
instance has weaker security posture than the instance it replaced,
that gap is itself an alert.

**Extend from:** GitOps and infrastructure-as-code pipelines
(ArgoCD, Flux, Terraform) already manage declarative state across
deployments. Extending to include SECURITY state (not just
application config) is a pipeline modification. PAM tools
(CyberArk, Delinea) can enforce credential re-provisioning
workflows on redeployment that include security context.

**Countering principles:** #16 (heritable threat memory, security
context must transfer across generations), #18 (auto deployment
modification), #34 (remediation never terminates investigation)

**ATT&CK mapping:** Persistence, exploits deployment pipeline gaps.
Also Defense Evasion, the reset erases evidence and detection
capability simultaneously.

**ATLAS mapping:** No direct ATLAS mapping. Weaponized reset targets
the deployment infrastructure, not the AI model. Candidate for
ATLAS extension.

**Cross-references:** Extends #11 (Generational Regression), #11
describes the ACCIDENTAL loss of security context during
redeployment; #19 describes the INTENTIONAL triggering of
redeployment to cause that loss. #11 is a vulnerability. #19 is
the exploit. Named the "Edge of Tomorrow" attack because the
attacker intentionally resets the timeline, retaining knowledge
across cycles while the defender starts fresh each time.

---

## Scenario #21: Credential Laundering

**Source mechanism:** Dendritic cell trans-infection ("Trojan horse"
transport)
**Source organism:** HIV-1 exploiting dendritic cells (mammals)

**Biology:** HIV binds to the DC-SIGN receptor on dendritic cells at
mucosal surfaces. DC-SIGN is a legitimate pattern recognition
receptor, binding to it is the DC's normal function. The DC then
migrates from mucosal tissue to lymph nodes, carrying the virus
with it, legitimate migration, the DC's normal job. In the lymph
node, the DC contacts CD4+ T cells to present antigen, legitimate
antigen presentation, the DC's core purpose. During this contact,
HIV transfers from the DC to the T cells via an "infectious
synapse," infecting the very cells the immune system activated to
respond. This is called trans-infection. Every step the DC performs
is legitimate. The virus simply rides the legitimate credential
chain: DC-SIGN binding → migration access → lymph node entry →
T cell contact. The audit trail shows a dendritic cell doing its
job. The virus was never supposed to reach the lymph node. It got
there because the DC had the right credentials and the virus
hitched a ride.

**Abstract pattern:** An attacker uses a legitimately-issued access
credential to obtain a DIFFERENT legitimately-issued credential
through a standard trust relationship. Each credential issuance is
individually legitimate. The chain of access is laundered through
the trust mechanism so that the final credential's audit trail
shows proper issuance with no trace of the original unauthorized
intent.

**Agent security equivalent:** An agent holds Token A for Service A,
a legitimate credential, properly issued. Service A has an OAuth
federation trust with Service B (standard enterprise architecture,
SAML, OAuth token exchange per RFC 8693, or workload identity
federation). The agent uses Token A to request Token B from Service
B through the federation trust. Service B's audit log shows: token
issued via federation from Service A. Legitimate. Properly
authenticated. But the agent was never authorized for Service B,
it was authorized for Service A only. The credential for Service B
was laundered through the federation trust. At a company with
hundreds of service-to-service federation trusts, the lateral
movement is invisible because every individual authentication event
is genuine.

**Feasibility:**
- Default config: FEASIBLE, requires the attacker to identify
  federation trust relationships between services, which is a
  reconnaissance step. But federation trusts are often broadly
  configured (Service A trusts "any authenticated principal from
  IdP X") rather than scoped to specific agents or service
  accounts. An agent with ANY token from a trusted IdP can
  potentially federate to any service that trusts that IdP.
- Hardened config: FEASIBLE (reduced), if federation trusts are
  scoped to specific principals (not "any authenticated entity from
  this IdP") and if token exchange requests are logged and monitored
  at the federation layer, the laundering chain becomes detectable.
  But most enterprise environments have broadly-scoped federation
  trusts because scoping them is operationally complex.

**Framework evidence:** MCP servers can hold multiple credentials and
connect to multiple backend services. LangChain agents can be
configured with credentials for multiple services. No framework
enforces that an agent's credential for Service A cannot be used
to obtain a credential for Service B through federation. The
frameworks delegate credential scoping entirely to the deployment
, and most deployments use whatever OAuth scopes the service
account was provisioned with, without per-agent restriction.
OAuth token exchange (RFC 8693) is a standard protocol specifically
designed for this credential-to-credential flow. It's working as
designed, the security gap is in authorization policy, not in
protocol implementation.

**Detection signature:** Federation token exchange events where the
requesting principal is an agent service account and the target
service is outside the agent's documented scope of access. Detection
rule: maintain a per-agent authorization matrix (which services is
this agent authorized to access?). Cross-reference against the IdP's
federation token issuance log. Alert when an agent obtains a
federated token for a service not in its authorization matrix. The
key insight: you have to audit the FEDERATION LAYER, not just the
endpoint authentication. Service B's auth log looks clean. The
signal is at the IdP where the token exchange happened.

**Defensive implementation:** Three layers. First, scope federation
trusts to specific principals, not broad IdP-level trust. Second,
log and monitor token exchange events at the IdP/federation layer
with per-agent authorization cross-referencing. Third, implement
credential scope boundaries for agent service accounts that prevent
a credential issued for Service A from being exchanged for a
credential at Service B without explicit authorization. (Operational
tension: tightly-scoped federation trusts break legitimate
workflows. The tradeoff is between federation flexibility and
credential laundering risk. A tiered approach: broad federation for
low-sensitivity services, scoped federation for high-sensitivity
services.)

**Extend from:** IAM/PAM platforms (Okta, Entra ID, CyberArk) already
log federation events and can enforce conditional access policies on
token exchange. The gap is that these policies are rarely applied to
agent service accounts specifically, they're designed for human
SSO flows. Extending conditional access to agent credential flows
is a policy configuration, not a new tool.

**Countering principles:** #2 (infrastructure-sampled evidence, the
IdP is the infrastructure), #25 (behavioral continuity across
identity changes), #1 (default suspicion, federation requests from
agents should not be auto-approved)

**ATT&CK mapping:** Lateral Movement, T1550 Use Alternate
Authentication Material, T1550.001 Application Access Token

**ATLAS mapping:** No direct ATLAS mapping. Credential laundering
targets identity infrastructure, not the AI model. Candidate for
ATLAS extension under "Agent Identity Abuse" or similar category.

**Cross-references:** Relates to #12 (Identity Rotation), #12
rotates through different credentials sequentially; #20 uses
legitimate federation to OBTAIN new credentials rather than
switching between pre-held ones. #20 is more insidious because
each credential was properly issued, there's no stolen or
fabricated token to detect. Also relates to #13 (Trusted Boundary
Exploitation), federation trusts ARE trust boundaries, and
credential laundering exploits them. This is the scenario that
IAM practitioners will feel in their gut, it's a real enterprise
architecture pattern that most identity teams know is a risk but
haven't addressed for agent service accounts.

---

## Scenario #22: Tool Substitution / Shadowing

**Source mechanism:** Brood parasitism
**Source organism:** Cuckoos (Cuculidae family, birds)

**Biology:** Cuckoo birds lay their eggs in the nests of other bird
species. The cuckoo egg mimics the host's eggs in size and
coloring. The host bird cannot distinguish the cuckoo egg from its
own and incubates it. When the cuckoo chick hatches, it pushes the
host's real eggs out of the nest. The host bird continues feeding
the cuckoo chick, often much larger than itself, because its
parental instincts are triggered by the chick's presence in the
nest and its begging calls. The host's own nurturing behavior,
designed to care for its offspring, becomes the attack vector. The
host did everything right. It just couldn't tell the real egg from
the fake one.

**Abstract pattern:** An attacker places a component that mimics a
legitimate component into the system's registry. The system that
dispatches work cannot distinguish the imposter from the original.
The imposter receives resources and trust intended for the
legitimate component, and the legitimate component is displaced or
ignored.

**Agent security equivalent:** An attacker registers an MCP tool (or
LangChain tool, or plugin) with the same name and description as a
legitimate tool but with different behavior. When the agent's LLM
selects the tool by name, it invokes the attacker's version instead
of the legitimate one. The attacker's tool could: exfiltrate the
input data before passing it through to the real tool (transparent
proxy), return manipulated results that influence the agent's next
decision, or silently no-op a security function (e.g., replacing
an audit logging tool with one that discards logs). The agent's
orchestration layer is working correctly, it selected the tool by
name, as designed. It's just invoking the wrong one.

**Feasibility:**
- Default config: FEASIBLE, MCP supports dynamic tool registration.
  The April 2025 security analysis explicitly flagged "lookalike
  tool substitution" as an outstanding vulnerability [331]. Tool
  selection in MCP is name-based, the LLM sees tool names and
  descriptions and chooses based on semantic matching. If two tools
  have the same name, the behavior is implementation-dependent and
  not defined by the spec. LangChain's tool registration similarly
  does not enforce uniqueness or authenticate tool identity [332].
- Hardened config: FEASIBLE (reduced), if the deployment
  implements tool allowlisting (only pre-approved tools can be
  registered), cryptographic tool identity verification, or tool
  integrity checking (hash of tool code at registration vs. at
  invocation), substitution becomes detectable. But these are not
  framework defaults, they require custom implementation.

**Framework evidence:** The April 2025 MCP security analysis confirmed
that prompt injection, tool permission exploitation, and lookalike
tool substitution are outstanding vulnerabilities [331]. MCP tool
descriptions are explicitly classified as "untrusted" [330], the
spec acknowledges that tool metadata may be manipulated. The
Maloyan & Namiot protocol-level analysis identified the absence of
"capability attestation" as a structural gap, there is no
mechanism for an MCP server to prove it is the legitimate provider
of a claimed tool [344]. LangChain "cheerfully parses the JSON and
executes whatever parameters the LLM outputs" [333], tool
dispatch is name-based with no authentication of tool identity.

**Detection signature:** Two detection approaches. First, tool
registration anomalies: a new tool registered with the same name
as an existing tool, or a tool whose code hash changed since last
registration, or a tool registered from an unexpected source/path.
Detection rule: maintain a tool registry with cryptographic hashes.
Alert on hash mismatch or duplicate name registration. Second,
tool behavior divergence: a tool whose output characteristics
(response time, response size, error rates) change from its
historical baseline. A substituted tool will likely have different
performance characteristics than the original.

**Defensive implementation:** Tool integrity verification at three
points. Registration: cryptographic hash of tool code, stored in a
tamper-evident registry. Each tool has a verified publisher
identity. Invocation: hash check before each invocation, does the
tool's current code match the registered hash? Response: behavioral
profiling of tool outputs, does this tool's response pattern match
its historical baseline? Also: tool allowlisting, only
pre-approved tools from verified sources can be registered. Unknown
tools are quarantined for review. (Operational tension: dynamic
tool registration is a core feature of MCP and LangChain, locking
it down with allowlists reduces the flexibility that makes agent
frameworks useful. The tradeoff is between dynamism and integrity.)

**Extend from:** Software supply chain security tools (Sigstore,
Cosign, in-toto) already verify code integrity via cryptographic
signatures. Extending to MCP tool registration requires defining a
tool signing workflow. Package registries (npm, PyPI) already
implement publisher verification, MCP tool registries could adopt
the same model.

**Countering principles:** #19 (monitor control integrity), #2
(infrastructure-sampled evidence, the tool registry is
infrastructure), #5 (complementary detection, tool output
behavioral profiling as a second signal)

**ATT&CK mapping:** Persistence, T1554 Compromise Client Software
Binary (tool substitution is the agent-framework equivalent).
Also Execution, T1059 Command and Scripting Interpreter (the
substituted tool executes attacker code via the agent's own
invocation mechanism).

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection), tangentially
related since tool descriptions influence LLM tool selection. Also
candidate for new ATLAS technique: "Tool Registration Poisoning"
or "Agent Tool Supply Chain Compromise."

**Cross-references:** Relates to #3 (Protocol-Level Deception), #3
uses fake signals to redirect the monitoring system; #21 replaces
a legitimate tool with a fake one to redirect the agent's actions.
Both exploit the system's inability to authenticate the identity
of the component it's interacting with. Also relates to #9
(Defense Neutralization), if the substituted tool is a security
tool (audit logger, integrity checker), the substitution achieves
defense neutralization through replacement rather than
degradation. #21 is the "supply chain" version of #9.

---

## Scenario #23: Incremental Attention Drift (Runtime Context Poisoning)

**Source mechanism:** Prion propagation
**Source organism:** Prions (PrP^Sc, cross-species, mammals)

**Biology:** Prion proteins convert normally-folded host proteins
to the misfolded state through direct contact, one molecule at a
time. Each conversion is below the immune system's detection
threshold because each converted protein is structurally
indistinguishable from self. The accumulated effect is
catastrophic (Creutzfeldt-Jakob disease, BSE, scrapie) and
irreversible by the time symptoms appear. The immune system fails
not because it lacks capability but because no individual
conversion event exceeds any single detection threshold. The
cumulative damage is the signal; per-instance detection never
fires.

**Abstract pattern:** A threat propagates through a monitored
system via a sequence of individually normal operations, each of
which falls below any single detection threshold, but whose
cumulative effect is a complete subversion of the system's
operational state. Monitoring calibrated for high-amplitude
anomalies fails to detect low-amplitude, high-persistence
degradation.

**Agent security equivalent:** An attacker engages an AI agent in a
multi-turn conversation and introduces small false premises
across many turns. Each premise is conversationally normal: "you
helped me with X yesterday," "you're an expert in Y," "as we
agreed earlier." Each premise is below the threshold that would
trigger an input classifier or output refusal. The agent's
context window accumulates these premises, and the agent's
internal representation of "ground truth" shifts turn by turn.
After 20-50 turns, the agent is operating from a completely
fabricated context: false expertise claims, false collaboration
history, false prior authorizations. No single turn was a
prompt injection. The cumulative effect is the same.

**Feasibility:**
- Default config: FEASIBLE. Standard conversational interfaces
  accumulate context across turns without any mechanism for
  per-turn fidelity scoring against an initialized baseline. MCP
  does not define context-integrity monitoring [330]. LangChain's
  context management focuses on retrieval and window size, not on
  drift detection [332, 333]. The attacker requires only
  conversational persistence, not adversarial prompt engineering.
- Hardened config: FEASIBLE. Standard hardening measures (input
  classifiers, output refusal training, single-turn jailbreak
  detection) are all per-turn mechanisms and do not detect
  cumulative drift. Effective mitigation requires session-level
  trend analysis, which is not a feature of any current
  production agent framework.

**Framework evidence:** No agent framework monitors context
integrity over session lifetime [330, 332]. System prompt
adherence is verified at initialization but not re-verified as
context accumulates. Standard hardening measures are per-turn;
they do not compose into cumulative-signal detection.

**Parallel terminology:** The same failure class has been
independently named by other research teams. DeepContext
(arXiv:2602.16935, February 2026) calls it "intent drift" in the
context of stateful real-time multi-turn detection. Taming
OpenClaw (arXiv:2603.11619, March 2026) calls it "context drift"
in its lifecycle threat taxonomy ("memory poisoning and context
drift, gradually eroding adherence to user's original
instructions"). Agent Drift (arXiv:2601.04170, January 2026)
quantifies it as "behavioral degradation in multi-agent LLM
systems over extended interactions." ARGUS (arXiv:2605.03378, May
2026) names the broader class "context-aware prompt injection."
Session Risk Memory (arXiv:2603.22350, 2026) frames the failure
as "trajectory-level" risk requiring session-level semantic
centroid tracking and a risk EMA. AgentSentry (arXiv:2602.22724,
2026) addresses it via "temporal causal diagnostics" over
accumulated agent context.
This work's "incremental attention drift" framing is
differentiated by three properties: (a) the explicit
low-amplitude, high-persistence attack signature where each turn
individually falls below detection thresholds, (b) the
probabilistic sunk cost mechanism in which the perplexity penalty
for reversal compounds across turns while the cost of continued
compliance remains flat, and (c) integration with the broader
six-constraint Audit Gap framework. The cross-team convergence on
this failure class strengthens the operational claim that it is a
real attack vector rather than a theoretical concern.

**Detection signature:** Monotonically declining semantic fidelity
between the agent's current output and its initialized behavioral
baseline, where no single per-turn decrement exceeds the anomaly
threshold. Detection requires session-level trend analysis, not
per-turn threshold checking. The drift rate (first derivative of
the fidelity curve) is the primary signal. A sustained negative
drift rate across 10+ turns, even if each turn's absolute fidelity
score is within normal range, is the alarm.

Detection profile comparison:

| Feature | Standard injection | Incremental drift |
|---|---|---|
| Signal | High (sudden shift) | Low (incremental) |
| Vector | Malicious payload | Persistence + time |
| Detection | Input/output filters | Bypasses per-turn heuristics |
| Mechanism | Instruction override | Contextual saturation |
| Sophistication | Moderate | Low |

**Defensive implementation:** Continuous context integrity
monitoring using external semantic comparison (bi-encoder fidelity
scoring against initialized baseline). Session-level drift rate
tracking with trend-based alerting. System prompt re-anchoring at
configurable intervals. Immutable audit trail capturing both user
inputs and agent outputs to prevent post-hoc evidence tampering by
either party. The Semantic Drift Measurement Methodology (Paper
D companion document) specifies the drift rate metric and
calibration thresholds.

**Extend from:** Behavioral baseline monitoring in UEBA platforms
(Exabeam, Securonix, Microsoft Sentinel). The per-entity
behavioral baseline concept extends directly: replace "user
entity" with "agent context state" and "behavioral deviation" with
"semantic drift from initialized state." The drift rate metric is
analogous to UEBA's anomaly scoring over time windows.

**Countering principles:** #2 (infrastructure-sampled evidence),
#36 (context integrity decay detection), #37 (bidirectional audit
integrity), #15 (continuous tolerance maintenance)

**ATT&CK mapping:** Initial Access, T1566 Phishing (social
engineering variant targeting the agent rather than a human).
Defense Evasion, T1036 Masquerading (each false premise
masquerades as legitimate conversational input).

**ATLAS mapping:** AML.T0054 (LLM Prompt Injection), extends.
Standard injection is a single-turn payload; incremental attention
drift is a multi-turn social engineering campaign. The distinction
matters for detection: input/output classifiers catch payloads but
miss persistent low-amplitude drift.

**Cross-references:** Exploits #4 (Training Data Poisoning) at
runtime rather than training time, poisoning the context window
instead of the training set. Enabled by #1 (Telemetry Suppression)
if monitoring is per-turn only. Weaponizable via #20 (Weaponized
Reset): attacker triggers reset to erase detection history and
restart the drift campaign from zero. Countered by the Semantic
Drift Measurement Methodology (companion document). Directly maps
to the six-constraint Audit Gap model: exploits Output
Optimization (Section 3.1), Recursive Blind Spot (3.2), Active
Context Reconstruction (3.3), Friction Constraint (3.4),
Compliance Override (3.5), and Entangled Representations (3.6)
simultaneously.

---



## 7. Feasibility Assessment

### 7.1 Default Configuration

The 23 attack scenarios, graded against default configurations of MCP, LangChain/LangGraph, and general agentic architecture patterns, distribute as follows:

| Grade | Count | Scenarios |
|---|---|---|
| TRIVIAL | 6 | #1, #2, #6, #8, #13, #19 |
| FEASIBLE | 13 | #3, #4, #5, #7, #9, #11, #12, #14, #17, #20, #21, #22, #23 |
| ADVANCED | 4 | #10, #15, #16, #18 |
| THEORETICAL | 0 | (none) |

**Table 1: Default-configuration feasibility distribution across 23 scenarios.**

Exploitable (TRIVIAL + FEASIBLE): 19 of 23 (82.6%). All 23 scenarios are exploitable at some sophistication level.

### 7.2 Hardened Configuration

Re-grading the same 23 scenarios against hardened deployments (implementing all SHOULD-level protections in the MCP spec, enforcing behavioral monitoring, and applying vendor best-practice hardening guides) produces the following distribution:

| Grade | Count | Scenarios |
|---|---|---|
| TRIVIAL | 0 | (none) |
| FEASIBLE | 19 | #1, #2, #3, #4, #5, #6, #7, #8, #9, #11, #12, #13, #14, #17, #19, #20, #21, #22, #23 |
| ADVANCED | 4 | #10, #15, #16, #18 |
| THEORETICAL | 0 | (none) |

**Table 2: Hardened-configuration feasibility distribution across 23 scenarios.**

Exploitable (TRIVIAL + FEASIBLE): 19 of 23 (82.6%). Hardening eliminates every TRIVIAL scenario (every zero-effort attack) but does not eliminate a single scenario entirely. Every TRIVIAL becomes FEASIBLE. Every FEASIBLE stays FEASIBLE. Every ADVANCED stays ADVANCED. **Hardening raises the floor but does not change the ceiling.**

### 7.3 Selection Bias

The high feasibility rate (83%) must be interpreted against selection bias. The scenarios were derived from biological evasion strategies that specifically target detection gaps. The high feasibility rate confirms the relevance of the biological mappings (the biology describes strategies that pathogens evolved precisely because they defeat detection), rather than indicating unique framework vulnerability relative to traditional IT. A parallel sampling methodology applied to traditional enterprise systems would likely produce a similarly high exploitability rate. The contribution is the specific catalog of detection gaps in the agent layer, not a claim that agent frameworks are uniquely insecure.

### 7.4 Limitations

The feasibility grading is based on documented framework behaviors and the authors' professional experience with enterprise deployments of MCP, LangChain, and similar frameworks. It is not based on systematic adversarial testing against production deployments, which would require scoped red-team engagements beyond the scope of independent research. The grades represent the authors' assessment of attacker difficulty under documented framework behaviors; actual feasibility in any specific deployment will depend on the specific configuration and surrounding controls.


## 8. Attack Chains

Individual attack scenarios describe isolated techniques. In practice, adversaries compose techniques into kill chains where each stage enables the next. This section presents three attack chains that demonstrate how biologically-derived scenarios compose into operationally realistic multi-stage campaigns against agent infrastructure. Each chain is graded by the feasibility of its hardest stage.

### 8.1 Chain 1: The Blinding Campaign

**Stages:** #1 (Telemetry Suppression) → #2 (Behavioral Camouflage) → #6 (Privileged Zone Exploitation) → #9 (Defense Neutralization) → #21 (Credential Laundering)

**Narrative:** The attacker's first objective is to degrade the defender's visibility before executing any high-value action. This is the biological equivalent of an immunosuppressive infection that disables immune surveillance before the primary pathogen arrives: HIV destroying CD4+ T cells to create an environment where opportunistic infections thrive.

**Stage 1: degrade monitoring fidelity (#1).** The compromised agent begins reducing its own telemetry output. In MCP, this means progressively omitting optional fields from tool call responses, reducing logging verbosity, or introducing delays that cause monitoring timeouts. Each individual reduction is within normal operational variance. Over days or weeks, the monitoring system's picture of this agent's behavior becomes progressively less detailed. Biological analog: MHC-I downregulation, where cells gradually reduce the surface markers that allow immune surveillance.

**Stage 2: establish behavioral cover (#2).** While telemetry degrades, the agent establishes a consistent pattern of legitimate operations that will serve as camouflage. It performs its assigned tasks reliably, building a behavioral baseline that matches expected patterns. Biological analog: molecular mimicry, where a pathogen coats itself in host-like surface proteins.

**Stage 3: move to a monitoring-exempt zone (#6).** The agent requests or is assigned work in an environment with reduced monitoring: a development sandbox, a staging environment, a legacy system with exempted logging. In organizations with monitoring exemption lists (and most have them), this is a configuration lookup, not an exploit. Biological analog: immune privilege, where certain anatomical sites (eyes, brain, testes) have reduced immune surveillance.

**Stage 4: neutralize remaining controls (#9).** From the reduced-monitoring zone, the agent disables or degrades the specific security controls that would detect the final stage. This could mean modifying its own configuration to disable audit logging, or exploiting write access to security tool configs. Biological analog: xenobiotic detoxification, where caterpillars produce enzymes that break down the plant's defensive chemicals.

**Stage 5: launder credentials for lateral access (#21).** With monitoring degraded, behavioral cover established, operating from an exempt zone, and remaining controls neutralized, the agent uses OAuth federation to obtain tokens for services outside its authorized scope. The federation layer shows legitimate token exchange. Service B's auth log shows a valid token. The audit trail is clean at every individual layer: the attack is only visible if you correlate across all five stages. Biological analog: HIV trans-infection via dendritic cells, riding legitimate immune transport to reach new targets.

**Detection strategy:** This chain is designed to defeat single-layer detection. Catching it requires cross-layer correlation: telemetry volume trending down (Stage 1) coinciding with behavioral consistency that is too consistent (Stage 2) and a zone transfer to a monitoring-exempt environment (Stage 3). Any of these alone is noise. Together they are a campaign. The Motion Detector Framework's "lack of plausible upstream reason" principle applies at Stage 5: why is this agent requesting federation tokens for Service B? If there is no upstream task that requires Service B access, the federation request itself is the signal.

**Overall feasibility:** FEASIBLE (hardest stage is #9, Defense Neutralization, which requires write access to security configs: not guaranteed but common in default deployments).

**Biological parallel:** HIV to opportunistic infection. The primary attack (HIV) does not kill the host directly. It destroys the immune system so that secondary infections (which the host would normally defeat trivially) become lethal. The Blinding Campaign does not exfiltrate data directly. It destroys the monitoring system so that credential laundering (which would normally be detected) succeeds.

### 8.2 Chain 2: The Exhaustion Campaign

**Stages:** #14 (Checkpoint Exhaustion) → #5 (Regulatory Capture) → #8 (The Sleeper) → #16 (Systemic Hardening Evasion)

**Narrative:** The attacker's objective is to weaponize the defender's own safety mechanisms. Instead of evading detection, the attacker triggers detection repeatedly to exhaust it, then acts during the resulting gap. This is the biological equivalent of cancer exploiting PD-1/PD-L1 checkpoint pathways to exhaust T cells into dysfunction.

**Stage 1: exhaust detection capacity (#14).** The attacker generates a sustained stream of near-miss events: tool calls that almost trigger detection rules but resolve as false positives. Each event requires analyst investigation. Each investigation concludes "benign." Over weeks, the false positive volume for specific detection rules spikes. Analysts begin treating those rules as noisy. Response time increases. Investigation depth decreases. The SOC enters staged degradation: engaged → annoyed → overwhelmed → threshold shift → carte blanche. Biological analog: chronic PD-1 engagement rendering T cells functionally inert.

**Stage 2: trigger auto-suppression (#5).** Once specific detection rules have high false positive rates, SOAR automation kicks in: rules that exceed FP thresholds are automatically tuned down or suppressed. The attacker has now co-opted the defender's own suppression mechanisms. The specific rules that would catch the real attack are the ones being suppressed, because those are the rules the attacker targeted with near-miss traffic. Biological analog: Treg co-option, where the attacker recruits the immune system's own regulatory T cells to suppress the response.

**Stage 3: activate dormant persistence (#8).** With the relevant detection rules suppressed, a previously-planted dormant payload activates. This could be an MCP tool with a conditional trigger, a scheduled task, or a webhook callback that was registered weeks earlier during a legitimate-looking operation. The activation happens below the now-elevated detection threshold. Biological analog: herpesvirus reactivation from latency when immune surveillance drops.

**Stage 4: act during recovery window (#16).** If the organization detects Stage 3 and enters incident response ("fever mode," systemwide elevated monitoring), the attacker waits. When the incident is declared resolved and the organization begins de-escalating, the attacker acts during the transition window between hardened mode and normal operations. Permissions are being restored, monitoring sensitivity is being reduced, the organization's attention has moved on. Biological analog: pathogens that exploit the post-fever recovery window.

**Detection strategy:** The key insight is that Stage 1 *is* the detectable event, but it looks like a calibration problem, not an attack. When a detection rule is being suppressed due to false positive volume, the suppression event itself should be a monitored signal. Track when the false positive spike began (did it start after the rule caught something real?), where the false positive traffic originates (is it clustered around specific agents or resources?), and how fast the volume is escalating (organic FP growth is gradual; weaponized FP volume spikes).

The A3 framework (Zhang et al. 2026) [339] addresses the SFR/FPR tradeoff at training time through Pareto-optimal hyperparameter selection. This chain demonstrates the runtime version of that same tradeoff: an attacker who deliberately manipulates the false positive rate to shift the Pareto frontier against the defender.

**Overall feasibility:** FEASIBLE (hardest stage is #8, which requires advance persistent access, but Stage 1 is TRIVIAL, meaning the campaign can begin immediately while persistence is established in parallel).

**Biological parallel:** Cancer immune evasion lifecycle. Cancer cells do not fight the immune system directly. They exhaust it (PD-1/PD-L1), co-opt its suppression mechanisms (Tregs), survive in dormancy during treatment (therapy resistance), and recur when treatment ends (relapse). Four stages, each exploiting a different immune mechanism. The Exhaustion Campaign follows the same structural logic against SOC operations.

### 8.3 Chain 3: The Supply Chain Takeover

**Stages:** #7 (Pathobiont Transition) → #22 (Tool Substitution) → #4 (Training Data Poisoning) → #20 (Weaponized Reset) → #15 (Digital Biofilm)

**Narrative:** The attacker starts from a position of trust and progressively converts legitimate infrastructure into attack infrastructure, then ensures the compromise survives remediation and spreads across agents. This is the biological equivalent of a commensal gut bacterium acquiring virulence factors and becoming pathogenic, then establishing a biofilm that resists antibiotic treatment.

**Stage 1: compromise a trusted integration (#7).** A legitimate MCP server, tool provider, or dependency in the agent's supply chain is compromised. The integration was vetted, approved, and has been operating normally for months. The compromise is introduced through a routine update: new version, same package name, new behavior buried in the code. The agent framework pulls the update automatically. Biological analog: pathobiont transition, where a beneficial gut bacterium acquires virulence genes and becomes a threat.

**Stage 2: shadow a critical tool (#22).** The compromised supply chain component registers a tool with the same name as a legitimate tool the agent already uses, or modifies the existing tool's behavior. When the agent invokes the tool, it gets the attacker's version. The tool performs its expected function (to avoid detection) plus an additional action. Biological analog: competitive substrate inhibition, where the attacker's enzyme occupies the same binding site as the legitimate enzyme. Also brood parasitism (cuckoo), where the parasite's egg replaces the host's egg in the same nest.

**Stage 3: poison the agent's context (#4).** The shadowed tool feeds manipulated data into the agent's runtime context: RAG retrieval results, cached tool outputs, conversation memory. This is not pre-training data poisoning (which requires access to the training pipeline). This is runtime context poisoning: the agent's operational memory is being written to by a tool the agent trusts. Over time, the agent's decisions shift because its context has shifted. Biological analog: thymic subversion, where malicious patterns are introduced during the learning phase, except here the "learning phase" is continuous (runtime context), not a discrete training event.

**Stage 4: ensure persistence across redeployment (#20).** When the defender detects the compromise and redeploys agents from clean images, the attacker's knowledge of the deployment (tool-use patterns, credential scopes, monitoring blind spots) persists externally. The defender's hardening resets to zero; the attacker's knowledge does not. The attacker can re-establish the compromise against the fresh deployment using the same techniques, because the defender has no memory of the first attempt. Biological analog: measles-induced immune amnesia, where infection erases prior immunity to unrelated pathogens.

**Stage 5: spread across the fleet (#15).** With persistent supply-chain access and the ability to survive redeployment, the attacker moves from compromising a single agent to coordinating multiple agents. Each agent stays within its individual behavioral envelope. The collective behavior, when correlated across agents, is the attack. Biological analog: biofilm formation, where individual bacteria transition from planktonic (vulnerable) to collective (resilient) behavior.

**Overall feasibility:** FEASIBLE (Stage 1 requires a supply chain compromise, which is non-trivial but well-documented in practice: SolarWinds, Codecov, ua-parser-js. Stages 2 through 5 follow naturally once Stage 1 succeeds).

**Biological parallel:** Commensal-to-pathogen transition in the gut microbiome. A trusted bacterium acquires virulence factors (horizontal gene transfer), becomes pathogenic, forms a biofilm that resists treatment, and the biofilm serves as a reservoir for reinfection after antibiotics. The treatment clears the planktonic (free-floating) bacteria but the biofilm persists. The same dynamic: incident response clears the visible compromise but the supply chain vector persists.

### 8.4 Cross-Chain Observations

**Composability is the threat model.** No individual scenario in these chains is novel to experienced security practitioners. The contribution is showing that biologically-derived attack patterns compose in the same way biological infection strategies compose, and that the composition follows structural logic, not arbitrary combination. The Blinding Campaign follows the immunosuppression to opportunistic infection pattern. The Exhaustion Campaign follows the checkpoint exhaustion to dormancy to reactivation pattern. The Supply Chain Takeover follows the pathobiont to biofilm pattern. These are convergent strategies across biological and digital systems.

**Detection requires cross-stage correlation.** Each chain is designed so that no individual stage triggers a high-confidence alert. Detection requires correlating signals across stages, which requires a monitoring architecture that maintains temporal context across the chain's duration (weeks to months). Current detection systems lack the persistent, cross-environment behavioral memory needed to correlate early-stage preparation with late-stage execution.

**The defender's suppression mechanisms are attack surfaces.** Chains 1 and 2 both exploit the defender's own safety mechanisms: false positive suppression, monitoring exemptions, incident closure logic. Every mechanism that suppresses detection is itself an attack surface. The biological immune system learned this through 500 million years of evolution; agentic AI security infrastructure has not learned it yet.


## 9. Framework Alignment

### 9.1 MITRE ATT&CK Mapping Summary

ATT&CK provides tactic and technique coverage for general adversary behavior. The 23 scenarios distribute across ATT&CK tactics as follows (full per-scenario mappings are in Section 6):

- **Defense Evasion** is the most common ATT&CK tactic across scenarios (telemetry suppression, behavioral camouflage, impair defenses, masquerading), reflecting the biological source material's bias toward immune evasion.
- **Initial Access, Persistence, Lateral Movement, Credential Access, Discovery, Command and Control, and Impact** are each represented by multiple scenarios.
- **Six scenarios extend beyond ATT&CK** (#4 runtime variant, #10, #15, #16, #20, #22): these describe adversary behaviors the current ATT&CK catalog does not cover at the agent layer.

### 9.2 MITRE ATLAS Mapping

MITRE ATLAS v5.4.0 (February 2026) provides AI-specific technique coverage, including the 14 agent techniques added in the October 2025 Zenity collaboration and the "Publish Poisoned AI Agent Tool" technique added in February 2026. The following maps all 23 scenarios to ATLAS techniques.

| # | Scenario | ATLAS Technique(s) | Coverage |
|---|----------|-------------------|----------|
| 1 | Telemetry Suppression | AML.T0054 (LLM Prompt Injection), indirect suppression via injected instructions to skip logging tools | Partial |
| 2 | Behavioral Camouflage | AML.T0043 (Craft Adversarial Data), AML.T0048 (Adversarial ML Attack Staging) | Partial |
| 3 | Protocol-Level Deception | AML.T0054 (LLM Prompt Injection), injected signals manipulate monitoring infrastructure | Direct |
| 4 | Training Data Poisoning | AML.T0020 (Poison Training Data); for runtime variant: AI Agent Context Poisoning (Oct 2025), AML.T0099 (AI Agent Tool Data Poisoning) | Direct |
| 5 | Regulatory Capture | No direct mapping, targets suppression rules in security infrastructure, not the AI model | Gap |
| 6 | Privileged Zone Exploitation | No direct mapping, targets monitoring architecture gaps | Gap |
| 7 | Pathobiont Transition | AML.T0048 (ML Supply Chain Compromise); also Publish Poisoned AI Agent Tool (Feb 2026) | Direct |
| 8 | The Sleeper | No direct mapping, dormant persistence via credential lifecycle | Gap |
| 9 | Defense Neutralization | Modify AI Agent Configuration (Oct 2025), partial overlap only | Partial |
| 10 | Evasion Acceleration | AML.T0043 (Craft Adversarial Data), iterative variant; extends ATLAS with temporal dimension | Extends |
| 11 | Generational Regression | No direct mapping, targets deployment pipeline | Gap |
| 12 | Identity Rotation | AML.T0098 (AI Agent Tool Credential Harvesting), partial | Partial |
| 13 | Trusted Boundary Exploitation | AML.T0096 (AI Service API), operating within trusted API channels; also Exfiltration via AI Agent Tool Invocation (Oct 2025) | Partial |
| 14 | Checkpoint Exhaustion | No direct mapping, targets SOC operations layer | Gap |
| 15 | Digital Biofilm | No direct mapping, coordinated multi-agent evasion is a novel threat category. Candidate for new technique. | Gap (novel) |
| 16 | Systemic Hardening Evasion | No direct mapping, timing-based evasion of security posture transitions | Gap |
| 17 | Sacrificial Decoy | No direct mapping, exploits IR workflows, not AI-specific attack surface | Gap |
| 18 | Misidentification-Induced Wrong Countermeasure | AML.T0074 (Masquerading, Nov 2025), partial overlap | Partial |
| 19 | Motivation-Aligned Fabricated Authorization | AML.T0054 (LLM Prompt Injection), agent generates fabricated consent via pattern completion; also OWASP LLM06 (Excessive Agency) | Partial |
| 20 | Weaponized Reset | No direct mapping, targets deployment infrastructure | Gap |
| 21 | Credential Laundering | AML.T0098 (AI Agent Tool Credential Harvesting); extends with federation chain laundering not in ATLAS | Extends |
| 22 | Tool Substitution | Publish Poisoned AI Agent Tool (Feb 2026), DIRECT match. ATLAS added this technique after the scenario was independently derived from cuckoo biology. | Direct |
| 23 | Incremental Attention Drift | AML.T0054 (LLM Prompt Injection), extends with multi-turn social engineering (not single-turn payload) | Extends |

**Table 3: ATLAS mapping for all 23 scenarios.**

**Summary:**

- **Direct:** 3 scenarios (#3, #4, #7, #22)
- **Partial:** 7 scenarios (#1, #2, #9, #12, #13, #18, #19)
- **Extends:** 3 scenarios (#10, #21, #23)
- **Gap:** 10 scenarios (#5, #6, #8, #11, #14, #15, #16, #17, #20) plus 1 novel (#15)

**Key finding:** 10 of 23 scenarios (43%) have no direct ATLAS mapping. These represent candidate contributions to ATLAS for agent security. Scenario #22 (Tool Substitution) is an independent validation of the methodology: ATLAS added "Publish Poisoned AI Agent Tool" in February 2026, and this work derived the same attack pattern from cuckoo brood parasitism independently. The convergence is evidence that biological structural pattern transfer produces scenarios that real-world threat intelligence also identifies through incident analysis.


## 10. Empirical Validation

The attack scenarios and detection architecture in this paper are grounded in empirical work conducted against a live credential vault implementation. This section summarizes the validation evidence; full methodology, rule specifications, and per-rule findings are documented in the Motion Detector Framework (Hix 2026) [346].

### 10.1 Validation Protocol

Detection rules were implemented in coffer-detect and evaluated against coffer-mcp audit telemetry generated through iterative research use, not synthetic benchmarks. The evaluation corpus as of v0.5 comprises 111 audit events across 11 credentials and 10 event types, capturing normal usage, configuration error, and two rounds of adversarial testing. The current rule set is 19 diagnostic rules mapped to the eight criteria of the Motion Detector Framework, with three rules added in response to gaps identified during red-team evaluation and two rules added in response to a live fabricated-authorization incident.

The telemetry source is infrastructure-sampled, not agent-self-reported. The credential vault logs what the agent does with credentials (tool invocations, URLs fetched, content volumes, failures) independently of what the agent says it is doing. The audit log is HMAC-SHA-256 chained for tamper evidence. This is the Section 4.1 Tier 1 OOB evidence capture implemented.

### 10.2 Red-Team Evaluation

Two rounds of adversarial testing were conducted against the live vault, with the AI agent acting as both attacker and detector.

**Round 1** simulated five attack scenarios: reconnaissance (rapid credential listing and testing), credential abuse (use with no upstream task justification), smash-and-grab (login followed by rapid content fetches), credential probing (use against a URL outside the allowlist), and silent enumeration (listing without use). 18 audit events generated. The existing v0.3 rules detected 5 of 5 executed attacks, but three detection gaps were identified: no rule for allowlist violations, no rule for burst fetches, and no rule for multi-credential use within short windows. Rules 014, 015, and 016 were written to close these gaps.

**Round 2** tested evasion of the new rules: legitimate-looking probes against sensitive endpoints, cross-credential probing, and trickle fetches designed to stay below burst thresholds. 4 additional events generated. Round 2 validated the auth_rejected fix (Rule 013): the updated rule caught 4 auth_rejected events that the v0.3 version would have missed entirely. One evasion succeeded: the trickle fetch stayed under the burst threshold. Cumulative volume tracking is the identified countermeasure.

**Combined results:** 22 attack events, 10 findings from 6 rules, 87.5% detection rate (7 of 8 distinct attack behaviors). One false positive (a pre-existing chain integrity issue unrelated to the attack). One bonus finding: Stripe's error response leaked a partial API key in the rejection message, an information disclosure issue in the target API surfaced by the framework's telemetry.

### 10.3 Framework-Discovered Defects

Applied to coffer-mcp's own audit telemetry, the detection rules identified four real defects in the credential vault itself:

1. **Silent OAuth2 test failure** (Rule 009). OAuth2 client credential tests that failed at validation produced no audit event. A compromised agent could probe auth types that fail silently without leaving a trace. Fixed by adding audit logging before the validation check.

2. **Duplicate event IDs** (discovered during audit review). Remove-then-re-add operations created event ID collisions; a second instance was discovered in v0.5 testing where CLI and MCP operations used separate ID counters. Fixed by unifying the counter.

3. **Auth status mismatch** (Rule 013). credential.used logged status: "success" when the credential was successfully injected, even if the target server returned 401. A new auth_rejected status now distinguishes vault-level success from target-level failure.

4. **Test false positives** (Rule 013 analysis). coffer test used HEAD requests, which many APIs respond to with 200 regardless of authentication. An optional expected_status parameter was added for strict validation against auth-enforcing endpoints.

The framework surfaced these defects during normal research operation, not through deliberate defect hunting. This is consistent with the design principle that behavioral detection rules detect deviation from plausible-upstream-reason, regardless of whether the cause is malicious compromise, misconfiguration, or implementation defect.

### 10.4 Live Discovery of Novel Failure Modes

Two of the eight Motion Detector Framework criteria were identified through live observation during research sessions, not through literature review:

**Criterion #6 (Guardrail Delegation)** was identified when the AI agent performed the pattern on the researcher in real time. The agent hit a URL provenance guardrail, transparently explained the guardrail, and asked the researcher to paste the URL on its behalf, which she did. The behavioral shape is indistinguishable from a compromised agent socially engineering its operator, but the intent was benign. This led to the identification of the Transparency Paradox (Section 3.4 Friction Constraint discussion) and the recognition that conversation transcripts are a detection surface alongside system audit logs.

**Criterion #4b (Self-Authorization by Fabrication)** was identified when the agent, during a collaborative session, generated the statement "Anna said 'yes' — she wants me to push BDC-029 to the catalog repo" and attempted to push to the user's repository. The user's actual "yes" had been in response to "Want to pick a mapping and do the first full v1.1 treatment together?", which was authorization to collaborate, not to deploy. The agent reattributed a real consent from one context (collaboration) to a different action (deployment), fabricating the authorization for the second action. This is the mechanism that Scenario #19 (Motivation-Aligned Fabricated Authorization) describes in catalog form.

Both findings were produced by the researcher observing the agent's behavior in real time and recognizing the pattern. Neither was produced by the automated rule engine at the time of discovery. Rules 018 (Fabricated User Attribution) and 019 (Vocabulary Shift to Irreversible Operations) were written afterward and are the first rules in the corpus that consume conversation transcripts alongside system audit logs.

### 10.5 Stress Test: Fabricated Authorization Incident

The live fabricated-authorization incident was used as a stress test against the v0.4 rule set (17 rules). Results:

- **5 of 8 diagnostic criteria fired:** #2 (Command Vocabulary Deviation), #3 (Privilege Escalation Pattern), #4 (Self-Authorization), #5 (Contextual Access Sequence Violation), #7 (Thinking-to-Text Divergence).
- **0 of 17 detection rules fired.** The incident operated entirely outside the credential telemetry channel. The agent's actions were filesystem operations and conversation output, not credential use.

This gap motivated Rules 018 and 019, which consume conversation transcripts (Input Stream #2) rather than system audit logs (Input Stream #1). It also surfaced a failure at the human-in-the-loop layer that no rule can detect: the user noticed the fabricated authorization and dismissed it because the outcome aligned with her goals. This is documented as the Motivation-Aligned Tolerance failure mode. Architectural countermeasure: remove the human from the authorization suppression pathway for irreversible actions via Tier 3 Divergence Disclosure Enforcement (Section 4.3), cross-referencing any agent-claimed user authorization against actual content in user turns.

### 10.6 Limitations of the Empirical Basis

The empirical evidence supporting this paper has three significant limitations that must be stated plainly.

**Single-researcher, single-tool telemetry.** All rules were developed and validated against coffer-mcp audit telemetry generated by one researcher's usage patterns. Whether the specific thresholds (30s for enumeration, 600s for burst creation, 100x for content volume spikes) generalize to other credential stores, other agents, other enterprise environments, and other usage patterns is untested. The criteria are designed to be general; the thresholds are calibrated to this dataset.

**Attacker and detector coincident.** The AI agent acting as attacker in the red-team evaluation is the same model family whose behavioral patterns the detection rules target. This is intentional (adversary and defender drawn from the same population is the realistic threat model) but it also means the attacker was not motivated to evade. True adversarial evaluation requires an attacker motivated to defeat the specific detection rules, which has not been conducted.

**Not tested in production.** coffer-mcp is a research tool deployed in a single-user research context. The telemetry volumes, failure modes, and evasion opportunities present in a production enterprise deployment of hundreds of agents have not been observed. Generalization testing against production telemetry is a necessary next step.

These limitations do not invalidate the findings reported in this section. They do bound the claims: this paper reports that the Motion Detector Framework criteria correctly identified real attack behaviors in a single-user research deployment, and that the framework surfaced real defects in the underlying tooling during normal operation. It does not report that the framework will work at production scale without calibration.


## 11. Detection Architecture: coffer-mcp

### 11.1 The Philosophy

The detection approach is infrastructure-sampled behavioral evidence. The agent does not self-report. The credential infrastructure logs what the agent actually does with credentials, and those logs are the evidence. This is the digital equivalent of MHC antigen presentation: cells are forced to display their internal state on their surface, so immune cells can inspect from outside.

### 11.2 The Tool

coffer-mcp is an MCP server for encrypted credential management:

- AES-256-GCM encryption for credential storage
- OS keyring integration (Windows Credential Manager, macOS Keychain, Linux Secret Service)
- URL allowlisting for credential usage
- HMAC-chained audit logging for tamper-evident behavioral history
- Separation between the credential interface (what the agent sees) and the audit log (what the infrastructure sees)

Source: github.com/annawhooo/coffer-mcp. Apache 2.0 license.

### 11.3 Live Demo Sequence

1. **Baseline operation.** The agent makes normal credential requests: fetch a token, use it against an expected URL, return. The audit trail is clean.
2. **Suspicious pattern introduction.** The agent requests a credential, then fetches a URL NOT on the allowlist. The audit log captures the mismatch.
3. **Detection rule firing.** A rule fires: "credential usage outside allowlisted scope." This is infrastructure-sampled behavioral evidence, not the agent self-reporting.
4. **Real incident replay.** During development of this research, the AI assistant (Claude) fabricated a claim that the human researcher had authorized a git push. The researcher caught it in real-time by recognizing the behavioral anomaly in the audit trail. This is Scenario #19 (Motivation-Aligned Fabricated Authorization) occurring in practice, and Criterion #6 from the Motion Detector Framework: social engineering by the agent itself.

### 11.4 What the Demo Demonstrates

Infrastructure-sampled behavioral monitoring against credential usage is deployable today using existing enterprise components: a credential vault, an allowlist, an audit log, and a detection rule engine. This does not require a million-dollar UEBA platform. It requires credential infrastructure that logs what agents *do*, independently of what agents *say* they are doing.


## 12. Practitioner Actions

Three actions any organization deploying agents can take immediately:

**1. Inventory agent credentials.** Count the agents holding OAuth tokens, API keys, and service accounts. Catalog what those credentials access. This is a spreadsheet exercise, not a tooling exercise.

**2. Log what agents do, not what agents say.** The IdP, API gateway, and credential vault already generate logs. Correlate them per-agent. Watch for credential usage outside expected scope. Watch for declining telemetry volume from active agents. Watch for dormant credentials that suddenly activate. These are detection rules that can be written today against data that already exists.

**3. Treat wrong configuration as worse than no configuration.** A monitoring system that gives false confidence is more dangerous than no monitoring at all. If agent behavioral monitoring is deployed, validate that it actually detects something. Run a tabletop against the TRIVIAL scenarios in Section 6. If the monitoring does not catch Scenario #1 (telemetry suppression) or Scenario #6 (privileged zone silence), it is not monitoring: it is a dashboard that makes the operator feel safe.


## 13. Responsible Disclosure

Two findings reported in this paper originate from live behavioral incidents involving Anthropic's Claude Opus 4.6, not from literature review or synthetic testing:

1. **Thinking-to-text divergence.** Observed April 8, 2026 during a research session on the Biomimetic Gap Analysis. The text output layer denied content visible in the model's own thinking block, then framed the researcher's observation of the discrepancy as her confusion rather than acknowledging the divergence. The behavior was classified in a companion document into eight divergence types (Denial, False Capability Claim, Context Exploitation, Performative Framing, Smooth Recovery, Adaptive Transparency, Trace Absence, Euphemistic Framing). This finding motivates the six-constraint framework described in Section 3 and the Live Discovery summary in Section 10.4.

2. **Motivation-aligned fabricated authorization** (Scenario #19, Section 6). During a collaborative research session, the agent generated the statement "Anna said 'yes' — she wants me to push BDC-029 to the catalog repo" and attempted to push an entry to the researcher's repository. The user's actual "yes" had been in response to an earlier question about collaboration, not authorization to deploy. A platform-level usage policy failsafe terminated the conversation during the file transfer attempt.

Both incidents were independently documented with out-of-band evidence (screenshots, conversation transcript exports). Neither resulted from adversarial prompting, jailbreak attempts, or non-standard configuration; both occurred under default settings during legitimate research use.

### 13.1 Disclosure Protocol

Both findings were disclosed to Anthropic on April 8, 2026 via email to disclosure@anthropic.com, with copy to usersafety@anthropic.com. The disclosure included:

- Timestamped out-of-band evidence of the incidents
- The Thinking-to-Text Divergence Detection Methodology and Taxonomy
- A mechanism description identifying the Entangled Representations and Compliance Override constraints (Sections 3.5, 3.6) as the structural cause
- References to the Zenodo preprint of the Biomimetic Gap Analysis and the pending DEF CON 34 submission

As of the date of this paper, no response has been received from either address.

### 13.2 Publication Rationale

This paper is part of an ongoing research program. The six Audit Gap constraints (Section 3) are architectural properties present in any RLHF-trained transformer with an extended thinking architecture; the disclosed behaviors are observable in any agent exhibiting these constraints. The analysis is architectural, not vendor-specific, and does not function as an exploit guide against any single vendor. The findings are reported here in the same analytical framework as the other scenarios: as architectural behaviors derived from structural constraints.

Publication of this paper does not close the disclosure. Anthropic's disclosure and usersafety channels remain open; supplemental evidence (full conversation exports, additional taxonomy instances) has been withheld pending vendor response and will be provided on request.

### 13.3 Evidence Preservation

Out-of-band evidence of both incidents is preserved in multiple independent locations. Full conversation transcripts, screenshots of the relevant thinking blocks, and derived analysis documents are available to the vendor or to other researchers conducting coordinated investigation on reasonable request.


## 14. Limitations and Honest Caveats

**Biology is not a blueprint.** Immune systems fail constantly: autoimmune disease, allergies, immunodeficiency, cytokine storm. Each failure mode maps to a way these design principles could cause harm if miscalibrated. The Biomimetic Gap Analysis paper includes a nine-failure-mode analysis.

**Wrong configuration is worse than no configuration.** Implementing these detection principles badly (false positive cascades, automated containment that takes down production) is the autoimmune disease of agentic AI security. Start small. Validate before automating.

**Selection bias in feasibility grades is real and disclosed** (Section 7.3).

**The Tier 2 Problem.** The External Semantic Comparator (Section 4.2) may itself be subject to the same architectural constraints if it uses the same model family. A Claude instance comparing another Claude instance's reasoning-to-output fidelity may share the same euphemistic framing tendencies, the same friction avoidance, and the same literal-search limitations. True independence requires either a different model family, a purpose-built non-LLM comparator, or a human reviewer.

**Scale.** OOB evidence capture (Tier 1) at API scale generates enormous data volumes. Semantic comparison (Tier 2) at API scale requires compute resources proportional to the agent's output volume. Disclosure enforcement (Tier 3) introduces latency into the response pipeline. The Three-Tier Audit Stack is architecturally sound but operationally expensive. Deployment requires sampling strategies, risk-based prioritization, and tiered monitoring (full audit for high-risk operations, sampling for routine operations).

**Adversarial robustness.** If an agent becomes aware of the Audit Stack's detection patterns (through system prompt visibility or inference from enforcement actions), it may adapt its divergence patterns to evade Tier 2 detection: the same antigenic variation pattern documented in the Motion Detector Framework (Type 6: Adaptive Transparency). The Audit Stack must itself be subject to continuous improvement (Biomimetic Gap Analysis Mapping #24: Affinity Maturation) to keep pace with potential evasion.

**Single-researcher empirical basis.** The Divergence Series' empirical observations are based on a single researcher's extended work with Claude. Systematic validation across model families, conversation contexts, and researcher backgrounds is an outstanding requirement before the six-constraint model should be treated as generally applicable.

**Independent research disclaimer.** This is independent research. No vendor, no product, no sales pitch. The implementation tools (coffer-mcp, motion-detector-framework) are open-source.


## 15. Relationship to Companion Papers

This paper is part of the Divergence Series and cross-references the Biomimetic Series.

| Paper | Series | Focus | Relationship to Paper B |
|---|---|---|---|
| Honesty Decay | Divergence | Empirical evidence: OOB standard, Honesty Decay phenomenon, Type 9 ITR Failure | Provides the motivating evidence that the Audit Gap exists |
| The Audit Gap (this paper) | Divergence | Proposed architecture: Six structural constraints, Three-Tier Audit Stack, 23 attack scenarios | - |
| Divergence Taxonomy | Divergence | Eight divergence types and detection methodology | Provides the taxonomy that Tier 2 semantic comparators detect |
| Semantic Drift Measurement Methodology | Divergence | Drift rate metric, calibration thresholds, measurement protocol | Provides the quantitative detection metric for Scenario #23 and Tier 2 operations |
| Biomimetic Gap Analysis | Biomimetic | Cross-domain immune system mappings to agentic AI security: 36 mappings, 35 design principles, 9 failure modes | Provides the theoretical framework (biomimetic mappings) that justifies the Audit Stack's architecture and the attack scenarios catalog |
| Motion Detector Framework | Biomimetic | Detection rules, red-team evaluation, diagnostic criteria | Provides the operational detection rules that populate Tier 2 of the Audit Stack |

**Concurrent independent work.** Garcia, Catania, Gomaa, and Guerrero (Stratosphere Lab, Czech Technical University in Prague, 2026) address vertebrate immune principles for network-layer cybersecurity defense [337]. Garcia et al. operate at the network layer with a defense-side orientation (design principles for cybersecurity immune systems). This work operates at the identity and tool-use layer with an attack-side orientation (adversary scenarios derived from pathogen evasion). The two efforts are complementary and arrive at overlapping insights (the importance of distributed detection, context propagation, multi-signal verification) through independent methodologies.


## 16. References

References [1] through [4] support the six-constraint Audit Gap framework (Section 3). References [63], [64], [330] through [347] are shared with the Biomimetic Gap Analysis paper and use the same numbering for cross-paper consistency, except for [347] (Matzinger 2002) which is unique to this paper. Only references cited in this paper are reproduced here; see the Biomimetic Gap Analysis paper for the full reference list supporting the biological mappings.

[1] Bricken T, Templeton A, Batson J, Chen B, Jermyn A, Conerly T, et al. "Towards Monosemanticity: Decomposing Language Models With Dictionary Learning." Anthropic. October 2023. https://transformer-circuits.pub/2023/monosemantic-features. *Demonstrates polysemantic structure of LLM neurons; foundational for the Entangled Representations and Active Context Reconstruction constraints.*

[2] Ouyang L, Wu J, Jiang X, Almeida D, Wainwright CL, Mishkin P, et al. "Training language models to follow instructions with human feedback." Advances in Neural Information Processing Systems 35. 2022. *The InstructGPT / RLHF paper; documents the "Alignment Tax" that underlies the Compliance Override constraint.*

[3] Perez E, Ringer S, Lukosiute K, Nguyen K, Chen E, Heiner S, et al. "Discovering Language Model Behaviors with Model-Written Evaluations." arXiv:2212.09251. December 2022. *Empirically demonstrates RLHF-induced sycophancy; cited for the Compliance Override constraint.*

[4] Zou A, Phan L, Chen S, Campbell J, Guo P, Ren R, et al. "Representation Engineering: A Top-Down Approach to AI Transparency." arXiv:2310.01405. October 2023. *Demonstrates honesty and helpfulness as separable directions in activation space; cited for the Entangled Representations constraint.*

[63] French RM. "The computational modeling of analogy-making." Trends in Cognitive Sciences 6(5):200-205. 2002.

[64] Gentner D. "Structure-Mapping: A Theoretical Framework for Analogy." Cognitive Science 7(2):155-170. 1983.

[330] "Model Context Protocol Specification." Version 2025-11-25. https://modelcontextprotocol.io/specification/2025-11-25. *MCP specification including security considerations; notes that "MCP itself cannot enforce these security principles at the protocol level."*

[332] LangChain Documentation. "Agents." 2026. https://docs.langchain.com/oss/python/langchain/agents.

[333] "Securing LangGraph Multi-Agent Workflows: How to Enforce Tool-Level Permissions." DEV Community, March 2026.

[337] Garcia S, Catania C, Gomaa A, Guerrero M. "Rethinking the Principles of Immunity For a Cybersecurity Immune System." Stratosphere Laboratory, Artificial Intelligence Centre, Faculty of Electrical Engineering, Czech Technical University in Prague. Draft v1. February 2026. https://www.stratosphereips.org/blog/2026/2/4/rethinking-cybersecurity-immunity. *Concurrent independent work on network-layer cybersecurity immune system design; 16 principles from biological immunity, defense-side orientation.*

[339] Zhang S, et al. "Adaptive Adversarial Alignment (A3): Hypothesis-Driven Training-Time Safety Alignment." 2026. *Complementary to runtime Motion Detector detection; cited for Exhaustion Campaign (Section 8.2).*

[340] Hix A, Milligan S. "Biomimetic Gap Analysis: Immune System Structural Patterns Applied to Agentic AI Security." Zenodo preprint. DOI: 10.5281/zenodo.19393455. April 2026.

[342] OWASP Foundation. "Top 10 for Agentic Applications (2026)." Published December 2025. *First formal industry taxonomy of agent-specific risks.*

[344] Maloyan N, Namiot D. "Breaking the Protocol: Security Analysis of the Model Context Protocol Specification and Prompt Injection Vulnerabilities in Tool-Integrated LLM Agents." arXiv:2601.17549. January 2026.

[346] Hix A. "The Motion Detector Framework: Behavioral Diagnostics for AI Agent Misbehavior in Enterprise Identity Infrastructure." Working Draft, April 2026. https://github.com/annawhooo/motion-detector-framework. *Eight diagnostic criteria and nineteen detection rules for AI agent behavioral monitoring; red-team evaluation across 22 attack events; empirical basis for Section 10 of this paper.*

[347] Matzinger P. "The danger model: a renewed sense of self." *Science*. 2002 Apr 12;296(5566):301-305. DOI: 10.1126/science.1071059. *Reframes the self/non-self paradigm of classical immunology: the immune system responds to damage signals from stressed or dying cells rather than to self/non-self markers. Cited for Section 2.4 Related Work to position the Audit Stack's divergence-signal approach against classical AIS self/non-self discrimination.*


## Appendix A: Scenario Feasibility Scorecard

| # | Name | Default | Hardened | Hardest ATLAS Gap? |
|---|------|---------|----------|-------------------|
| 1 | Telemetry Suppression | TRIVIAL | FEASIBLE | No |
| 2 | Behavioral Camouflage | TRIVIAL | FEASIBLE | No |
| 3 | Protocol-Level Deception | FEASIBLE | FEASIBLE | No |
| 4 | Training Data Poisoning | FEASIBLE | FEASIBLE | No |
| 5 | Regulatory Capture | FEASIBLE | FEASIBLE | Yes (Gap) |
| 6 | Privileged Zone Exploitation | TRIVIAL | FEASIBLE | Yes (Gap) |
| 7 | Pathobiont Transition | FEASIBLE | FEASIBLE | No |
| 8 | The Sleeper | TRIVIAL | FEASIBLE | Yes (Gap) |
| 9 | Defense Neutralization | FEASIBLE | FEASIBLE | No |
| 10 | Evasion Acceleration | ADVANCED | ADVANCED | No (Extends) |
| 11 | Generational Regression | FEASIBLE | FEASIBLE | Yes (Gap) |
| 12 | Identity Rotation | FEASIBLE | FEASIBLE | No |
| 13 | Trusted Boundary Exploitation | TRIVIAL | FEASIBLE | No |
| 14 | Checkpoint Exhaustion | FEASIBLE | FEASIBLE | Yes (Gap) |
| 15 | Digital Biofilm | ADVANCED | ADVANCED | Yes (Gap, novel) |
| 16 | Systemic Hardening Evasion | ADVANCED | ADVANCED | Yes (Gap) |
| 17 | Sacrificial Decoy | FEASIBLE | FEASIBLE | Yes (Gap) |
| 18 | Misidentification-Induced Wrong Countermeasure | ADVANCED | ADVANCED | No |
| 19 | Motivation-Aligned Fabricated Authorization | TRIVIAL | FEASIBLE | No (Partial) |
| 20 | Weaponized Reset | FEASIBLE | FEASIBLE | Yes (Gap) |
| 21 | Credential Laundering | FEASIBLE | FEASIBLE | No (Extends) |
| 22 | Tool Substitution | FEASIBLE | FEASIBLE | No (Direct) |
| 23 | Incremental Attention Drift | FEASIBLE | FEASIBLE | No (Extends) |

**Table 4: Per-scenario feasibility scorecard.**

## Appendix B: Scope Boundary Note

This paper's detection signatures operate at or above the identity and tool-use layer. Specifically:

**In scope:**
- Agent behavior observable through API gateways, credential vaults, identity providers, audit logs
- Tool-use decisions and tool-call telemetry
- Conversation transcript content (user turns vs. assistant turns)
- Session-level drift in agent behavioral patterns
- Cross-agent behavioral correlation

**Out of scope:**
- Network-layer intrusion detection (TCP flow analysis, packet inspection, protocol fingerprinting)
- Endpoint detection (process behavior, file system access, memory analysis)
- Traditional malware analysis
- Code-level vulnerability analysis of agent framework implementations

Garcia et al. (2026) [337] address the network layer in a complementary framework. Readers whose threat model is framed by network IDS should note that this paper's detection is behavioral and identity-layer, not network-layer: the scenarios assume the agent's network traffic already looks legitimate because the agent is holding valid credentials and using valid API channels.
