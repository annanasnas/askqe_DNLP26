# ASKQE: Question Answering as Automatic Evaluation for Machine Translation

## Repository Structure

- **LLM Judge/**: Directory containing implementations of various LLM judges.
    - **Fine Tuned/**: **Fine-Tuned Multi-lingual LLM Judge**. Builds upon the Multi-Judge architecture but applies **Supervised Fine-Tuning (SFT)** to specialize the model for the biomedical domain (using BioMQM dataset).
        - `Golden_Fine_Tuning_Judge.ipynb`: Notebook for training the model (fine-tuning).
        - `Golden_LLM_Judge_Multi_Fine_Tuned.ipynb`: Inference of the fine-tuned model in multi-perspective mode.
        - `askqe_results_final.jsonl`: Final metric results.
    - **Mono/**: **Mono-lingual LLM Judge**. Evaluates translation quality by performing a **mono-lingual consistency check** between the Source text and its Back-translation using an LLM to compare semantic equivalence.
        - `Golden_LLM_Judge_Mono.ipynb`: Main notebook with mono-judge logic.
        - `askqe_data.jsonl`, `askqe_results.jsonl`: Intermediate data and results.
    - **Multi/**: **Multi-lingual LLM Judge**. Utilizes a **direct cross-lingual evaluation strategy**, comparing the **Source Sentence** directly against the **Target Translation**.
        - `Golden_LLM_Judge_Multi.ipynb`: Main notebook with multi-judge logic.

- **ablation/**: Ablation study experiments to verify hypotheses about component contributions.
    - `Golden_Baseline_ablation.ipynb`: Notebook for analyzing the importance of baseline components.

- **baseline/**: Directory with baseline solutions for various models (Qwen, Yi, Gemma, etc.).
    - In each subdirectory (e.g., `01-ai:Yi-1.5-9B-Chat/`):
        - `Golden_Baseline.ipynb`: Notebook implementing the baseline for a specific model.
        - `askqe_data.jsonl`, `askqe_results.jsonl`, `askqe_results_final.jsonl`: Experiment data and obtained metrics.

