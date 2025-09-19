# MakeDataCount
Make Data Count Project in Kaggle
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

