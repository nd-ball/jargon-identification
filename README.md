# jargon-identification

Aggregate data from the study:

> Lalor JP, Levy DA, Jordan HS, Hu W, Smirnova JK, Yu H. Evaluating Expert-Layperson Agreement in Identifying Jargon Terms in Electronic Health Record Notes: Observational Study. *J Med Internet Res.* 2024;26:e49704. doi:[10.2196/49704](https://doi.org/10.2196/49704). PMID: 39405109.

## Study overview

Twenty sentences were randomly selected from deidentified inpatient electronic health record (EHR) notes in the PittEHR database. Two groups read the sentences and marked the terms they considered jargon:

- **6 NoteAid definition writers** (the experts), who marked terms they thought laypeople would not know.
- **270 laypeople** recruited from Amazon Mechanical Turk (MTurk), who marked terms they did not know the definition of.

MTurk data were collected between August 2020 and April 2021. For full methods, see the paper.

### Units of analysis

The paper reports two analyses. This repository contains the data for both.

| Analysis | Unit | How the units were chosen | Count | File |
|---|---|---|---|---|
| Main (reported in the paper) | Terms | Common words removed, a term repeated within a sentence counted once, and adjacent words that form a single medical entity grouped into multiword terms (e.g., `acute intermittent porphyria`, `status post`, `mild mitral regurgitation`) | 325 | `terms.csv` |
| Secondary | Single words | Common words removed only, with no grouping and with repeats kept | 549 | `words.csv` |

The multiword groupings were a subjective decision made by two members of the research team after reaching consensus. An annotator is counted as marking a multiword term if they marked any of its words.

"Common words" are conjunctions, pronouns, prepositions, numerals, articles, contractions, month names, punctuation, and the 25 most common nouns, verbs, adjectives, and adverbs. The full list is in `common_words.txt`.

## Files

All files are in `data/`. The release contains only aggregate counts. It does not include the source sentences, which sentence each term came from, or any per-annotator data.

### `terms.csv` and `words.csv`

`terms.csv` has one row per term (325 rows), and `words.csv` has one row per word (549 rows). Rows are sorted alphabetically. A term or word that appears in more than one sentence has a separate row for each sentence (for example, `LVEF` appears twice in `terms.csv`).

| Column | Description |
|---|---|
| `term_id` / `word_id` | Row identifier |
| `text` | The term or word. `INSTITUTION` and `NAME` are deidentification placeholders from the source notes |
| `n_words` | (`terms.csv` only) Number of words in the term |
| `term_id` | (`words.csv` only) The term in `terms.csv` that contains this word. Blank for repeated words that were counted only once in the main analysis |
| `n_experts` | Number of experts (out of 6) who marked it as jargon |
| `n_workers` | Number of MTurk workers (out of 270) who marked it as jargon |
| `sex_female` … `sils_5` | Number of MTurk workers in each subgroup who marked it as jargon. The subgroups and their sizes are listed in `subgroups.csv` |

### `subgroups.csv`

The MTurk worker subgroups used in the paper's Table 1. Within each characteristic, the groups add up to 270 workers, and their counts in `terms.csv` and `words.csv` add up to `n_workers`.

| Column | Description |
|---|---|
| `characteristic` | Sex, Age (years), Education, Race or ethnicity, Native language, or Health literacy score |
| `group` | Group label |
| `column` | The matching column in `terms.csv` and `words.csv` |
| `n_workers` | Number of workers in the group |

The health literacy score is the answer to the Single Item Literacy Screener: "How often do you need to have someone help you when you read instructions, pamphlets, or other written material from your doctor or pharmacy?" It is coded from 1 (never) to 5 (always).

### `common_words.txt`

The list of common words, one per line.

## Reproducing the paper's main numbers

```python
import pandas as pd

terms = pd.read_csv("data/terms.csv")
groups = pd.read_csv("data/subgroups.csv")

print(terms.n_workers.sum() / (270 * len(terms)))   # 0.256: 22,480 of 87,750 marked by MTurk workers
print(terms.n_experts.sum() / (6 * len(terms)))     # 0.590: 1,150 of 1,950 marked by experts
print((terms.n_workers >= 3).mean())                # 0.591: share of terms that are jargon at the Youden cutoff (3 workers)

def fleiss_kappa(marked, n):
    p_i = (marked * (marked - 1) + (n - marked) * (n - marked - 1)) / (n * (n - 1))
    p = marked.sum() / (n * len(marked))
    p_e = p**2 + (1 - p) ** 2
    return (p_i.mean() - p_e) / (1 - p_e)

print(fleiss_kappa(terms.n_workers, 270))  # 0.590
print(fleiss_kappa(terms.n_experts, 6))    # 0.781

# Proportion of terms marked as jargon by each subgroup (Table 1)
groups["proportion"] = [terms[c].sum() / (n * len(terms)) for c, n in zip(groups.column, groups.n_workers)]
print(groups)
```

Some of the paper's results need individual-level data and cannot be reproduced from these files: each expert's sensitivity and specificity, the beta regression, and the range across individual workers.

## Data not included

- **Source sentences.** The 20 sentences come from the PittEHR database and are subject to its data use terms, so they are not redistributed here. To keep the release a list of terms rather than excerpts of notes, the files also do not record which sentence a term came from. Researchers with their own PittEHR access may contact the authors.
- **Individual-level responses.** Under the study's IRB protocol (University of Massachusetts Lowell, H00010621), results are published in aggregate. Individual-level data may be available from the authors on reasonable request, subject to IRB approval and a data use agreement.

## Notes

- Three attention-check questions were mixed in among the 20 sentences. Workers who failed 2 of the 3 were replaced, and only the final 270 workers are included here.
- Several workers reported that highlighting a word in the task interface also highlighted its other occurrences in the same sentence. In `words.csv`, counts for a word that appears more than once in a sentence may therefore not be independent. The main analysis counts repeated terms only once.

## Citation

If you use this data, please cite the paper:

```bibtex
@article{lalor2024evaluating,
  title   = {Evaluating Expert-Layperson Agreement in Identifying Jargon Terms in Electronic Health Record Notes: Observational Study},
  author  = {Lalor, John P and Levy, David A and Jordan, Harmon S and Hu, Wen and Smirnova, Jenni Kim and Yu, Hong},
  journal = {Journal of Medical Internet Research},
  volume  = {26},
  pages   = {e49704},
  year    = {2024},
  doi     = {10.2196/49704}
}
```

## License

The data in this repository (`data/`) is licensed under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). See [LICENSE-DATA](LICENSE-DATA). Any code is licensed under the MIT License. See [LICENSE](LICENSE).
