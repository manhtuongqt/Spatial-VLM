# Original RoboRefer B1 vs best P-CRA-U V2

Paired exploratory evaluation on the same 400 dev samples; grounding uses the 168 truth-FOUND samples.

| Metric | Original B1 RGB-D | Best V2 | V2 - B1 |
|---|---:|---:|---:|
| Answerability accuracy | 42.00% | 81.50% | +39.50 pp |
| Answerability macro-F1 | 14.79% | 82.14% | +67.35 pp |
| Point in target (truth FOUND) | 95.24% (160/168) | 97.62% (164/168) | +2.38 pp |
| Point in interior (truth FOUND) | 95.24% (160/168) | 95.83% (161/168) | +0.60 pp |
| False FOUND: AMBIGUOUS + ABSENT | 100.00% | 4.21% | -95.79 pp |
| False FOUND: all non-FOUND | 100.00% | 7.76% | -92.24 pp |

Paired grounding: both hit 158, original-only 2, V2-only 6, neither 2.
Family-cluster bootstrap for V2-B1 target-hit delta: +2.38 pp (95% CI -0.62 to +5.71 pp).
V2 joint true-FOUND classification and target hit: 137/168 (81.55%).

The original model always emits a point, so it preserves FOUND recall but cannot reject ambiguous, absent, or insufficient-evidence requests. V2's main gain is selective answerability and uncertainty handling; its localization gain is smaller. The tradeoff is that V2 abstains or rejects some genuinely answerable samples.

These are exploratory dev results, not a held-out test or robot-success measurement.
