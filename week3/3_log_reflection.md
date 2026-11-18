# AI use and reflection log

## Prompt used

"Interpret this table for a short referee's note. Separate observations from possible explanations and causal claims. Do not claim that the data proves why prices changed."

## AI interpretation excerpt

The four items increased between the first and last dates in the supplied file. Sweetener D had the largest percentage increase (19.41%), while Pulse C had the smallest (3.69%). These are descriptive comparisons. The data does not identify the reasons for the changes, and the file is labelled illustrative teaching data rather than actual PBS observations.

## Audit table

| AI statement | Colour | Check | Decision |
|---|---|---|---|
| All four item averages increased from the first to last date. | Green | Recalculated the first and last city averages for each item. | Accepted |
| Sweetener D had the largest increase at 19.41%. | Green | Recalculated percentage change using (last - first) / first × 100. | Accepted |
| Pulse C had the smallest increase at 3.69%. | Green | Recalculated the same formula for all four items. | Accepted |
| Supply or demand may help explain the increases. | Amber | These factors are possible explanations, but the CSV has no evidence about them. | Kept only as possibilities |
| A policy or event caused the price increases. | Red | No causal evidence or comparison group is provided. | Rejected |
| The data represents actual PBS prices. | Red | The CSV explicitly says it is illustrative teaching data, not actual PBS observations. | Rejected |

## Sensitivity check

The CSV does not identify a flagged value. I used the Staple A, City 1 observation of 99.5 on 8 January as a possible low value for a simple sensitivity check. This is an assumption for the exercise, not a confirmed data error.

| Version | Observations | Mean | Median |
|---|---:|---:|---:|
| With all observations | 24 | 111.17 | 111.75 |
| Without 99.5 | 23 | 111.67 | 112.50 |

Removing this observation increases both the mean and median slightly. There is no basis in the file to delete it from the main results, so the original data is retained.

## Classmate discussion (AI role-play)

**Me:** The mean is higher than the median in some cases. Does that mean the mean is wrong?

**Classmate:** No. They measure the centre differently. The mean uses every value, while the median is the middle value after sorting. A difference between them does not automatically make either one wrong.

**Me:** Can we say why the prices went up?

**Classmate:** Not from this table. We can describe the increase, but we would need other information to explain its cause.

## Reflection

My first draft could have gone too far by making the increases sound like proof of inflation across the whole market. The file only covers four teaching items, two cities and twelve dates. It is also marked as illustrative data. I kept the final note focused on what the numbers show and labelled possible explanations separately.
