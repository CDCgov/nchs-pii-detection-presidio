# PII Detection with Presidio - Example Code

## Overview

This repository contains example notebooks for using
[Presidio](https://presidio.dataprivacystack.org/) to detect
personally identifiable information (PII) in text, preparing evaluation
datasets, comparing named entity recognition (NER) backends, and calculating
model performance metrics.

The examples are based on the process used to assess the Presidio framework with
other machine learning and NLP methods for detecting PII in the 2023 [NAMCS
HC](https://www.cdc.gov/nchs/namcs-health-center/about/index.html) laboratory results field, which was presented at the 2026 FCSM Conference.

The repository provides two versions of the same workflow:

- `Plain Python version/`: local notebook workflow using pandas and local
  files. (Small sample *synthetic* data file is included. See Notes and Limitations Section for details of creation.)
- `Spark Databricks version/`: Databricks notebook workflow using Spark,
  catalog tables, and cloud storage paths.

## Workflow

Run the notebooks in numeric order. Each notebook has a `Set parameters`
section where you should update paths, column names, output locations, and
model-specific settings for your data.

### 1. Prepare evaluation data

- `1.1_explore_data_and_filter_out_non_pii*`: inspect the input text field and
  filter out values that are unlikely to contain PII, such as simple numeric
  lab results, dates, units, codes, and common non-PII terms using SQL.
- `1.2_create_evaluation_data_subsets*`: create review subsets, including the
  longest records with unique starting words and random records by reporting
  unit, using SQL.
- `1.3_apply_text_preprocessing_methods*`: create model input variants such as
  combined evaluation data, punctuation-stripped text, uppercase text,
  lowercase text, and optional context-augmented text, using SQL.

### 2. Run Presidio with different NER engines

- `2.1_run_presidio_with_default_spacy_ner*`: Presidio with [spaCy](https://huggingface.co/spacy/en_core_web_lg).
- `2.2_run_presidio_with_stanza_ner*`: Presidio with [Stanza English NER](https://stanfordnlp.github.io/stanza/ner.html).
- `2.3_run_presidio_with_stanford_ner*`: Presidio with [Stanford de-identifier](https://huggingface.co/StanfordAIMI/stanford-deidentifier-base).
- `2.4_run_presidio_with_roberta_ner*`: Presidio with [RoBERTa](https://huggingface.co/obi/deid_roberta_i2b2) fine-tuned for de-identification of medical notes
- `2.5_run_presidio_with_clinicalbert_ner*`: Presidio with [ClinicalBERT](https://huggingface.co/obi/deid_bert_i2b2) fine-tuned for de-identification of medical notes.


The Presidio notebooks write model output with fields such as
`detected_entities` and `flagged_text`. The default entity list includes
`PERSON`, `NRP`, `DATE_TIME`, `LOCATION`, `EMAIL_ADDRESS`, `PHONE_NUMBER`,
`US_SSN`, and `ORGANIZATION`; edit `my_entities_list`, `my_allow_list`, and
`my_deny_list` as needed.

### 3. Evaluate results

- `3.1_calculate_metrics_one_model*`: compare one model's output to manual
  annotations and calculate recall, precision, accuracy, F1, and confusion
  matrices.
- `3.2_calculate_metrics_combine_models*`: combine outputs from multiple
  models, flag a row if any selected model found PII, and calculate the same
  metrics.

The metric notebooks expect manual annotations with a binary
`manual_pii_flag` column and model output with a `detected_entities` column.
For PII detection, the notebooks emphasize recall because missed PII is usually
the highest-risk failure mode.

## Getting Started

### Prerequisites

- Python 3.11
- `uv` for dependency management
- Jupyter, VS Code, Databricks, or another notebook runner
- Network access, or a populated local model cache, for Stanza and Hugging Face
  transformer model downloads

### Local Python setup

From the repository root:

```powershell
uv sync
```

Then open the notebooks in `Plain Python version/` and select the virtual
environment created by `uv`.

If you are running outside an IDE that already provides notebook support, you
may need to install a notebook kernel in your local environment.

```powershell
uv run python -m pip install ipykernel jupyter
```

The main workflow is notebook-based. `main.py` is only a minimal placeholder
entry point.

### Databricks setup

Import the notebooks from `Spark Databricks version/` into a Databricks
workspace. Review each notebook's setup cells and update:

- catalog, schema, table, and storage paths
- input text column and ID columns
- model output tables
- annotation file paths
- cluster libraries or `%pip install` cells required by the selected model

Transformer and Stanza models may need to be downloaded or made available to
cluster workers before distributed analysis runs.

## Data

The repository includes a small example JSON file (synthetic data, no real PII) in
`Plain Python version/data/` for local experimentation:

- `pii_finetune_data_formatted.json`

Generated CSVs, model outputs, real datasets, and other potentially sensitive
artifacts are ignored by `.gitignore` by default based on file type. Keep source data and model outputs outside version control unless they have been reviewed and explicitly
approved for sharing.

For the metric notebooks, manually reviewed evaluation files should include:

- stable join keys, such as `encounter_id` and `lab_eventnumber`
- the target text column, usually `text`
- `manual_pii_flag`, where `1` means PII is present and `0` means PII is not
  present

## Repository Structure

```text
.
|-- Plain Python version/
|   |-- 1.* data preparation notebooks
|   |-- 2.* Presidio model notebooks
|   |-- 3.* metric notebooks
|   `-- data/
|       |-- pii_finetune_data_formatted.json
|-- Spark Databricks version/
|   |-- 1.* data preparation notebooks
|   |-- 2.* Presidio model notebooks
|   `-- 3.* metric notebooks
|-- pyproject.toml
|-- uv.lock
`-- ruff.toml
```

## Notes and Limitations

- These notebooks are examples based on analyzing laboratory results data and should be adapted to the data source, annotation rules, entity definitions, and review workflow for your project.
- Presidio and NER models can produce false positives and false negatives. Human review is still needed, especially for high-risk PII workflows.
- Data fields and generated outputs may contain sensitive information. Follow applicable data handling, access control, and disclosure review policies.
- The sample synthetic data file was generated by ChatGPT to provide a quick test dataset for several PII redaction projects. Authors made a few manual additions and edits to ensure the data represented different cases that the code should address. The initial prompts were:
  - "I want to construct a dataset that I can use for finetuning an llm to tell me whether a given string contains PII. I'm particularly interested in finding any names or addresses. Can you give me a .jsonl file with enough examples to accurately capture this concept for finetuning?"
  - "Also, can you vary some of the text (upper case/lower case, typos, spacing, special characters etc.) like you might see in real world data."
- The plain Python notebooks were created using OpenAI's Codex to adapt the Spark Databricks notebooks for a wider audience. Authors tested and verified notebook functionality using the included test dataset. This README was drafted using OpenAI's Codex to describe the code's structure and suggested usage. Authors reviewed and edited this document for accuracy, clarity, and relevance.

## Project Admins

- **Point of Contact**: Rachel Fan RFan@cdc.gov
  (CDC/OD/OPHDST/NCHS/DRM)
- **Contributors**: Priyam Patel PPatel12@cdc.gov
  (CDC/OD/OPHDST/NCHS/DRM)
