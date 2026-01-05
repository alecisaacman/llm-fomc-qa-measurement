# How Does Powell Answer Questions?
## An Analysis of Federal Reserve Press Conference Communication

---

## 1. Introduction

After each Federal Open Market Committee (FOMC) meeting, the Chair of the Federal Reserve holds a press conference in which journalists ask questions about monetary policy, the economic outlook, and the Federal Reserve’s decision-making process. These press conferences play a critical role in shaping market expectations and public understanding of monetary policy. While prior research has focused on the *content* of policy decisions, less attention has been paid to **how** the Fed Chair answers questions.

This project studies the communication style of Chair Jerome Powell during the **October 29, 2025 FOMC press conference**. Specifically, it examines whether Powell’s answers are:

- Forward-looking  
- Certain or uncertain  
- Hawkish, dovish, or neutral  
- Assertive  
- Optimistic  
- Specific  

By combining large language models (LLMs), pre-trained text-classification models, and fine-tuned neural networks, this project provides a **structured and reproducible framework** for analyzing central bank communication.

---

## 2. Constructing the Press Conference Q&A Dataset

### 2.1 Transcript Processing and Q&A Extraction

There is no readily available dataset containing structured question–answer (Q&A) pairs from Federal Reserve press conferences. Therefore, the first step in this project was to construct such a dataset.

The full transcript of the October 29, 2025 FOMC press conference was obtained from the Federal Reserve website. A custom **GPT-based prompt** was designed to extract Q&A pairs directly from the transcript PDF.

A Q&A pair was defined as:
- One journalist question  
- The Chair’s immediate response  

Follow-up questions were treated as separate questions, and moderator transitions (e.g., identifying the next reporter) were excluded.

The prompt was instructed to return **strict JSON output**, preserving the original wording of both questions and answers. To remain within model context limits, the transcript was processed in text chunks, and the extracted pairs were combined into a single dataset.

The final output of this step was a CSV file (`powell_qa.csv`) in which each row represents a unique Q&A pair with two columns: `question` and `answer`. In total, **30 Q&A pairs** were extracted.

The full extraction prompt is provided in **Appendix A**.

---

## 3. The SubjECTive-QA Dataset and Pre-trained Models

To analyze communication style, this project builds on the **SubjECTive-QA** dataset introduced in *SubjECTive-QA: Measuring Subjectivity in Question-Answering Systems*.

The SubjECTive-QA dataset consists of question-answer pairs annotated for multiple subjective attributes, including:

- Assertiveness  
- Optimism  
- Specificity  
- Caution  
- Relevance  

### 3.1 Label Generation in SubjECTive-QA

In the original paper, labels were generated using a combination of expert annotation and LLM-assisted labeling, followed by validation procedures. The authors then fine-tuned transformer-based models to predict individual attributes from Q&A text.

Separate models were trained for each attribute, allowing flexible reuse without additional fine-tuning.

### 3.2 Limitations for Central Bank Communication

While SubjECTive-QA provides a powerful framework, it was not designed specifically for monetary policy contexts:

- The dataset spans corporate, financial, and general QA domains  
- Certain attributes (e.g., optimism) may reflect tone rather than policy intent  
- Institutional norms such as deliberate ambiguity may not be fully captured (e.g., central bank hedging)

These limitations motivate cautious interpretation and the use of **custom labeling and fine-tuned models** in this project.

---

## 4. Adding Assertive, Optimistic, and Specific Labels

Three attributes—**assertive**, **optimistic**, and **specific**—were added using pretrained SubjECTive-QA models hosted on Hugging Face:

- `gtfintechlab/SubjECTiveQA-ASSERTIVE`  
- `gtfintechlab/SubjECTiveQA-OPTIMISTIC`  
- `gtfintechlab/SubjECTiveQA-SPECIFIC`  

Each model was applied directly to the Powell Q&A pairs without additional fine-tuning, consistent with assignment instructions. The models output ordinal labels indicating the degree to which an answer exhibits each attribute.

The resulting dataset was saved as `powell_step3_subjective.csv`.

---

## 5. Forward-Looking and Certainty Labels via GPT

### 5.1 Definitions

Two additional attributes were required but not covered by SubjECTive-QA:

- **Forward-looking**: discusses future economic conditions, outlooks, or policy paths  
- **Certain**: presents information definitively without hedging or speculation  

### 5.2 Prompt Design

Two structured GPT prompts were designed to label these attributes. Each prompt:
- Took a Q&A pair as input  
- Returned **strict JSON output**  
- Applied clear, rule-based definitions  

The prompts are provided in **Appendix B** (forward-looking) and **Appendix C** (certainty).

### 5.3 Prompt Validation

To evaluate prompt performance, a random sample of 25 Q&A pairs from SubjECTive-QA was manually labeled for both attributes. GPT-generated labels were compared to manual labels using accuracy, precision, recall, and F1 score.

These validation statistics are reported in the notebook outputs referenced in this repository.

---

## 6. Fine-Tuning DistilBERT Models

To move beyond prompt-based labeling, two **DistilBERT** models were fine-tuned using GPT-labeled SubjECTive-QA data:

