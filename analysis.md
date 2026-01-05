
How Does Powell Answer Questions?

An Analysis of Federal Reserve Press Conference Communication

1. Introduction
After each Federal Open Market Committee (FOMC) meeting, the Chair of the Federal Reserve holds a press conference in which journalists ask questions about monetary policy, the economic outlook, and the Federal Reserve’s decision-making process. These press conferences play a critical role in shaping market expectations and public understanding of monetary policy. While prior research has focused on the content of policy decisions, less attention has been paid to how the Fed Chair answers questions.

This project studies the communication style of Chair Jerome Powell during the October 29, 2025 FOMC press conference. Specifically, it examines whether Powell’s answers are forward-looking, certain or uncertain, hawkish or dovish, assertive, optimistic, and specific. By combining large language models (LLMs), pre-trained text-classification models, and fine-tuned neural networks, this project provides a structured, reproducible approach to analyzing Fed communication.

2. Constructing the Press Conference Q&A Dataset
2.1 Transcript Processing and Q&A Extraction
There is no readily available dataset containing structured question-answer (Q&A) pairs from Federal Reserve press conferences. Therefore, the first step in this project was to construct such a dataset.

The full transcript of the October 29, 2025 FOMC press conference was obtained from the Federal Reserve website. A custom GPT-based prompt was designed to extract Q&A pairs directly from the transcript PDF. A Q&A pair was defined as a single journalist question and the Chair’s immediate response. Follow-up questions were treated as separate questions, and moderator transitions (e.g., identifying the next reporter) were excluded.

The GPT prompt was instructed to return strict JSON output containing a list of Q&A pairs, preserving the original wording of both questions and answers. The transcript was processed in text chunks to remain within model context limits. The extracted pairs were combined into a single dataset.

The final output of this step was a CSV file (powell_qa.csv) in which each row represents a unique Q&A pair with two columns: question and answer. In total, 30 Q&A pairs were extracted from the press conference.

The full Q&A extraction prompt is provided in Appendix A.

3. The SubjECTive-QA Dataset and Pre-trained Models
To analyze communication style, this project builds on the SubjECTive-QA dataset introduced in SubjECTive-QA: Measuring Subjectivity in Question-Answering Systems (arXiv:2410.20651). The SubjECTive-QA dataset consists of question-answer pairs annotated for multiple subjective attributes, including assertiveness, optimism, specificity, caution, and relevance.

3.1 How SubjECTive-QA Labels Were Generated
In the original paper, labels were generated using a combination of expert annotation and LLM-assisted labeling, followed by validation procedures. The authors then fine-tuned transformer-based models to predict individual attributes from Q&A text. Separate models were trained for each attribute, allowing flexible reuse without additional fine-tuning.

3.2 Limitations for Fed Communication Analysis
While SubjECTive-QA provides a powerful framework, it was not designed specifically for central bank communication. As a result:

The dataset spans corporate, financial, and general QA contexts rather than monetary policy discussions.

Certain attributes (e.g., optimism) may reflect tone rather than policy intent.

The models may not fully capture institutional norms such as deliberate ambiguity used by central banks.

These limitations are addressed by interpreting results cautiously and supplementing SubjECTive-QA labels with custom labeling and fine-tuned models tailored to this project.

4. Adding Assertive, Optimistic, and Specific Labels
Three attributes—assertive, optimistic, and specific—were added to the Powell Q&A dataset using pretrained SubjECTive-QA models hosted on Hugging Face:

gtfintechlab/SubjECTiveQA-ASSERTIVE

gtfintechlab/SubjECTiveQA-OPTIMISTIC

gtfintechlab/SubjECTiveQA-SPECIFIC

These models were applied directly to each Q&A pair without additional fine-tuning, consistent with the assignment instructions. Each model produces ordinal labels indicating the degree to which an answer exhibits the given attribute.

The resulting labeled dataset was saved as powell_step3_subjective.csv.

5. Forward-Looking and Certainty Labels via GPT
5.1 Definitions
Two additional attributes were required but not covered by SubjECTive-QA:

