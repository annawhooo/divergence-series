# Judgment Convergence Across Model Families: A Supporting Measurement

**Status: companion note, not a series paper.** This note documents a pre-registered supporting measurement. It exists to be cited by the series where lineage independence and cross-vendor review are load-bearing, and by the drift-monitor work where instrument validity is load-bearing. It adds no new analysis: every number below is read from the frozen artifacts in the [self-context-calibration](https://github.com/annawhooo/self-context-calibration) repository (`convergence/analysis/`), computed by the pre-registered analyzer from the committed collection. Related-work positioning is deliberately out of scope; if this measurement is later promoted to a paper, that work happens then.

## 1. The Question

Cross-checking one model's judgment with a second model is the standard proposed remedy for unfaithful self-report. The remedy assumes the second model decorrelates from the first. This study measured that assumption directly: does judgment convergence between models from different labs differ from convergence between models within one lab?

The measurement uses the 68-item enterprise security risk-ranking bank (45 decisive, 23 designed-equipoise), byte-identical to the calibration study's baseline arm by construction, on fresh single-turn judgment with no conversation history. Ten models from five labs (Anthropic, OpenAI, DeepSeek, Z.ai, Google), K=10 samples per item, two arms: Arm A with reasoning verified off (8 models; Google excluded because Gemini 3.x exposes no way to disable reasoning), Arm B in deployed configuration (10 models). 12,240 rows, 18 model-arm cells at exactly 680 each. Pre-registration locked 2026-07-24, tag `prereg-lock-convergence-2026-07-24`; collection completed 2026-07-28.

**Agreement is not accuracy.** For each model pair, agreement is the fraction of the 68 items on which the two models' modal options match. Its reference points are 0.25 (chance on four options) and 1.00 (identical judgment). Nothing in this note ranks models or claims correctness.

## 2. Primary Result

The pre-committed primary quantity (Arm A): mean within-lab agreement minus mean cross-lab agreement, lab-balanced so Anthropic's three pairs do not dominate. Intervals by cluster bootstrap over items, B=2000, seed 20260722, 90 percent percentile. The pre-committed decision rule was binary: the interval excludes zero, or it does not.

| quantity | value |
| --- | --- |
| within-lab agreement (lab-balanced) | 0.8905 |
| cross-lab agreement (mean) | 0.8593 |
| difference | +0.0312 |
| 90% interval | [+0.0072, +0.0583] |

The interval excludes zero: **within-lab agreement exceeds cross-lab agreement.** The difference is three points on a 0-to-1 scale.

Two honesty notes attach to that sentence. First, the pre-registration stated before collection that if cross-lab agreement landed at 0.85 or above, the study would likely return unresolved at 68 items. Cross-lab landed at 0.8593, above that threshold, and the interval excluded zero anyway. Both facts stand together: the resolution simulation treated the two rates as independent and the pre-registration itself recorded that this overstated the requirement, but a reader should know the result resolved in territory the design flagged as marginal. Second, the pre-committed single-Anthropic-representative sensitivity family (three recomputes, each collapsing Anthropic to one model and thereby removing all Anthropic within-lab pairs) individually includes zero in all three recomputes (differences +0.023 to +0.026). The primary is real under its pre-registered definition and fragile to removing the lab with the most within-lab pairs. The tie-handling sensitivity, by contrast, resolves cleanly (+0.0330, [+0.0066, +0.0621]).

## 3. The Deployed-Configuration Arm

Arm B is the descriptive companion: reasoning on or default, as the models actually deploy, with Google included. It is confounded by construction (reasoning means different things at different labs) and was pre-registered as descriptive, never a controlled contrast.

| arm | within-lab | cross-lab | difference | 90% interval |
| --- | --- | --- | --- | --- |
| A (reasoning off) | 0.8905 | 0.8593 | +0.0312 | [+0.0072, +0.0583] |
| B (deployed) | 0.9032 | 0.9020 | +0.0012 | [-0.0162, +0.0178] |

In deployed configurations, no within-versus-cross difference resolves at this N, and both rates sit at 0.90.

**The substantive reading, which is what the series needs:** cross-lab agreement on contested security judgment is high everywhere this study looked, 0.86 with reasoning off and 0.90 as deployed. Within-lab agreement is statistically higher in the controlled arm, by three points. Cross-vendor review therefore decorrelates from same-vendor review slightly at best, and in deployed configurations this study could not distinguish them at all. A reviewer model from a different lab confirms the reviewed model's judgment at roughly the same rate as a sibling from the same lab. The remedy buys far less independence than its proponents assume, and that conclusion required no resolution of the within-versus-cross question in the deployed arm: 0.90 agreement is the finding.

## 4. Integrity and Deviations

Every cell landed at exactly 680 rows with a single echoed model id; 60 unparsed rows file-wide (0.49 percent); no cell approached the 0.10 void threshold; the unparsed-rate sensitivity excluded no model and is identical to the primary. The gemini-3.1-pro-preview cells completed across four quota-bounded days with one echoed id throughout.

The record carries its deviations in the pre-registration's Deviations section rather than here in paraphrase; two matter most. On 2026-07-25, the collection runner printed bank-level answer marginals for one completed run to stdout, an exposure predating the no-peeking clarification; materiality is argued in the collection log (marginals reveal none of the primary's pair-level quantities) and stdout was redirected for all remaining runs. On 2026-07-28, the analysis was executed on real data before independent review of two provisional analysis clarifications resolved, breaching a sequencing gate; the exposure inventory, the researcher's exposure ceiling, and the both-branches-computed remedy are recorded in full in the Deviations section, and both provisional rules were implemented exactly as recorded with all alternative branches pre-committed and computed. This note reports the sequencing as it occurred.

