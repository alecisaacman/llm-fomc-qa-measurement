# LLM-Based Measurement of FOMC Q&A Communication

This project applies large language models (LLMs) and transformer-based classifiers to measure **how** Federal Reserve Chair Jerome Powell answers questions during FOMC press conferences.

Rather than analyzing policy outcomes, the focus is on **communication style**—including forward-looking language, certainty, policy tone, and subjective attributes—using reproducible NLP pipelines.

---

## Research Question

How does Chair Powell communicate monetary policy during FOMC press conferences?

Specifically:
- Are answers forward-looking or backward-looking?
- How certain or uncertain is the language?
- Is the tone hawkish, dovish, or neutral?
- How assertive, optimistic, and specific are responses?

---

## Data

- **Press conference transcript**: October 29, 2025 FOMC meeting (Federal Reserve)
- **Constructed dataset**: 30 journalist question–answer (Q&A) pairs extracted via a GPT-based parser
- **Labeled outputs**: Forward-looking, certainty, tone, and subjective attributes

Raw transcripts are not included. Derived datasets used in the analysis are provided.

---

## Methodology Overview

1. **Q&A Extraction**  
   A custom GPT prompt extracts structured Q&A pairs from the official transcript.

2. **Subjective Attribute Labeling**  
   Pretrained SubjECTive-QA models are applied to measure:
   - Assertiveness  
   - Optimism  
   - Specificity  

3. **Forward-Looking & Certainty Classification**  
   - GPT-based labeling with strict JSON outputs  
   - Prompt validation against manually labeled samples  

4. **Fine-Tuned Models**  
   DistilBERT models are fine-tuned to classify:
   - Forward-looking vs. not forward-looking  
   - Certain vs. uncertain language  

5. **Policy Tone Classification**  
   A GPT-based classifier labels answers as hawkish, dovish, or neutral.

Full methodological details, prompts, validation results, and analysis are documented separately.

---

## Results (High Level)

- Powell’s answers are predominantly **forward-looking**
- Policy tone is overwhelmingly **neutral**
- Language exhibits **deliberate uncertainty**, consistent with central bank communication norms
- Forward-looking answers tend to be more **assertive and specific**

Detailed results and correlation analysis are provided in the full write-up.

---

## Repository Structure

```text
.
├── README.md        # Project overview (this file)
├── analysis.md      # Full methodological write-up and results
└── notebooks/       # Reproducible analysis pipeline
```

```markdown
## How to Reproduce

1. Open the main notebook in the `notebooks/` directory using Jupyter or Google Colab.
2. Install the required Python packages listed at the top of the notebook.
3. Ensure the labeled Q&A CSV file is available at the path specified in the notebook.
4. Run the notebook top-to-bottom to reproduce all tables and figures.

LLM-based labeling steps are documented in detail in `analysis.md`.

---

## Notes

This project was completed as part of an advanced applied machine learning course in economics and is intended to demonstrate the use of LLMs as **measurement tools**, not black-box predictors, in economic research.

