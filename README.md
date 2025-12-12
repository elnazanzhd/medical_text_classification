# Medical Text Classification with Transformers

This project investigates **sentence-level rhetorical role classification** in biomedical research abstracts using modern transformer models. Automatically identifying whether a sentence represents *background, objective, methods, results,* or *conclusion* is essential for large-scale literature review, evidence synthesis, and downstream biomedical NLP tasks such as automated systematic review support.

The model fine-tunes **DistilBERT** on the **PubMed 20k RCT** dataset and includes full training, evaluation, and detailed error analysis.

---

##  Dataset

We use the **PubMed 20k RCT** dataset, a widely used benchmark containing:

- 20,000 randomized controlled trial abstracts  
- Sentence-level annotations with five rhetorical labels  
  - **BACKGROUND**  
  - **OBJECTIVE**  
  - **METHODS**  
  - **RESULTS**  
  - **CONCLUSIONS**

This dataset is challenging due to high semantic overlap between adjacent sections—for example between *background ↔ objective* or *methods ↔ results*—making it a good testbed for fine-grained biomedical NLP.

Dataset link: https://huggingface.co/datasets/pietrolesci/pubmed-20k-rct

---

##  Model

We fine-tune **DistilBERT (distilbert-base-uncased)** for 5-way sentence classification.

Key characteristics:

- Lightweight transformer model suitable for limited hardware  
- Cross-entropy objective  
- Trained on individual sentences without surrounding context  
- Output layer: 5-class linear classifier  

Future work explores domain-specific encoders such as **PubMedBERT** or **BioClinicalBERT** for improved boundary detection.

---

## Training Setup

- **Max sequence length:** 128  
- **Optimizer:** AdamW  
- **Epochs:** 3  
- **Metrics:** Accuracy, Macro-F1  
- **Hardware:** Consumer GPU / trained on a subset due to resource limits  
- **Environment:** See `requirements.txt` for dependencies

---

##  Results

Performance on the validation split:

| Epoch | Train Loss | Val Loss | Accuracy | Macro F1 |
|-------|------------|----------|----------|----------|
| 1     | 0.392      | 0.389    | 0.863    | 0.802    |
| 2     | 0.306      | 0.405    | 0.867    | 0.806    |
| 3     | 0.225      | 0.442    | 0.865    | 0.803    |

These values are consistent with expectations for compact transformer models on this benchmark.  
Macro-F1 is particularly relevant due to class imbalance (e.g., fewer “objective” sentences).

---

##  Error Analysis

The most frequent failure mode was **OBJECTIVE → BACKGROUND** misclassification.  
This is a known difficulty in biomedical discourse segmentation because objective sentences often lack explicit intent markers.

Below are **five representative misclassified samples** (true label: objective, predicted: background):

---

### **Example 1**
> “Opioid antagonists (e.g., naltrexone) and positive modulators of GABAA receptors (e.g., alprazolam) modestly attenuate the abuse-related effects of stimulants like amphetamine.”

**Why it fooled the model:**  
Reads like general domain knowledge; lacks any explicit research aim markers.

---

### **Example 2**
> “In a randomised controlled open study, nurses from hospitals and primary healthcare were randomised to either e-learning or classroom teaching.”

**Why it fooled the model:**  
Sentence describes the study setup, but the goal is implicit, not stated.

---

### **Example 3**
> “Previous investigation showed that the volume-time curve technique could be an alternative for endotracheal tube (ETT) cuff management.”

**Why it fooled the model:**  
Classic prior-work sentence typically found in the background section.

---

### **Example 4**
> “The subjects who received the volume-time curve technique for ETT cuff management presented a significantly lower incidence and severity of sore throat and cough…”

**Why it fooled the model:**  
Outcome-heavy phrasing resembles results rather than an objective.

---

### **Example 5**
> “Patients ranged in age from @ years to @ years (mean @ years) and all completed @ year follow-up.”

**Why it fooled the model:**  
Demographic / contextual details often appear in background or methods sections.

---

### **Error Pattern Summary**

Misclassified objective sentences commonly:

- Lack explicit intent cues (e.g., “we aimed to…”)  
- Resemble high-level background statements  
- Contain study context that blurs rhetorical boundaries  
- Require surrounding-sentence context to interpret correctly  

This highlights the limitation of **single-sentence classification** for tasks that depend on discourse structure.

---

##  Limitations & Future Work

- **Loss of context:** Classifying individual sentences ignores abstract-level structure.  
- **Boundary ambiguity:** Many objectives are context-dependent.  
- **Model choice:** Larger or domain-specific models (PubMedBERT, BioClinicalBERT) may reduce errors.  
- **Potential extensions:**  
  - Hierarchical or multi-sentence context models  
  - Sentence-order modeling (e.g., transformers with positional discourse features)  
  - Using abstract-level sequence models (e.g., Longformer, Hierarchical BERT)

---

## Reproducibility

To reproduce the results:

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
