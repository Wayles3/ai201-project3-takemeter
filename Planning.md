# TakeMeter Planning Document: r/TrueFilm Discourse Classifier

## 1. Community Selection
- **Target Community:** r/TrueFilm (Reddit)
- **Why it fits:** Discourse on r/TrueFilm ranges from structured film critique to short emotional reactions. Distinguishing between deep thematic breakdowns, technical scene craft, and surface opinions allows for automated essay discovery and moderation.

## 2. Label Taxonomy
- `thematic_analysis`: Explores underlying themes, subtext, motifs, structural narrative, or allegorical interpretations.
  - *Example 1:* "The motif of mirrors in Persona represents dual identity and subconscious fragmentation."
  - *Example 2:* "Eggers uses isolation in The Lighthouse as a psychoanalytic commentary on masculine hubris."
- `scene_critique`: Focuses specifically on technical or cinematic execution (cinematography, editing, sound design, lighting, acting performance).
  - *Example 1:* "The wide-angle 35mm framing in The Zone of Interest emphasizes detached observation."
  - *Example 2:* "Diegetic sound design in Sound of Metal creates sensory immersion."
- `surface_opinion`: Expresses high-level preference, pacing feel, emotional reaction, or star ratings without structural or technical evidence.
  - *Example 1:* "Honestly I thought the pacing was sluggish and completely overrated."
  - *Example 2:* "Best sci-fi movie of the decade, hands down 10/10."

## 3. Hard Edge Cases & Boundary Decision Rules
- **Boundary 1 (`thematic_analysis` vs `scene_critique`):** A post analyzes how lighting or camera movements highlight a theme.
  - *Decision Rule:* If technical craft is cited as secondary support for an overarching thematic thesis, label as `thematic_analysis`. If the post focuses primarily on the technical accomplishment itself, label as `scene_critique`.
- **Boundary 2 (`scene_critique` vs `surface_opinion`):** A short post claims acting or cinematography was "bad" or "good".
  - *Decision Rule:* Must cite at least one specific technical detail (e.g., lens choice, color grade, blocking, score) to be `scene_critique`. General praise/criticism defaults to `surface_opinion`.

## 4. Data Collection Plan
- **Source:** r/TrueFilm top/new posts and high-engagement comment threads.
- **Target Count:** ~210–220 posts (~70–75 per label to maintain class balance).
- **Imbalance Mitigation:** If `surface_opinion` dominates, collect posts directly from essay-style threads to boost `thematic_analysis` and `scene_critique`.

## 5. Evaluation Metrics & Success Criteria
- **Metrics:** Overall Accuracy, Per-Class Precision, Recall, and F1-Score (Macro F1).
- **Metric Justification:** Accuracy alone is insufficient because class distributions or text length bias can skew results. Macro F1 ensures high performance across all three categories.
- **Success Threshold:**
  - Macro F1 $\ge 0.70$ across all 3 classes on the test set.
  - Fine-tuned DistilBERT must outperform zero-shot Llama 4 Scout baseline by $\ge 10\%$ Macro F1.

## 6. AI Tool Plan
- **Label Stress-Testing:** Prompt Claude/ChatGPT with taxonomy definitions to generate 10 ambiguous posts to test decision rules before annotation.
- **Annotation Assistance:** Pre-label dataset using Groq LLM API, followed by 100% manual review and correction.
- **Failure Analysis:** Feed test set misclassifications into an LLM to identify error patterns (e.g., length bias, sarcasm).

## 7. Stretch Feature Plan
- **S1 (Inter-Annotator Agreement):** Have a secondary reviewer label 35 test examples; calculate Cohen's Kappa ($\kappa$).
- **S2 (Confidence Calibration):** Bin prediction confidence scores vs. actual accuracy to evaluate calibration.
- **S3 (Error Pattern Analysis):** Analyze systematic failure modes (e.g., post length vs. accuracy).
- **S4 (Deployed Interface):** Build a `Gradio` or `Streamlit` app (`app.py`) for live text classification.