Forward-looking: The answer discusses future economic conditions, policy decisions, or outlooks.

Certain: The answer presents information definitively, without speculation or hedging.

5.2 GPT Prompt Design
Two structured GPT prompts were designed to label these attributes. Each prompt took a Q&A pair as input and returned strict JSON output with binary labels. The prompts emphasized clear definitions and required deterministic responses.

The prompts are provided in Appendix B (forward-looking) and Appendix C (certainty).

5.3 Prompt Validation
To evaluate prompt performance, a random sample of 25 Q&A pairs from the SubjECTive-QA dataset was manually labeled for forward-looking and certainty. GPT-generated labels were compared to manual labels, and standard classification metrics (accuracy, precision, recall, and F1 score) were computed. These validation statistics were used to assess prompt reliability and are reported in the code outputs referenced in this project.

6. Fine-Tuning DistilBERT Models
To move beyond prompt-based labeling, two DistilBERT models were fine-tuned using the GPT-labeled SubjECTive-QA data:

Forward-looking vs. not forward-looking

Certain vs. uncertain

6.1 Training Setup
Base model: distilbert-base-uncased

Input: concatenated Q&A text

Loss: cross-entropy

Evaluation metrics: accuracy and F1 score

Stratified train/validation/test splits

Models were trained for three epochs and saved for later inference.

6.2 Model Performance
Forward-looking model:
Test accuracy ≈ 0.79, F1 ≈ 0.87

Certainty model:
Test accuracy ≈ 0.73, F1 ≈ 0.00

The certainty model’s low F1 score reflects severe class imbalance, with the model predicting the majority (uncertain) class. This outcome is informative rather than erroneous, highlighting the inherent difficulty of identifying definitive statements in institutional central bank communication.

Both models were saved and later applied to the Powell Q&A dataset.

7. Hawkish, Dovish, or Neutral Classification
Each Powell answer was also labeled as hawkish, dovish, or neutral using a GPT-based prompt. The prompt defined hawkish language as emphasizing inflation risks and tighter policy, dovish language as emphasizing downside risks or accommodation, and neutral language as balanced or technical commentary.

This approach was chosen because hawkish/dovish tone is highly context-dependent and benefits from natural language reasoning rather than purely statistical classification. The prompt is provided in Appendix D.

8. Results
8.1 Label Counts
From the 30 Q&A pairs:

Forward-looking:
22 forward-looking, 8 not forward-looking

Certain:
All 30 answers classified as uncertain

Hawkish/Dovish/Neutral:
28 neutral, 2 dovish, 0 hawkish

Assertive and Specific:
Roughly balanced between medium and high levels

Optimistic:
Nearly all answers classified as optimistic

These results suggest that Powell’s communication is predominantly forward-looking, neutral in policy tone, and deliberately cautious.

8.2 Correlation Analysis
A Spearman correlation matrix was computed across all labels. Key findings include:

A positive correlation between forward-looking and specific answers.

A positive relationship between forward-looking and assertiveness.

Negative correlations between hawkish/dovish tone and both assertiveness and specificity.

No computable correlations involving certainty due to zero variance.

These patterns indicate that when Powell discusses the future, he tends to provide more structured and detailed explanations, while maintaining a neutral policy stance.

9. Limitations
Several limitations should be noted:

The analysis focuses on a single press conference.

Certainty labels exhibit extreme class imbalance.

GPT-based labeling introduces potential noise.

SubjECTive-QA models were not designed specifically for monetary policy contexts.

Despite these limitations, the methodology provides a scalable framework for analyzing central bank communication.

10. Conclusion
This project demonstrates that Chair Powell’s press conference communication is systematically forward-looking, neutral in tone, and characterized by deliberate uncertainty. By combining GPT-based extraction, pre-trained subjective classifiers, and fine-tuned DistilBERT models, the analysis offers a reproducible approach to studying how central bankers communicate policy without making explicit commitments.

Appendix A: Q&A Extraction Prompt
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

Appendix B: Forward-Looking Prompt
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

Appendix C: Certainty Prompt
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

Appendix D: Hawkish/Dovish Prompt
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
