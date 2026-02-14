# AskQE++: Strict Binary Verification and LLM-Based Judges for Explainable Quality Estimation

## Project Description

This repository contains the implementation and extension of the [AskQE framework](https://github.com/dayeonki/askqe) that utilizes LLMs to verify semantic consistency through question generation and Back-Translation.

In this study, we revisited the original AskQE pipeline, evaluated the capability of small, quantized LLMs (4B-14B parameters) to act as interpretable evaluators under compute constraints, and developed architectural improvements that outperform the baseline approach.

## Key Features & Results

- **Replication & Benchmarking:** Conducted a comparative analysis of various quantized models (Qwen, Llama, Gemma, Mistral families) on the BioMQM dataset. **Qwen3-4B-Instruct** was selected as the primary backbone.
- **Multi-Lingual LLM Judge:** Introduced a direct **Source-Target comparison** strategy that bypasses back-translation, reducing information loss and outperforming traditional metrics.
- **SOTA Fine-Tuning:** Developed a specialized model fine-tuned on BioMQM, achieving **State-of-the-Art** correlation with human judgments ($\tau \approx 0.41$) and record decision accuracy ($82\%$).
- **Binary QA Extension:** Reformulated AskQE as a Binary Yes/No verification framework with a four-channel hybrid question generator (template, SRL, word-difference, atomic facts).

## Repository Structure

- **LLM Judge/**: Directory containing implementations of various LLM judges.
  - **Mono/**: **Mono-lingual LLM Judge**.
    - **Methodology**: Evaluates translation quality by performing a semantic consistency check between answers derived from the **Source Sentence** and answers derived from the **Back-translation**.
    - **Process**: An LLM (`Qwen/Qwen3-4B-Instruct-2507`) compares the _Source Answers_ against the _Back-translation Answers_ to detect semantic conflicts.
    - **Scoring**: Mismatches are classified into severity levels: **CRITICAL** (contradictions, factually wrong), **MAJOR** (missing key details or hallucinated info), **MINOR** (slight precision loss), or **NONE** (semantically equivalent).
    - `Golden_LLM_Judge_Mono.ipynb`: Main notebook implementing the pipeline.
    - `askqe_results.jsonl`, `askqe_data.jsonl`: Experiment data.

  - **Multi/**: **Multi-lingual LLM Judge**.
    - **Methodology**: Utilizes a **direct cross-lingual evaluation strategy**, comparing the **Source Sentence** directly against the **Target Translation** without back-translation.
    - **Process**: Uses a "STRICT MODE" prompt where the LLM (`Qwen/Qwen3-4B-Instruct-2507`) extracts meaning units from the source and verifies their presence and accuracy in the target.
    - **Scoring**: Detects errors such as **Expansion** (added meaning), **Omission** (missing meaning), **Alteration** (antonyms, polarity flips), and **Hallucinations**. Assigns severity (CRITICAL/MAJOR/MINOR/NONE) based on the impact on truth conditions.
    - `Golden_LLM_Judge_Multi.ipynb`: Main notebook for the multi-judge.

  - **Fine Tuned/**: **Fine-Tuned Multi-lingual LLM Judge**.
    - **Methodology**: Builds upon the Multi-Judge architecture but applies **Supervised Fine-Tuning (SFT)** to specialize the model for the biomedical domain.
    - **Base Model**: `unsloth/Qwen3-4B-Instruct-2507-bnb-4bit`.
    - **Data**: Trained on a combination of the `bio-mqm-dataset` and internal `askqe` data.
    - **Preprocessing**: Includes strict filtering (deduplication, length checks, language validation) and class balancing (downsampling common 'NONE' and 'MINOR' classes to improve sensitivity to 'MAJOR' and 'CRITICAL' errors).
    - **Training**: Fine-tuned using LoRA on prompt-response pairs to predict error classifications and generate reasoning.
    - `Golden_Fine_Tuning_Judge.ipynb`: Training notebook.
    - `Golden_LLM_Judge_Multi_Fine_Tuned.ipynb`: Inference notebook.
    - `askqe_results_final.jsonl`: Final metrics.

- **Binary_QA/**: **Binary QA Extension** — Multilingual hybrid question generation with Boolean scoring.
  - **Methodology**: Reformulates the QA stage as a **Binary Yes/No semantic verification** framework. Instead of comparing open-ended answers with SBERT cosine similarity, each question becomes a Boolean consistency probe answered against both the source and back-translation.
  - **Question Generation**: Four-channel hybrid generator with priority-based deduplication:
    1. **Template** — spaCy-based surface anchors (negations, numbers, dates, entities, verb triples).
    2. **SRL** — LLM-extracted semantic roles (verb, agent, patient, time, location, negated).
    3. **Word-Difference** — Lemmatised token diff with fuzzy matching and negation detection.
    4. **Atomic Facts** — Few-shot LLM decomposition with NLI filtering (DeBERTa-v3-base-mnli).
  - **Model**: `Qwen2.5-7B-Instruct-AWQ` via vLLM (greedy, T=0, NVIDIA L4 GPU).
  - **Data**: Balanced subset of 500 samples (100 per language: de, es, fr, ru, zh-CN) from BioMQM.
  - `Binary_extension.ipynb`: Main notebook implementing the full pipeline.
  - `binary_extension_results.json`: Per-sample results (500 entries).
  - `extension_summary.csv`: Summary metrics per sample.

- **ablation/**: Ablation study experiments to verify hypotheses about component contributions.
  - `Golden_Baseline_ablation.ipynb`: Notebook for analyzing the importance of baseline components.

- **baseline/**: Directory with baseline solutions for various models (Qwen, Yi, Gemma, etc.).
  - In each subdirectory (e.g., `01-ai:Yi-1.5-9B-Chat/`):
    - `Golden_Baseline.ipynb`: Notebook implementing the baseline for a specific model.
    - `askqe_data.jsonl`, `askqe_results.jsonl`, `askqe_results_final.jsonl`: Experiment data and obtained metrics.
