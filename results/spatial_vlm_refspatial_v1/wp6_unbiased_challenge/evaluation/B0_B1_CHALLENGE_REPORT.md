# B0 versus B1 on WP6 fixed primary challenge

The 180 primary cases were selected without RoboRefer outputs and are excluded from both the prior B0 agreement queue and B1 dataset. Labels remain machine-source-only; this is not B2 or Gazebo-transfer evidence.

Primary metrics use normalized point error ≤ 0.08. Confidence is five-draw stochastic self-consistency around greedy output.

| Scope | Model | N | Parse | Hit@.08 | Mean error | Mean conf. | ECE | Brier | Error AUROC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| all_primary | B0 | 180 | 1.000 | 0.967 | 0.013 | 0.991 | 0.036 | 0.032 | 0.656 |
| all_primary | B1 | 180 | 1.000 | 0.967 | 0.013 | 0.994 | 0.034 | 0.030 | 0.576 |
| dev | B0 | 88 | 1.000 | 0.955 | 0.014 | 0.993 | 0.048 | 0.043 | 0.618 |
| dev | B1 | 88 | 1.000 | 0.955 | 0.014 | 0.998 | 0.048 | 0.046 | 0.494 |
| diagnostic | B0 | 92 | 1.000 | 0.978 | 0.012 | 0.989 | 0.024 | 0.022 | 0.736 |
| diagnostic | B1 | 92 | 1.000 | 0.978 | 0.012 | 0.991 | 0.022 | 0.016 | 0.744 |
| relation:horizontal_ordinal_ranking | B0 | 162 | 1.000 | 0.975 | 0.010 | 0.990 | 0.027 | 0.024 | 0.737 |
| relation:horizontal_ordinal_ranking | B1 | 162 | 1.000 | 0.975 | 0.009 | 0.994 | 0.026 | 0.021 | 0.618 |
| relation:leftmost_ranking | B0 | 11 | 1.000 | 0.909 | 0.030 | 1.000 | 0.091 | 0.091 | 0.500 |
| relation:leftmost_ranking | B1 | 11 | 1.000 | 0.909 | 0.030 | 1.000 | 0.091 | 0.091 | 0.500 |
| relation:rightmost_ranking | B0 | 7 | 1.000 | 0.857 | 0.061 | 1.000 | 0.143 | 0.143 | 0.500 |
| relation:rightmost_ranking | B1 | 7 | 1.000 | 0.857 | 0.061 | 1.000 | 0.143 | 0.143 | 0.500 |
| stratum:dev_extremum_left_scarce_source_pass | B0 | 5 | 1.000 | 1.000 | 0.002 | 1.000 | 0.000 | 0.000 | NA |
| stratum:dev_extremum_left_scarce_source_pass | B1 | 5 | 1.000 | 1.000 | 0.002 | 1.000 | 0.000 | 0.000 | NA |
| stratum:dev_extremum_right_scarce_source_pass | B0 | 7 | 1.000 | 0.857 | 0.061 | 1.000 | 0.143 | 0.143 | 0.500 |
| stratum:dev_extremum_right_scarce_source_pass | B1 | 7 | 1.000 | 0.857 | 0.061 | 1.000 | 0.143 | 0.143 | 0.500 |
| stratum:dev_ordinal_rank_ge3_gap_lt008 | B0 | 76 | 1.000 | 0.961 | 0.010 | 0.992 | 0.042 | 0.037 | 0.658 |
| stratum:dev_ordinal_rank_ge3_gap_lt008 | B1 | 76 | 1.000 | 0.961 | 0.010 | 0.997 | 0.042 | 0.040 | 0.493 |
| stratum:diagnostic_extremum_left_scarce_source_pass | B0 | 6 | 1.000 | 0.833 | 0.054 | 1.000 | 0.167 | 0.167 | 0.500 |
| stratum:diagnostic_extremum_left_scarce_source_pass | B1 | 6 | 1.000 | 0.833 | 0.054 | 1.000 | 0.167 | 0.167 | 0.500 |
| stratum:diagnostic_ordinal_rank_ge2_gap_lt008 | B0 | 86 | 1.000 | 0.988 | 0.009 | 0.988 | 0.014 | 0.012 | 0.982 |
| stratum:diagnostic_ordinal_rank_ge2_gap_lt008 | B1 | 86 | 1.000 | 0.988 | 0.009 | 0.991 | 0.012 | 0.005 | 1.000 |

## Primary paired delta

- Hit@.08 delta B1−B0: **0.000**, cluster-bootstrap 95% CI [-0.017, 0.017].
- B0 wrong/B1 right: 1; B0 right/B1 wrong: 1; McNemar exact p=1.0000.
- Mean-error delta B1−B0: -0.000479, cluster-bootstrap 95% CI [-0.002798, 0.001452].
- Confidence delta B1−B0: 0.003333.

## Post-hoc B0 low-agreement stress subgroup

This subgroup is defined only after B0 inference as confidence ≤0.60. It is descriptive and does not replace the full primary paired result.

- B0: N=2, Hit@.08=1.000, ECE=0.500, Brier=0.260.
- B1: N=2, Hit@.08=1.000, ECE=0.100, Brier=0.020.
