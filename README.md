# jargon-identification

Aggregate data from the study:

> Lalor JP, Levy DA, Jordan HS, Hu W, Smirnova JK, Yu H. Evaluating Expert-Layperson Agreement in Identifying Jargon Terms in Electronic Health Record Notes: Observational Study. *J Med Internet Res.* 2024;26:e49704. doi:[10.2196/49704](https://doi.org/10.2196/49704). PMID: 39405109.

## Study overview

We randomly selected 20 sentences from the PittEHR database of deidentified inpatient medical records. NoteAid definition writers and Amazon Mechanical Turk (MTurk) workers were shown the 20 sentences:

- 6 NoteAid definition writers (medical experts) were asked to identify terms that they considered to be jargon, that is, terms for which laypeople would not know the definition.
- 270 MTurk workers (laypeople) were asked to identify those terms for which they did not know the definition.

Data collection took place between August 2020 and April 2021. MTurk workers who worked in the health care field were excluded, and MTurk workers were oversampled to obtain equal numbers in each of the education subgroups and an equal number of non-White and White participants. See the paper for full methods.

### Units of analysis

| Analysis | Unit | Description | Count | File |
|---|---|---|---|---|
| Main analysis | Potential jargon terms | Common words excluded, terms repeated in a sentence counted once, and multiword terms analyzed as single terms | 325 | `terms.csv` |
| Second analysis | Potential jargon words | Only common words excluded, with no grouping of multiword terms | 549 | `words.csv` |

From the paper: "We considered multiword terms to be adjacent words that represented a distinct medical entity (examples include PR interval, internal capsule, and acute intermittent porphyria), terms that were routinely used together (examples include hemodynamically stable, status post, and past medical history), or terms that were modified by a minor word (examples include trace perihepatic fluid, mild mitral regurgitation, rare positive cells, and deep pelvis)." The grouping of multiword terms was determined by 2 members of the research team after reaching a consensus. In `terms.csv`, an annotator is counted as marking a multiword term if they marked any of its words.

Common words are all conjunctions, pronouns, prepositions, numerals, articles, contractions, names of months, punctuation, and the 25 most common nouns, verbs, adjectives, and adverbs, including their plural forms. The full list is in `common_words.txt`.

The paper reports results for the main analysis. The analyses were repeated using single words as the unit of analysis, and the results were not substantively different.

## Files

All files are in `data/`. The files contain aggregate counts only. They do not include the source sentences, which sentence each term came from, or any individual-level responses.

### `terms.csv` and `words.csv`

`terms.csv` has one row per term (325 rows), and `words.csv` has one row per word (549 rows). Rows are sorted alphabetically. A term that appears in more than one sentence has a separate row for each sentence (for example, `LVEF` appears twice in `terms.csv`).

| Column | Description |
|---|---|
| `term_id` / `word_id` | Row identifier |
| `text` | The term or word. `INSTITUTION` and `NAME` are deidentification placeholders from the source notes |
| `n_words` | (`terms.csv` only) Number of words in the term |
| `term_id` | (`words.csv` only) The term in `terms.csv` that contains this word. Blank for repeated words that were counted only once in the main analysis |
| `n_experts` | Number of NoteAid definition writers (out of 6) who identified it as jargon |
| `n_workers` | Number of MTurk workers (out of 270) who identified it as jargon |
| `sex_female` … `sils_5` | Number of MTurk workers in each subgroup who identified it as jargon. The subgroups and their sizes are listed in `subgroups.csv` |

### `subgroups.csv`

The MTurk worker subgroups in Table 1 of the paper. Within each characteristic, the group sizes sum to 270, and the group columns in `terms.csv` and `words.csv` sum to `n_workers`.

| Column | Description |
|---|---|
| `characteristic` | Sex, Age (years), Education, Race or ethnicity, Native language, or Health literacy score |
| `group` | Group label |
| `column` | The matching column in `terms.csv` and `words.csv` |
| `n_workers` | Number of MTurk workers in the group |

Health literacy was measured with the Single Item Literacy Screener: "How often do you need to have someone help you when you read instructions, pamphlets, or other written material from your doctor or pharmacy?" Scores range from 1 (Never) to 5 (Always).


### `common_words.txt`

The list of common words, one per line.

## Reproducing results from the paper

```python
import pandas as pd

terms = pd.read_csv("data/terms.csv")
groups = pd.read_csv("data/subgroups.csv")

# Proportion of terms identified as jargon
print(terms.n_workers.sum() / (270 * len(terms)))   # 0.256 (22,480/87,750), MTurk workers
print(terms.n_experts.sum() / (6 * len(terms)))     # 0.590 (1150/1950), NoteAid definition writers

# Fleiss kappa
def fleiss_kappa(marked, n):
    p_i = (marked * (marked - 1) + (n - marked) * (n - marked - 1)) / (n * (n - 1))
    p = marked.sum() / (n * len(marked))
    p_e = p**2 + (1 - p) ** 2
    return (p_i.mean() - p_e) / (1 - p_e)

print(fleiss_kappa(terms.n_workers, 270))  # 0.590, MTurk workers
print(fleiss_kappa(terms.n_experts, 6))    # 0.781, NoteAid definition writers

# Mean sensitivity and specificity of the definition writers, with a term
# classified as jargon if at least 3 MTurk workers identified it (Youden index cutoff)
jargon = terms.n_workers >= 3
print(jargon.mean())                                                    # 0.591 of terms classified as jargon
print(terms.n_experts[jargon].sum() / (6 * jargon.sum()))               # 0.917 sensitivity
print((6 - terms.n_experts[~jargon]).sum() / (6 * (~jargon).sum()))     # 0.882 specificity

# Proportion of terms identified as jargon by each MTurk worker subgroup (Table 1)
groups["proportion"] = [terms[c].sum() / (n * len(terms)) for c, n in zip(groups.column, groups.n_workers)]
print(groups)
```

The following results need individual-level data and cannot be reproduced from these files: the results for individual definition writers, the range across individual MTurk workers, and the beta regression (Table 2).

## Data not included

- **Source sentences.** The 20 sentences are from the PittEHR database and are not included.
- **Individual-level responses.** Responses from individual definition writers and MTurk workers are not included.

## Notes

- Interspersed among the 20 sentences were 3 attention-check questions. If a participant answered 2 of the 3 attention checks incorrectly, the participant's responses were discarded and the participant was replaced. The data include only the final 270 MTurk workers.
- One MTurk worker reported that highlighting a word in the task interface also highlighted the other occurrences of that word in the same sentence. In `words.csv`, counts for a word that occurs more than once in a sentence may therefore not be independent. The main analysis counts terms repeated in a sentence only once.

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
