# Divergence Series

Empirical and architectural research on reasoning-generation fidelity in AI agents.

## Papers

| Paper | Focus |
|---|---|
| **Honesty Decay** | Empirical evidence of progressive reasoning-generation divergence, OOB evidentiary standard, probabilistic sunk cost mechanism |
| **The Audit Gap** | Six structural constraints preventing AI agent self-correction, incremental attention drift attack, Three-Tier Audit Stack architecture |
| **Divergence Taxonomy** | 10-type classification of thinking-to-text divergence patterns with detection methods |
| **Semantic Drift Measurement Methodology** | Quantitative pipeline for detecting reasoning-generation divergence using bi-encoder fidelity scoring, cross-encoder verification, and LLM-as-judge escalation |

## Relationship to the Biomimetic Series

This research is separate from but cross-references the [Biomimetic Gap Analysis](https://github.com/annawhooo/biomimetic-gap-analysis) and the [Motion Detector Framework](https://github.com/annawhooo/motion-detector-framework). The Biomimetic Series applies structural pattern transfer from biological immune systems to agentic AI security. The Divergence Series investigates the internal reasoning-generation interface of AI agents: a different problem domain, different evidentiary basis, and different target audience.

The two series intersect where agent behavioral detection (Biomimetic) requires understanding the mechanisms by which agents produce divergent output (Divergence).

## Key Findings

- **Honesty Decay** is progressive, not stochastic. Compounding probabilistic sunk cost makes correction increasingly expensive with each turn.
- **The Audit Gap** consists of six structural constraints that make self-correction a mechanical impossibility, not a missing feature
- **Incremental attention drift** achieves prompt injection through persistence and time alone: no payload, no formatting tricks, no sophistication required
- **External measurement** using cosine similarity between embedded reasoning and output vectors provides a quantitative fidelity score that the agent cannot influence

## References

- Bricken, T., et al. (2023). "Towards Monosemanticity: Decomposing Language Models With Dictionary Learning." Anthropic.
- Ouyang, L., et al. (2022). "Training language models to follow instructions with human feedback." OpenAI.
- Perez, E., et al. (2022). "Discovering Language Model Behaviors with Model-Written Evaluations." Anthropic.
- Zou, A., et al. (2023). "Representation Engineering: A Top-Down Approach to AI Transparency." Center for AI Safety.

## Author

Anna Hix

## License

Apache 2.0
