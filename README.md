# Medical Text Classification with Transformers

This project explores **transformer-based sentence classification for biomedical research abstracts**, specifically predicting the structural role of each sentence (Background, Objective, Methods, Results, Conclusion). Accurate identification of these components supports large-scale literature review, automated evidence extraction, and clinical research summarization.

##  Dataset
We use the **PubMed 20k RCT** dataset, which contains 20,000 randomized controlled trial abstracts annotated at the sentence level.  
Each sentence is labeled with one of five rhetorical roles.  
The task is challenging because adjacent sections often exhibit **high semantic overlap**, making fine-grained classification difficult.

Dataset link: https://huggingface.co/datasets/pietrolesci/pubmed-20k-rct

##  Model
The system fine-tunes **DistilBERT (distilbert-base-uncased)** for sentence-level classification.

- 5 output classes  
- Cross-entropy objective  
- Lightweight transformer baseline suitable for biomedical NLP tasks

##  Training Setup
- **Max sequence length:** 128  
- **Optimizer:** AdamW  
- **Epochs:** 3  
- **Metrics:** Accuracy, Macro F1  
- **Hardware:** Training performed on a subset due to GPU limitations

##  Results

| Epoch | Train Loss | Val Loss | Accuracy | Macro F1 |
|------|------------|----------|----------|-----------|
| 1 | 0.392 | 0.389 | 0.863 | 0.802 |
| 2 | 0.306 | 0.405 | 0.867 | 0.806 |
| 3 | 0.225 | 0.442 | 0.865 | 0.803 |

The model reaches **~86% accuracy** and **~0.80 macro-F1**, consistent with expectations for compact transformer models on this benchmark.

##  Error Analysis
The most common failure mode was:

### **Objective → Background misclassification**

Many sentences describing study aims do not include explicit objective markers (e.g., “we aimed to…”, “the purpose of this study…”). When phrased as general statements, they resemble background context.

Representative misclassified examples (true: objective, predicted: background):

## 🔍 Error Analysis

The dominant failure mode was:

### **Objective → Background misclassification**

This reflects a common boundary ambiguity in biomedical writing: many objective sentences lack explicit goal markers (“we aimed…”, “the purpose of this study…”), and without neighboring sentences, they resemble background context.

Below are **five representative misclassified examples** (true: objective, predicted: background), exactly as produced during analysis:

---

**Example 1:**  
“Opioid antagonists (e.g., naltrexone) and positive modulators of GABAA receptors (e.g., alprazolam) modestly attenuate the abuse-related effects of stimulants like amphetamine.”  
**Why it fooled the model:** Reads like general domain knowledge; no explicit study aim signal.

---

**Example 2:**  
“In a randomised controlled open study, nurses from hospitals and primary healthcare were randomised to either e-learning or classroom teaching.”  
**Why it fooled the model:** Sounds like study context/setup; aim is implicit, not stated.

---

**Example 3:**  
“Previous investigation showed that the volume-time curve technique could be an alternative for endotracheal tube (ETT) cuff management.”  
**Why it fooled the model:** Looks like prior work summary, typical “background” phrasing.

 ## 🔍 Error Analysis

The dominant failure mode was:

### **Objective → Background misclassification**

This reflects a common boundary ambiguity in biomedical writing: many objective sentences lack explicit goal markers (“we aimed…”, “the purpose of this study…”), and without neighboring sentences, they resemble background context.

Below are **five representative misclassified examples** (true: objective, predicted: background), exactly as produced during analysis:

---

**Example 1:**  
“Opioid antagonists (e.g., naltrexone) and positive modulators of GABAA receptors (e.g., alprazolam) modestly attenuate the abuse-related effects of stimulants like amphetamine.”  
**Why it fooled the model:** Reads like general domain knowledge; no explicit study aim signal.

---

**Example 2:**  
“In a randomised controlled open study, nurses from hospitals and primary healthcare were randomised to either e-learning or classroom teaching.”  
**Why it fooled the model:** Sounds like study context/setup; aim is implicit, not stated.

---

**Example 3:**  
“Previous investigation showed that the volume-time curve technique could be an alternative for endotracheal tube (ETT) cuff management.”  
**Why it fooled the model:** Looks like prior work summary, typical “background” phrasing.

**Pattern:**  
Sentence-level models struggle whenever **intent depends on surrounding context**, not only local phrasing.

This highlights the need for models that incorporate **sentence ordering** or **multi-sentence windows**.

## Limitations & Future Work
- Sentence-only classification loses abstract-level structure  
- Future work: hierarchical document models or sliding windows of multiple sentences  
- Using domain-specific encoders (e.g., PubMedBERT, BioClinicalBERT) may reduce boundary ambiguity  
- Evaluating across full abstracts may better reflect downstream usefulness

## Reproducibility
All experiments can be reproduced by installing the dependencies in `requirements.txt` and running the notebook top-to-bottom.  
The notebook includes training, evaluation, and error extraction for transparency.
