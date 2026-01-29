# DataLit-Scientific-Novelty-Impact
Analysis of scientific papers to understand relation between  novelty and impact


> ** Work in Progress:** This repository is part of an ongoing project. An analysis notebook with visualizations will be added soon.

This repository contains the data collection and preprocessing pipeline for a scientific paper analysis study. It processes academic papers from the [Semantic Scholar Academic Graph](https://www.semanticscholar.org/product/api) across three research domains: **Computer Science (AI)**, **Physics**, and **Psychology**.

## Repository Structure

```
.
├── get_semantic_scholar_papers.ipynb      # Step 1: Download paper metadata
├── get_semantic_scholar_abstracts.ipynb   # Step 2: Download paper abstracts
├── merge_papers_and_abstracts.ipynb       # Step 3: Merge metadata with abstracts
├── clean_and_merge_dbs.ipynb              # Step 4: Clean data & classify domains
├── generate_embeddings.ipynb              # Step 5: Generate SPECTER2 embeddings
├── viz/                                   # Generated visualizations
│   ├── Computer_Science_wordcloud_merged.png
│   ├── Physics_wordcloud_merged.png
│   └── Psychology_wordcloud_merged.png
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

### 4. Data Cleaning and Domain Classification (`clean_and_merge_dbs.ipynb`)

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

### 5. Embedding Generation (`generate_embeddings.ipynb`)

Generates dense vector embeddings for papers using the SPECTER2 model.

- **Model:** [SPECTER2](https://huggingface.co/allenai/specter2) from Allen AI
- **Input Format:** `"Title [SEP] Abstract"`
- **Embedding Dimension:** 768
- **Output:** Embeddings uploaded to Hugging Face Hub: [`lalit3c/S2_CS_PHY_PYSCH_papers`](https://huggingface.co/datasets/lalit3c/S2_CS_PHY_PYSCH_papers)

## Data Availability

The processed dataset is publicly available on Hugging Face Hub:

🤗 **[lalit3c/S2_CS_PHY_PYSCH_papers](https://huggingface.co/datasets/lalit3c/S2_CS_PHY_PYSCH_papers)**

## Requirements

### Python Dependencies

```
pandas
duckdb
requests
tqdm
torch
transformers
adapters
huggingface_hub
numpy
```

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

# 4. Clean data and classify domains
jupyter notebook clean_and_merge_dbs.ipynb

# 5. Generate embeddings
jupyter notebook generate_embeddings.ipynb
```

> **Note:** Steps 1-4 require significant download time and storage due to the large size of the Semantic Scholar dataset. GPU is recommended for Step 5 (embedding generation).


---

*This repository is part of a project work for "Data Literacy” course at the University of Tübingen, Winter 2025/26 (Module ML4201).*
