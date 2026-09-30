# TakeMeter: r/TrueFilm Discourse Classifier

[![Demo Video](https://youtu.be/d-1TgFe5Dk4)]

TakeMeter is an NLP machine learning project built for **AI201** that classifies film discussion posts from `r/TrueFilm` into a 3-class discourse taxonomy (`thematic_analysis`, `technical_critique`, `surface_opinion`). 

The project evaluates whether a fine-tuned transformer encoder (**DistilBERT**) outperforms a zero-shot LLM baseline (**Groq / Allam-2-7B**) on complex, subtle human discourse boundaries.

---

## 1. Project Overview & Community Choice

* **Community:** `r/TrueFilm` (Reddit)
* **Goal:** Automatically categorize film discourse to filter low-effort posts and highlight high-value academic film critiques.
* **Target Metric:** Macro F1 $\ge 0.70$.

### Label Taxonomy

| Label | Definition | Example 1 | Example 2 |
| :--- | :--- | :--- | :--- |
| **`thematic_analysis`** | Explores core themes, subtext, symbolism, character psychology, or allegories. | *"The recurring water imagery in Parasite symbolizes class mobility and uncontrollable environmental degradation."* | *"Midsommar is fundamentally an allegory for codependency and tragic emotional catharsis."* |
| **`technical_critique`** | Evaluates filmmaking mechanics: cinematography, sound design, editing, color grading, blocking. | *"The match cuts in the opening 10 minutes establish a rhythmic montage that mirrors the protagonist's heartbeat."* | *"The decision to shoot in native 1.43:1 IMAX ratio enhances the overwhelming sense of vertical scale."* |
| **`surface_opinion`** | Personal preference, emotional reactions, ratings, box-office chat, or general praise/dislike. | *"Honestly couldn't finish it, totally overrated and super boring 2/10."* | *"Just watched this last night and I am absolutely blown away! Easily top 5 movie of the decade."* |

---

## 2. Dataset & Data Collection

* **Source:** Extracted from `r/TrueFilm` via `PRAW` / Reddit API.
* **Total Size:** 210 total rows (Stratified 80/10/10 split: 168 Train / 21 Validation / 21 Test).
* **Class Balance:** Exactly 70 `thematic_analysis`, 70 `technical_critique`, and 70 `surface_opinion`.

### 3 Difficult-to-Label Examples & Annotation Decisions
1. *"The long tracking shot during the battle scene made me feel like I was right there in the mud."*
   * **Decision:** `technical_critique`
   * **Reasoning:** Despite referencing personal immersion ("made me feel"), the post explicitly focuses on camera technique (a long tracking shot).
2. *"I think the director completely failed to make us care about the main character's grief."*
   * **Decision:** `thematic_analysis`
   * **Reasoning:** The criticism targets thematic character execution ("grief") rather than technical camera/editing choices or raw emotional venting.
3. *"A visual masterpiece that suffers from a completely hollow screenplay."*
   * **Decision:** `technical_critique`
   * **Reasoning:** Focuses on the structural separation between visual aesthetic execution and screenplay mechanics.

---

## 3. Model Fine-Tuning & Baseline Setup

### Fine-Tuned Model Setup
* **Base Model:** `distilbert-base-uncased`
* **Training Framework:** Hugging Face `Trainer` PyTorch pipeline.
* **Key Hyperparameters:**
  * **Learning Rate:** `2e-5`
  * **Batch Size:** 16 (Train) / 16 (Eval)
  * **Epochs:** 5 (Early convergence observed at Epoch 3)
  * **Optimizer / Weight Decay:** AdamW with weight decay of `0.01`
  * **Max Sequence Length:** 256 tokens

### Zero-Shot Baseline Setup
* **Model:** Groq API (`allam-2-7b`)
* **Prompt Strategy:** Zero-shot system prompt requiring the model to classify raw post text into one of the 3 exact string categories without chain-of-thought examples.

---

## 4. Full Evaluation Report

### Comparative Metrics Summary

| Model / Approach | Strategy | Overall Accuracy | Macro F1 | Macro Precision | Macro Recall |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **DistilBERT** | Fine-Tuned (5 Epochs) | **1.000** | **1.0000** | **1.0000** | **1.0000** |
| **Allam-2-7B** | Groq Zero-Shot Baseline | 0.762 | **0.7464** | 0.7810 | 0.7619 |

### Confusion Matrix (DistilBERT Test Set)

| True \ Predicted | thematic_analysis | technical_critique | surface_opinion |
| :--- | :---: | :---: | :---: |
| **thematic_analysis** | **7** | 0 | 0 |
| **technical_critique** | 0 | **7** | 0 |
| **surface_opinion** | 0 | 0 | **7** |

---

### Sample Classifications

| Post Text Excerpt | Predicted Label | Confidence Score | Status |
| :--- | :---: | :---: | :---: |
| *"The director's use of low-key lighting and high contrast shadows reflects the noir roots..."* | `technical_critique` | **99.82%** | Correct |
| *"An exploration of patriarchal violence and generational trauma hidden inside a horror film..."* | `thematic_analysis` | **99.64%** | Correct |
| *"I literally fell asleep 30 minutes in. Boring garbage, don't waste your money!"* | `surface_opinion` | **99.91%** | Correct |
| *"The framing in the dinner scene is amazing, showing how isolated everyone is emotionally."* | `technical_critique` | **98.45%** | Correct |

> **Explanation of Correct Prediction:** For the first post, the model correctly assigned `technical_critique` with 99.82% confidence because it identified explicit filmmaking vocabulary ("low-key lighting", "high contrast shadows") as the primary subject, rather than mistaking "noir roots" for pure thematic discourse.

---

### Failure Mode Analysis (Zero-Shot Baseline Failure Patterns)

While DistilBERT achieved 1.000 F1 on the test set, analyzing the **Groq Zero-Shot Baseline failures** reveals critical insights into why the classification boundary is difficult:

1. **Failure 1: Boundary Blur between `technical_critique` and `surface_opinion`**
   * **Text:** *"The editing in the third act was so choppy it gave me a headache and ruined the entire movie for me."*
   * **True Label:** `technical_critique` | **Baseline Predicted:** `surface_opinion`
   * **Analysis:** The zero-shot baseline over-indexed on emotional sentiment keywords ("gave me a headache", "ruined the entire movie") and ignored the core structural critique regarding editing pacing.

2. **Failure 2: Confounding Topic with Structure (`thematic_analysis` vs `surface_opinion`)**
   * **Text:** *"I loved how this film talked about grief. 10/10 favorite movie of the year!"*
   * **True Label:** `surface_opinion` | **Baseline Predicted:** `thematic_analysis`
   * **Analysis:** The zero-shot prompt detected the presence of a thematic noun ("grief") and misclassified the post as thematic analysis, despite the post lacking any actual analytical depth or subtextual breakdown.

3. **Failure 3: Technical Terminology Used as Metaphor**
   * **Text:** *"The narrative pacing feels like a badly edited montage with zero room to breathe."*
   * **True Label:** `thematic_analysis` / Narrative Structure | **Baseline Predicted:** `technical_critique`
   * **Analysis:** The baseline model triggered strongly on explicit technical vocabulary ("edited", "montage") without recognizing that the user was speaking metaphorically about narrative plot progression rather than physical film editing.

---

## 5. Reflection

### What the Model Learned vs. Intended Behavior
The fine-tuned DistilBERT model learned to rely heavily on structural linguistic cues (e.g., presence of formal analytical syntax vs. informal affective language) rather than simple keyword matching. While this perfectly aligned with the intended taxonomy boundaries on our test set, small dataset fine-tuning carries a risk of overfitting to specific Reddit writing tropes. A larger, more noisy dataset would test whether the model overfits to sentence length or formatting styles (e.g., bulleted lists).

### Spec Reflection
* **How the Spec Guided Implementation:** The requirement for a strict, multi-class taxonomy forced clear boundary definitions early in `planning.md`, preventing label drift during dataset creation.
* **Where Implementation Diverged & Why:** The original plan assumed a complex multi-prompt zero-shot strategy for Groq. However, due to API model deprecations, implementation diverged to dynamic runtime model discovery (`client.models.list()`) using `allam-2-7b` to ensure reliable baseline execution.

---

## 6. AI Usage Section

1. **AI Instruction - Evaluation & Error Scripting:**
   * **Directed:** Instructed AI to generate a clean Pandas/Scikit-Learn evaluation script for confusion matrix plotting and dynamic metric extraction.
   * **Produced:** Provided code using `trainer.predict()` which threw an `AttributeError` due to Hugging Face `AccelerateState` re-initialization resets across Colab cells.
   * **Revised/Overrode:** Discarded `trainer.predict()`, wrote a clean PyTorch native inference loop (`model.eval()`, `torch.no_grad()`) to compute predictions, and saved `confusion_matrix.png` without framework errors.

2. **AI Instruction - Baseline Endpoint Fallback:**
   * **Directed:** Instructed AI to resolve Groq API `404 model_not_found` errors.
   * **Produced:** Hardcoded alternate model strings that were also deprecated on Groq's active endpoint.
   * **Revised/Overrode:** Wrote a dynamic retrieval check using `client.models.list().data` to fetch available models directly from the user account at runtime, guaranteeing a zero-shot baseline execution (`allam-2-7b`).

3. **Annotation Assistance Disclosure:**
   * AI tools were utilized during initial data prep to generate candidate keyword rules for heuristic filtering. Every post was manually reviewed and verified against the taxonomy rules prior to final dataset generation.

---

## 7. Submission Deliverables Checklist
* [x] `planning.md` committed in repository root.
* [x] `takemeter_dataset.csv` committed in repository root.
* [x] `evaluation_results.json` and `confusion_matrix.png` committed in repository root.
* [x] `README.md` complete with all metrics, text confusion matrix, failure mode analysis, and reflections.
* [x] Demo video recorded and linked at top of README.