- Forward-looking vs. not forward-looking  
- Certain vs. uncertain  

### 6.1 Training Setup

- Base model: `distilbert-base-uncased`  
- Input: concatenated Q&A text  
- Loss: cross-entropy  
- Evaluation metrics: accuracy and F1  
- Stratified train/validation/test splits  
- Training epochs: 3  

### 6.2 Model Performance

**Forward-looking model**  
- Test accuracy ≈ 0.79  
- F1 score ≈ 0.87  

**Certainty model**  
- Test accuracy ≈ 0.73  
- F1 score ≈ 0.00  

The certainty model’s poor F1 reflects extreme class imbalance, with the model predicting the majority (uncertain) class. This result is informative, highlighting the inherent difficulty of identifying definitive statements in institutional central bank communication.

Both models were saved and applied to the Powell Q&A dataset.

---

## 7. Hawkish, Dovish, or Neutral Classification

Each answer was labeled as **hawkish**, **dovish**, or **neutral** using a GPT-based prompt. Because policy tone is highly context-dependent, a reasoning-based approach was preferred over purely statistical classification.

Prompt definitions are provided in **Appendix D**.

---

## 8. Results

### 8.1 Label Counts (30 Q&A pairs)

- **Forward-looking**:  
  - 22 forward-looking  
  - 8 not forward-looking  

- **Certainty**:  
  - 30 uncertain  
  - 0 certain  

- **Hawkish / Dovish / Neutral**:  
  - 28 neutral  
  - 2 dovish  
  - 0 hawkish  

- **Assertive & Specific**:  
  - Roughly balanced between medium and high levels  

- **Optimistic**:  
  - Nearly all answers classified as optimistic  

Overall, Powell’s communication is predominantly **forward-looking, neutral in policy tone, and deliberately cautious**.

### 8.2 Correlation Analysis

A Spearman correlation matrix was computed across all labels. Key patterns include:

- Positive correlation between **forward-looking** and **specific** answers  
- Positive relationship between **forward-looking** and **assertiveness**  
- Negative correlations between **policy tone** (hawkish/dovish) and both assertiveness and specificity  
- No computable correlations involving certainty due to zero variance  

These results suggest that when Powell discusses the future, he tends to provide structured and detailed explanations while maintaining a neutral policy stance.

---

## 9. Limitations

- Analysis focuses on a single press conference  
- Certainty labels exhibit extreme class imbalance  
- GPT-based labeling introduces potential noise  
- SubjECTive-QA models were not designed for monetary policy contexts  

Despite these limitations, the framework is scalable to additional meetings and policymakers.

---

## 10. Conclusion

This project demonstrates that Chair Powell’s press conference communication is systematically **forward-looking**, **neutral in tone**, and characterized by **deliberate uncertainty**. By combining GPT-based extraction, pretrained subjective classifiers, and fine-tuned DistilBERT models, the analysis provides a reproducible methodology for studying how central bankers communicate policy without making explicit commitments.

---

## Appendix A — Q&A Extraction Prompt
You are extracting question–answer (Q&A) pairs from a Federal Reserve press conference transcript.

Rules:
- A Q&A pair is ONE question and its immediate answer by the Chair.
- Follow-up questions count as separate questions.
- Exclude moderator transitions (e.g., “MICHELLE SMITH: Nick”).
- Do NOT merge multiple questions together.
- Preserve original wording.
- Output STRICT JSON ONLY.

Schema:
{
  "pairs": [
    {
      "question": "...",
      "answer": "..."
    }
  ]
}

Transcript:
<<TRANSCRIPT>>

## Appendix B — Forward-Looking Prompt
You are labeling a Q&A pair.

Forward-looking (1):
- Discusses future economic conditions
- Discusses future monetary policy decisions
- Discusses projections, outlooks, or what will happen

Not forward-looking (0):
- Discusses past or current conditions only
- Provides explanations of previous decisions
- Does not reference future events or policy paths

Return STRICT JSON ONLY:
{"forward_looking": 1 or 0}

Q&A:
<<QA>>

## Appendix C — Certainty Prompt
You are labeling a Q&A pair.

Certain (1):
- Presents information definitively
- Uses committed language
- Avoids hedging or conditional phrasing

Uncertain (0):
- Uses speculative or conditional language
- Includes words such as “might,” “could,” “depends,” “we’ll see,” or similar hedging
- Indicates uncertainty or flexibility

Return STRICT JSON ONLY:
{"certain": 1 or 0}

Q&A:
<<QA>>

## Appendix D — Hawkish/Dovish Prompt
You are labeling the monetary policy TONE of a Federal Reserve Chair's answer.

Definitions:
- Hawkish (2):
  Emphasizes inflation risks, tighter monetary policy, higher interest rates for longer,
  restraining demand, or concern about persistent inflation.

Dovish (0):

  improving inflation, patience, or accommodative policy.
  Emphasizes downside risks to growth or employment, openness to easing,

Neutral (1):

  acknowledges multiple risks,
  explanatory in nature without a clear policy tilt.
  Balanced or technical discussion,

Return STRICT JSON ONLY:
{"hawk_dove_neutral": 0 or 1 or 2}

Answer:
<<ANSWER>>
