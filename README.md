
# Medical Text Classification with Transformers

This project fine-tunes a transformer-based model to classify sentences from biomedical abstracts into structural sections: background, objective, methods, results, and conclusions.

## Dataset
We use the PubMed 20k RCT dataset, which contains sentence-level annotations from randomized controlled trial abstracts. The task is challenging due to semantic overlap between adjacent sections.

link to dataset: https://huggingface.co/datasets/pietrolesci/pubmed-20k-rct

## Model
- DistilBERT (distilbert-base-uncased)
- Sentence-level classification
- 5 output classes

## Training Setup
- Max sequence length: 128
- Optimizer: AdamW
- Epochs: 3
- Metric: Accuracy and Macro F1
- Training performed on a subset due to hardware constraints

## Results
| Epoch | Train Loss | Val Loss | Accuracy | Macro F1 |
|------|-----------|----------|----------|----------|
| 1 | 0.392 | 0.389 | 0.863 | 0.802 |
| 2 | 0.306 | 0.405 | 0.867 | 0.806 |
| 3 | 0.225 | 0.442 | 0.865 | 0.803 |

## Error Analysis
The most frequent error type was objective → background. It’s a label boundary ambiguity issue: many “objective” sentences are written like general context statements and do not contain explicit goal markers (e.g., “to evaluate…”, “we aimed…”). At sentence-level, without neighboring sentences, these often resemble background.

Below are 5 representative misclassified examples (true: objective, predicted: background) from the validation errors:

“Opioid antagonists (e.g., naltrexone) and positive modulators of GABAA receptors (e.g., alprazolam) modestly attenuate the abuse-related effects of stimulants like amphetamine.”
Why it fooled the model: reads like general domain knowledge, no explicit study aim signal.

“In a randomised controlled open study, nurses from hospitals and primary healthcare were randomised to either e-learning or classroom teaching.”
Why it fooled the model: sounds like study context/setup; aim is implicit, not stated.

“Previous investigation showed that the volume-time curve technique could be an alternative for endotracheal tube (ETT) cuff management.”
Why it fooled the model: looks like prior work summary, typical “background” phrasing.

“The subjects who received the volume-time curve technique for ETT cuff management presented a significantly lower incidence and severity of sore throat and cough…”
Why it fooled the model: outcome-heavy wording resembles results; lack of objective cue confuses boundary.

“Patients ranged in age from @ years to @ years (mean @ years) and all completed @ year follow-up.”
Why it fooled the model: demographic/setting detail reads like background or methods context.


Pattern summary (Objective → Background):

Objective sentence written as general context (no “we aimed / to investigate” markers).

High-level domain statements that could sit in background anywhere.

Sentence-level classification loses abstract-level intent (surrounding sentences often disambiguate).



## Limitations & Future Work
- Sentence-level classification lacks abstract-level context
- Future work could incorporate sentence ordering or multi-sentence windows
- Larger models or domain-specific encoders (e.g., PubMedBERT) may improve boundary cases

## Reproducibility
All experiments can be reproduced by running the notebook top-to-bottom after installing dependencies from `requirements.txt`.
