# Cleaning Log
Data: PDHS 2017-18 Kids' Recode (PKKR71FL.DTA), 12,708 children by 1,186 columns, of which 2,278 are in Sindh. The raw file in data/raw/ is never edited. Every change below is made in code on a copy, and new columns are added next to the originals instead of overwriting them.
1. Flagged height-for-age codes (changed)

When I summarised hw70 (height-for-age z-score), the maximum was 9998 and the standard deviation was 1,435, which is impossible for a z-score. Counting values of 9990 or more gave 85 children nationally, all coded 9998, and all 85 are labelled measured in hw13. In Sindh, 27 of the 806 scored children carry this code, so only 779 Sindh scores are usable. I decided to treat 9998 as missing in a new column, haz, because the DHS codebook labels it "flagged cases" and a flagged value cannot be trusted as a measurement. This changed 85 cells (27 in Sindh) and no rows were dropped, so the children remain available for the missing-data comparison below. I rejected two alternatives. Keeping 9998 as a real score would put a value 50 standard deviations from the mean into the outcome, and dropping the whole row would also discard the child's predictors for no benefit.

2. Scale of hw70 (changed)

After removing the flag codes, the 4,141 valid scores ran from -599 to 581, with a median of -153. These are z-scores multiplied by 100, so -258 means -2.58. I decided to create haz = hw70 / 100 and define stunting as haz < -2, while keeping the original hw70 column untouched. Dividing by 100 changed 4,141 cells in the new column and none in the original. After conversion the range is -5.99 to 5.81 with a mean of -1.55, which is a plausible shape for a z-score. I rejected leaving the values scaled and using a threshold of -200 directly, because every later chart and table would then show unitless numbers that a reader would take for z-scores.

3. "Don't know" coded as 98 in antenatal visits (changed)

The tail of the value counts for m14 (number of antenatal visits) shows 16, 17, 18, 20 and then 98, with 98 occurring 31 times. No mother plausibly had 98 visits, and the codebook labels 98 "don't know". I decided to set 98 to missing in a new column m14_clean, which changed 31 cells. I rejected leaving 98 as a number because it would make the mean number of visits meaningless, and I rejected dropping those children because there are only 31 of them and their other answers are fine.

4. Children with no score: missing, not imputed (decision not to change values)

8,482 of 12,708 children have no hw70, and 1,472 of those are in Sindh. The measurement-result column explains most of them: 7,490 are "no measurement found in household", 719 are "dead", 111 "not present", 86 "refused" and 72 "other". Because 719 of the missing are dead children who can never have a score, I compared the 4,226 scored children with the 7,763 living children without a score. The two groups look alike: 54.4% of scored children are rural against 56.1% of the others, 51.5% of their mothers have no education against 50.6%, and the wealth-quintile shares differ by at most 3.4 percentage points. The share of children with a score is also similar in every province, between 29% and 35% (Sindh 35%). I decided not to impute the height-for-age scores and to keep the missing children out of the modelling sample. The outcome is the thing I want to predict, so inventing it for 7,763 children would make the model learn from my assumptions. This check is only on residence, wealth and education, so I cannot rule out other differences.

5. Duplicate rows (decision not to change)

caseid appears 4,421 more times than it should if it were unique, but that is expected, because a mother with several children under five has one row per child. The real duplicate test is caseid plus midx (the child's index in the birth history), and it returned 0 repeated rows. I therefore removed nothing. I rejected dropping repeated caseid values to keep one child per mother, because it would discard thousands of real children and would still not remove the dependence between children of the same cluster. It does mean that rows are not independent, which is why the train/test split will hold out whole clusters (v001) instead of random rows.

6. Inconsistent measurement flags (decision not to change)

In total, 4,227 children are labelled measured in hw13, but 4,226 have a score. Two measured children have no score (both outside Sindh) and one child not labelled measured has a score. Three cells in hw13 are also empty (one of them in Sindh). I left all of these as they are, because they total only a handful of cells and none of them changes the Sindh sample size of 779. The score column, not the result column, defines my usable sample, so the label mismatch does not affect the analysis. I rejected overriding the DHS labels by hand because I have no evidence about which of the two columns is right.

7. Column types and interview months (decision not to change)

caseid is stored as text, which is right for an identifier, and v113 (water source) and v116 (toilet type) are stored as int8, although their values (11 to 97) are category codes and not quantities. I did not change any of the types in the cleaned file. I will treat the two columns as categorical when I build features, which does not require editing the data. The interview months run from January to April and from November to December only, with nothing for May to October. Nothing in the data can fix that, so I left it as a coverage limit and will list it under limitations.

The six things

Missing. Found. 8,482 of 12,708 height-for-age scores are missing (1,472 of 2,278 in Sindh), plus 85 flagged values that I convert to missing. I checked it by counting the measurement-result column and by comparing scored and unscored living children on residence, wealth and education (entries 1 and 4).

Duplicates. Not found as repeated rows: 0 duplicates on caseid plus midx. I checked it directly, and repeated caseid values are siblings (entry 5).

Wrong types. Found, minor. v113 and v116 are numeric codes for categories, and m14 mixes counts with a special code. I checked the dtypes and the value counts (entries 3 and 7).

Wrong units. Found. hw70 is stored as a z-score times 100. I checked it from the range and standard deviation and confirmed it after dividing by 100 (entry 2).

Encoding. Found. Special codes sit among real values: 9996, 9997 and 9998 in hw70 (only 9998 occurs) and 98 in m14. I checked value counts at the tail of each column (entries 1 and 3).

Drift. Not found over the survey period. The stunting rate among valid scores is 38.0% for children interviewed in 2017 and 38.2% in 2018. The check can only compare two survey years, because the survey is a single snapshot and the months May to October are not covered at all. As an outside check on the figures, the weighted stunting rate among the 779 usable Sindh scores is 49.3% (46.5% unweighted), close to the 50% in the PDHS 2017-18 factsheet.
