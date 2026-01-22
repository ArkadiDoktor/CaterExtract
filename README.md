# CaterExtract: Structured Feature Extraction from Business Email Negotiations

## Project Motivation

Modern business negotiations often occur through unstructured email threads. Critical contractual details—such as pricing, guest counts, dietary requirements, or cancellation policies—are implicitly agreed upon across multiple messages rather than stated explicitly in a single place. Manually extracting these details is time-consuming and error-prone. **CaterExtract** aims to explore and compare multiple modeling paradigms for automatically extracting structured contract-level features from multi-turn catering-related email negotiations.

## Problem Statement

Given a multi-message business email thread between a food supplier and an event organizer, the task is to extract a fixed set of structured features representing the *final agreed terms* of the contract. The challenge lies in handling implicit mentions, negotiation dynamics, value revisions, missing fields, and heterogeneous feature types (boolean, categorical, numeric, lists, and free text).

## Visual Abstract

The project follows a pipeline of:

1. Synthetic email thread generation
2. Ground truth feature annotation (JSON schema)
3. Feature extraction using multiple modeling approaches
4. Quantitative evaluation and comparison

Visual summaries of model performance are available in the **Visuals/** directory (charts and tables per method).

## Datasets Used / Collected

Two datasets are used:

### 1. Complete Dataset

* **1,000 synthetic email threads**
* Each thread contains exactly four emails simulating a short negotiation
* Each thread has a corresponding ground truth JSON file

Structure:

* `email threads/` – email_thread_001.txt … email_thread_1000.txt
* `ground truth/` – ground_truth_001.json … ground_truth_1000.json
* `extracted features zero-shot/` – LLM zero-shot predictions
* `extracted features few-shot/` – LLM few-shot predictions

### 2. Intermediate Dataset

* **30 email threads**
* Each thread contains exactly four emails simulating a short negotiation
* Each thread has a corresponding ground truth JSON file

Structure mirrors the complete dataset but at smaller scale.

## Data Augmentation and Generation Methods

All email threads and ground truth annotations are **synthetically generated** using an LLM under strict constraints:

* Natural business prose only (no templates or placeholders)
* Realistic names, companies, and negotiation flow
* Implicit mention of all features across the conversation
* Final agreement or cancellation always appears in the last email

Ground truth features are generated separately and stored in structured JSON format to avoid information leakage during extraction.

## Input / Output Examples

**Input:**
A four message email thread discussing catering for an event, including negotiation over price, guest count, and services.

**Output:**
A JSON object with the following fields:

* event_type
* price_type
* final_price_value
* min_guests / max_guests
* includes_vat
* is_kosher / kosher_supervision
* includes_bar
* menu_type
* dietary_options
* event_date
* cancellation_policy
* menu_highlights
* extra_notes

Missing or unspecified values are represented as `null`.

## Models and Pipelines Used

The project compares four distinct approaches:

1. **Zero-Shot LLM Extraction**

   * Prompt-based extraction using an LLM
   * No examples provided

2. **Few-Shot LLM Extraction**

   * Same as zero-shot, but with a single annotated example in the prompt

3. **Fine-Tuned Transformer Model**

   * DistilBERT encoder
   * Multi-head architecture supporting:

     * Classification
     * Regression
     * Binary prediction
     * Multi-label prediction

4. **NER-Based Baseline (GLiNER)**

   * Entity extraction from first and last emails only
   * Rule-based post-processing to map entities into schema fields

## Training Process and Parameters

**Fine-Tuned Model Details:**

* Encoder: DistilBERT-base-uncased
* Max sequence length: 512
* Batch size: 1
* Epochs: 10
* Optimizer: AdamW
* Learning rate: 2e-5

**Loss Functions:**

* Cross-Entropy (categorical fields)
* BCEWithLogits (binary & multi-label fields)
* MSE (numeric regression fields)

Loss weighting is applied to emphasize harder or more important fields (e.g., price and numeric values).

## Metrics and Results

Evaluation is performed at the **field level** and averaged across all features:

* Exact match for categorical and boolean fields
* Relative error tolerance for numeric fields
* Overlap / F1-style scoring for list fields

Summary results are stored in:

* `Results/Zero-Shot-LLM_results.csv`
* `Results/Few-Shot-LLM_results.csv`
* `Results/Fine-Tuned-Model_results.csv`
* `Results/Interm_results.csv`

Key findings:

* LLMs excel at boolean and semantic fields
* Schema-constrained fields (e.g., price_type) are challenging for prompt-based methods
* The fine-tuned model provides the best overall consistency
* Pure NER approaches struggle with agreement resolution

## Repository Structure

```
Code/
 ├── Finale_notebook.ipynb
 └── Interm_notebook.ipynb

Complete Dataset/
 ├── email threads/
 ├── extracted features zero-shot/
 ├── extracted features few-shot/
 └── ground truth/

interm_dataset/
 ├── email threads/
 ├── extracted features zero-shot/
 └── ground truth/

Results/
 ├── Zero-Shot-LLM_results.csv
 ├── Few-Shot-LLM_results.csv
 ├── Fine-Tuned-Model_results.csv
 └── Interm_results.csv

Slides/
 ├── Proposal Slides/
 ├── Interm Slides/
 └── Final Slides/

Visuals/
 ├── Zero-Shot-LLM/
 ├── Few-Shot-LLM/
 ├── Fine-Tuned-Model/
 └── Interm/
```

## Team Members

* Project Author: *[Arkadi Doktorovich]*
* Project Participant: *[Hen Mandelbaum]*

---

This repository accompanies an academic project on structured information extraction from business email negotiations and is intended for research and educational use.