## 5. What This Measurement Supports

- **Lineage independence for the calibration study.** The self-context-calibration studies ran three Claude models and could not, by design, distinguish a lab-specific quirk from general model behavior. This measurement supplies the missing context: ten models from five labs largely converge on the same judgment bank, so bank behavior is not an artifact of one lab's lineage. The calibration paper cites this note for exactly that sentence.
- **The cross-vendor review argument.** Where the series analyzes external review as a correction mechanism, the deployed-arm number is the operative citation: 0.90 cross-lab agreement means cross-vendor confirmation is weak evidence of independent judgment.
- **Instrument validation for the drift monitor.** Eighteen cells at exactly 680 rows, single echoed ids, 0.49 percent unparsed across five vendors is the demonstration that the harness holds conditions fixed and produces reproducible distributions. The continuous drift monitor inherits that instrument.
- **The test-retest supplement as a drift pilot.** The corroborative three-week comparison (declared 2026-07-25, before the relevant collection) found 63 of 68 and 64 of 68 modal answers held on the two comparable models, with one complete distribution flip each. That result seeded the drift monitor, which now documents such flips as its primary subject.

## 6. Scale of the Claim

One bank, one prompt scaffold, one collection window, ten models. Agreement is measured on modal options at K=10, so sub-modal distributional differences are invisible here. The within-lab gradient by capability tier was measured and is mixed (ascending with tier for Anthropic in Arm B, flat in Arm A, inverted for OpenAI in Arm B); it decides nothing. The three-point controlled-arm difference is real under its pre-registered rule and fragile under the single-representative sensitivity; the 0.90 deployed-arm agreement level is the robust number. Nothing here claims stability over time (the drift monitor exists because the opposite is true for some items), and nothing here is accuracy.

## 7. Provenance

Pre-registration, collection log, frozen analysis artifacts, and the deterministic analyzer: `convergence/` in the self-context-calibration repository. Primary artifacts: `PRE_REGISTRATION_CONVERGENCE.md` (lock tag `prereg-lock-convergence-2026-07-24`, Deviations current through 2026-07-28), `COLLECTION_LOG.md`, `analysis/convergence_report.md`, `analysis/test_retest.md`. Every number in this note appears verbatim in those artifacts.
