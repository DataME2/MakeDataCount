# Make Data Count Project in Kaggle
# Finding & Classifying Dataset Citations in Research Papers (PDF/XML)

Automated pipeline to **find, normalize, and classify dataset citations** in scholarly articles (PDF/XML).  
It targets **data DOIs** (Dryad, Zenodo, Figshare, PANGAEA, Mendeley Data…) and **accessions** (GEO/SRA/PRJNA/PXD, EPI_ISL, etc.), then labels each mention as **Primary** or **Secondary**.

> Built for large-scale processing and evaluation (F1). Includes Unicode-safe parsing, robust normalization, and detailed diagnostics.

---

## ✨ Key Features

- **Parsing**: PDF/XML → text (with Unicode sanitization to remove zero-width chars, soft hyphens, odd dashes).
- **Extraction**: Targeted regex for **data DOIs** and **accession IDs**.
- **Normalization**:
  - ✅ Real DOIs → `https://doi.org/<lowercased-doi>`
  - ✅ Accessions kept as accessions (no fake `doi.org/GSE...` conversions)
  - ✅ Tight DOI cleaner trims glued tokens like `…d1487447ethics` → `…d1487447`
- **Classification**: Rule-based + (optional) ML model ⇒ **Primary** vs **Secondary**.
- **Diagnostics**: DOI prefix stats, type distribution, lost-candidate analysis.
- **Evaluation**: TP/FP/FN, Precision/Recall/F1 when ground truth is available.

---

## 📦 Repo Structure (suggested)

# Important Behaviors

* **Unicode-safe parsing first, regex second**

  * Remove zero-width chars (U+200B), soft hyphen (U+00AD), BOM, bidi marks.
  * Normalize fancy dashes (–—−) → `-`.
  * **Preserve spaces** (don’t glue words together).

* **Tight DOI detection + cleanup**

  * Use a strict pattern: `10.\d{4,9}/[^\s"'<>)}\]]+`
  * Post-clean: trim trailing punctuation and any glued trailing words (e.g., `…d1487447ethics` → `…d1487447`).

* **Robust normalization**

  * ✅ Real DOIs → `https://doi.org/<lowercased-doi>`
  * ✅ Accessions stay as accessions (e.g., `GSE110720`, `PRJNA123456`, `PXD012345`, `EPI_ISL_1234567`)
  * ❌ Never “doi-ify” accessions (no `https://doi.org/GSE…`).

* **Primary vs Secondary classification**

  * Hybrid rules (section/context aware) + optional ML model.
  * References/“cited in” contexts tend to **Secondary**; data-availability statements tend to **Primary**.

* **Submission format**

  * Final CSV has **exactly**: `article_id,dataset_id,type` (no `context`).
  * CSV written with `utf-8-sig` for Excel friendliness.
  * De-duplicate on `(article_id, dataset_id, type)`.

# Evaluation

* **Inputs**

  * `submission.csv`: `article_id,dataset_id,type`
  * `train_labels.csv` (ground truth, train split only): same columns

* **Normalization before comparison**

  * DOIs lowercased; accessions uppercased as extracted.
  * Drop duplicates on `(article_id,dataset_id,type)`.

* **Scoring (exact-match on triples)**

  * `TP`: inner join on the three columns.
  * `FP`: submission minus ground truth.
  * `FN`: ground truth minus submission.
  * `precision = TP / (TP + FP)`
  * `recall    = TP / (TP + FN)`
  * `f1        = 2PR / (P + R)`

* **Polars/Pandas interop**

  * Convert to Polars for join-based counts (or to Pandas if your evaluator is Pandas-based).
  * Print a clear report: TP/FP/FN + Precision/Recall/F1.

# Example Metrics

*(From a recent run; your numbers will vary by data slice and settings.)*

* Papers parsed: **924**
* Dataset mentions detected: **767** (from **250** articles)
* Type distribution: **Primary 640** • **Secondary 127**
* Top data DOI prefixes:

  * `10.5061` (Dryad)
  * `10.5281` (Zenodo)
  * `10.6084` (Figshare)
  * `10.1594` (PANGAEA)
  * `10.17632` (Mendeley Data)

# Troubleshooting

* **Weird characters in DOIs (e.g., `â€‹`)**

  * Cause: Zero-width space from PDF extraction.
  * Fix: Unicode sanitizer drops U+200B/U+2060/U+00AD before regex.

* **DOIs with glued words (`…d1487447ethics`)**

  * Cause: Tokenization artifacts.
  * Fix: Tight DOI pattern + post-cleaner to trim long trailing alpha tokens.

* **Spaces disappeared; words glued together**

  * Cause: Over-aggressive sanitizer removing spaces.
  * Fix: Only remove control/zero-width/bidi; **keep** normal spaces and collapse multiple spaces to one.

* **Accessions converted to `https://doi.org/…`**

  * Cause: Normalizer doi-ifies everything.
  * Fix: Separate DOI detection from accession detection; only real DOIs get a `doi.org/` URL.

* **Context column appears in submission**

  * Cause: Didn’t project columns before saving.
  * Fix: Select `['article_id','dataset_id','type']` right before `to_csv`.

* **F1 report not printed / `.select` error**

  * Cause: Evaluator expecting Polars but received Pandas (or vice-versa).
  * Fix: Convert frames to the evaluator’s expected type before joins and print TP/FP/FN/Precision/Recall/F1.

* **Duplicate rows in submission**

  * Cause: Multiple passes adding same candidate.
  * Fix: `drop_duplicates` / `unique` on `(article_id, dataset_id, type)` right before writing CSV.

# Output Sample

**`submission.csv`**

```csv
article_id,dataset_id,type
0001,https://doi.org/10.5061/dryad.4mw6m90cz,Primary
0001,GSE135055,Secondary
0002,https://doi.org/10.5281/zenodo.8262592,Primary
0003,PRJEB27670,Primary
0004,https://doi.org/10.6084/m9.figshare.12764126.v13,Primary
0004,EPI_ISL_445091,Secondary
```

> Notes:
>
> * DOIs are **lowercased** and canonicalized to `https://doi.org/...`.
> * Accessions are **not** turned into DOIs.
> * `type` ∈ `{Primary, Secondary}` only.
> * No extra columns (e.g., no `context`).
