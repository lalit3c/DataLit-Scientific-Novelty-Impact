# DataLit-Scientific-Novelty-Impact
Analysis of scientific papers to understand relation between  novelty and impact



This repository contains the data collection, preprocessing pipeline and analysis for a scientific paper analysis study. It processes academic papers from the [Semantic Scholar Academic Graph](https://www.semanticscholar.org/product/api) across three research domains: **Computer Science (AI)**, **Physics**, and **Psychology**.

## Repository Structure

```
.
├── get_semantic_scholar_papers.ipynb      # Step 1: Download Semantic Scholar metadata
├── get_semantic_scholar_abstracts.ipynb   # Step 2: Download abstracts from Semantic Scholar
├── merge_papers_and_abstracts.ipynb       # Step 3: Merge metadata with abstracts
├── harvest_openalex_primary.ipynb         # Step 4: Download papers from OpenAlex by topic
├── clean_and_merge_dbs.ipynb              # Step 5: Clean data and merge databases
├── generate_embeddings.ipynb              # Step 6: Generate SPECTER2 embeddings
├── compute_metrics.ipynb                  # Step 7: Compute novelty metrics
├── AI_tsne.ipynb                          # Step 8: t-SNE visualization for AI papers
├── tsne_per_paper.ipynb                   # Step 9: t-SNE visualization per paper (top papers)
├── analysis.ipynb                         # Step 10: Novelty and Citation analysis
├── data/                                  # Stored data artifacts
│   ├── S2_papers_cleaned.db               # Cleaned Semantic Scholar papers
│   ├── S2_papers_cleaned_additional_papers.db  # Additional cleaned papers
│   ├── metrics.db                         # Computed novelty metrics
│   ├── ai_top_papers_tsne_per_query.csv   # Top AI papers t-SNE data
│   └── topic_sheets/                      # Topic definitions (AI, Physics, Psychology)
├── viz/                                   # Generated visualizations
│   ├── Computer_Science_wordcloud_merged.png
│   ├── Physics_wordcloud_merged.png
│   ├── Psychology_wordcloud_merged.png
│   ├── TSNE-top_5_AI.png
│   ├── attention_is_all_you_need_neighbors.pdf
│   └── novelty_vs_citations_fixed_width_bins.pdf
├── Report-OpeningLinesThatMatter.pdf      # Project report
├── requirements.txt                       # Python dependencies
└── README.md

```

## Pipeline Overview

### 1. Paper Collection (`get_semantic_scholar_papers.ipynb`)

Downloads and filters academic papers from the Semantic Scholar dataset.

- **Data Source:** [Semantic Scholar Datasets API](https://api.semanticscholar.org/api-docs/datasets)
- **Target Fields:** Physics, Computer Science, Psychology
- **Output:** `db/ss_aws.db` - DuckDB database with paper metadata (corpusid, title, publication date, citation counts, field of study)

### 2. Abstract Collection (`get_semantic_scholar_abstracts.ipynb`)

Downloads abstracts for papers collected in Step 1.

- **Prerequisite:** Run `get_semantic_scholar_papers.ipynb` first
- **Output:** `db/ss_abstracts.db` - DuckDB database with abstracts and external IDs

### 3. Merge Papers and Abstracts (`merge_papers_and_abstracts.ipynb`)

Combines paper metadata with abstracts into a unified database.

- **Input:** `ss_aws.db`, `ss_abstracts.db`
- **Output:** `db/S2_papers.db` - Merged database with papers and their abstracts
- **Additional Output:** Wordcloud visualizations by field and decade


### 4. Download papers from OpenAlex (`harvest_openalex_primary.ipynb`)

Downloads openalex datsets for given topics
- **Inputs:** `data/topic_sheets/AI.xlsx`, `data/topic_sheets/Physics.xlsx`, `data/topic_sheets/Psychology.xlsx`
- **Outputs:** `data/dbs/openalex_ai.db`, `data/dbs/openalex_physics.db`, `data/dbs/openalex_psychology.db`

### 5. Data Cleaning and Domain Classification (`clean_and_merge_dbs.ipynb`)

Cleans the merged data and classifies papers into research domains using OpenAlex databases.

**Processing Steps:**
1. Filter papers with complete metadata (title, abstract, publication date)
2. Exclude papers published before 2000
3. Classify papers by domain (AI, Physics, Psychology) using hierarchical matching:
   - Title matching (exact match)
   - DOI matching
   - MAG ID matching
4. Extract primary topic information from OpenAlex
5. Optimize storage for the final dataset

**Input Databases:**
- `S2_papers.db` - Merged Semantic Scholar papers
- `openalex_ai.db` - OpenAlex AI papers
- `openalex_physics.db` - OpenAlex Physics papers
- `openalex_psych.db` - OpenAlex Psychology papers

**Output:**
- `db/S2_papers_cleaned.db` - Cleaned and domain-classified papers
- Uploaded to Hugging Face Hub: [`lalit3c/S2_CS_PHY_PYSCH_papers`](https://huggingface.co/datasets/lalit3c/S2_CS_PHY_PYSCH_papers)

### 6. Embedding Generation (`generate_embeddings.ipynb`)

Generates dense vector embeddings for papers using the SPECTER2 model.

- **Model:** [SPECTER2](https://huggingface.co/allenai/specter2) from Allen AI
- **Input Format:** `"Title [SEP] Abstract"`
- **Embedding Dimension:** 768
- **Output:** Embeddings uploaded to Hugging Face Hub: [`lalit3c/S2_CS_PHY_PYSCH_papers`](https://huggingface.co/datasets/lalit3c/S2_CS_PHY_PYSCH_papers)

### 7. Compute Novelty Metrics(`compute_novelty.ipynb`)

**Input Databases:**
- `S2_papers_cleaned.db` – Cleaned Semantic Scholar metadata
- `S2_papers_cleaned_additional_papers.db` – Additional cleaned papers
- `embeddings/embeddings_*.db` – Sharded embedding databases

**Output:**
- `metrics.db` – Per-paper novelty metrics computed via ANN search
- Uploaded to Hugging Face Hub: [`lalit3c/S2_CS_PHY_PYSCH_papers`](https://huggingface.co/datasets/lalit3c/S2_CS_PHY_PYSCH_papers)

### 8. t-SNE Visualization for AI Papers (`AI_tsne.ipynb`)

Generates t-SNE visualizations to explore the embedding space of academic papers.

- **Model:** SPECTER2 embeddings
- **Input:** `S2_papers_cleaned.db`, `embeddings/*.db` files
- **Output:** t-SNE plots for the top 5 most cited topics in AI, Physics, and Psychology
- **Visualization:** `viz/TSNE-top_5_AI.png`

### 9. t-SNE Per Paper Analysis (`tsne_per_paper.ipynb`)

Generates per-paper t-SNE visualizations for top papers to analyze nearest neighbors in the embedding space.

- **Input:** Embeddings from Hugging Face Hub, `S2_papers_cleaned.db`
- **Output:** 
  - `data/ai_top_papers_tsne_per_query.csv` - Top AI papers t-SNE coordinates
  - `viz/attention_is_all_you_need_neighbors.pdf` - Neighbor analysis visualization


### 10. Analysis and Plot Generation (`analysis.ipynb`)

Analyzes the relationship between novelty metrics and citation counts across research domains.

- **Input:** `metrics.db`, `S2_papers_cleaned.db`
- **Output:** Analysis plots including `novelty_vs_citations_fixed_width_bins.pdf`


## Data Availability

The processed dataset is publicly available on Hugging Face Hub:

🤗 **[lalit3c/S2_CS_PHY_PYSCH_papers](https://huggingface.co/datasets/lalit3c/S2_CS_PHY_PYSCH_papers)**

## Requirements

### Python Dependencies

See `requirements.txt` for the full list. 

### API Keys Required

- **Semantic Scholar API Key:** Required for `get_semantic_scholar_papers.ipynb` and `get_semantic_scholar_abstracts.ipynb`
- **Hugging Face Token:** Required for `generate_embeddings.ipynb` and uploading datasets

## Usage

Execute the notebooks in sequential order:

```bash
# 1. Collect paper metadata
jupyter notebook get_semantic_scholar_papers.ipynb

# 2. Collect abstracts
jupyter notebook get_semantic_scholar_abstracts.ipynb

# 3. Merge papers and abstracts
jupyter notebook merge_papers_and_abstracts.ipynb

# 4. Download openalex papers
jupyter notebook harvest_openalex_primary.ipynb

# 5. Clean data and classify domains
jupyter notebook clean_and_merge_dbs.ipynb

# 6. Generate embeddings
jupyter notebook generate_embeddings.ipynb

# 7. Compute novelty metrics
jupyter notebook compute_metrics.ipynb

# 8. t-SNE visualization for AI papers
jupyter notebook AI_tsne.ipynb

# 9. t-SNE per paper analysis for top papers
jupyter notebook tsne_per_paper.ipynb

# 10. Analysis and plot generation
jupyter notebook analysis.ipynb


```

> **Note:** Steps 1-4 require significant download time and storage due to the large size of the Semantic Scholar dataset. GPU is recommended for Step 6 (embedding generation). Steps 9-10 require the openTSNE package.


---

*This repository is part of a project work for "Data Literacy” course at the University of Tübingen, Winter 2025/26 (Module ML4201).*
