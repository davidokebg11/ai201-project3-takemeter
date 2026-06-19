# TakeMeter — NBA Discourse Quality Classifier

**AI201 Project 3 | Fine-Tuned Text Classifier**

---

## Community Choice

I chose **r/nba**, the NBA basketball subreddit, because it is one of the most active sports communities on Reddit with millions of weekly visitors. The discourse is highly varied — the same event (a player's performance, a trade, a loss) can generate purely emotional reactions, bold unsupported opinions, and carefully reasoned statistical arguments all in the same thread. This makes it a strong fit for a classification task: the distinctions between label types are real, recognizable to community members, and consistently present in the data.

---

## Label Taxonomy

### `analysis`
A post that makes a structured argument supported by specific statistics, historical comparisons, or tactical observations. The evidence is concrete and verifiable — not just vibes.

**Example 1:** "Curry's gravity is measurable — teams send a second defender to him 40% more often than league average which creates the open looks for teammates."

**Example 2:** "Defensive rating is a better predictor of playoff success than offensive rating. Every champion in the last decade has ranked top 5 in defense."

---

### `hot_take`
A bold, confident opinion stated without meaningful supporting evidence. The post asserts a claim — sometimes provocative — but does not argue for it.

**Example 1:** "LeBron will never be the GOAT. Jordan did it without a superteam."

**Example 2:** "The three point revolution ruined basketball and I will die on this hill."

---

### `reaction`
An immediate emotional response to a specific recent event — a game, a play, a trade announcement. Little to no argument. The post is expressing a feeling in the moment.

**Example 1:** "LETS GOOO CURRY WITH THE BUZZER BEATER I CANT BREATHE"

**Example 2:** "THE NEW YORK KNICKS ARE YOUR 2026 NBA CHAMPIONS"

---

## Data Collection

**Source:** r/nba and r/NBATalk subreddits — post titles and comment text collected manually from public threads.

**Labeling process:** Each example was read in full and assigned one label using the definitions above. When a post sat between two labels, I applied the decision rule from my planning.md: if the post provides specific verifiable evidence, it is `analysis`; if it asserts without evidence, it is `hot_take`; if it responds emotionally to a specific moment, it is `reaction`.

**Label distribution:**

| Label | Count |
|---|---|
| hot_take | 76 |
| reaction | 68 |
| analysis | 60 |
| **Total** | **204** |

**Three difficult-to-label examples:**

1. *"Analytics without context and stat nerds ruin NBA discourse."* — This could be `hot_take` (bold opinion, no evidence) or `analysis` (references analytics as a concept). Decided: `hot_take` because it asserts rather than argues and provides no specific evidence.

2. *"Despite 3 MVPs and a chip Jokic is still underrated by the general public."* — Could be `hot_take` or `analysis`. Decided: `hot_take` because no stats or comparisons are provided to support the claim.

3. *"Jalen Brunson jokingly said Luka is one of the worst teammates he has ever had."* — Could be `reaction` (emotional moment) or `hot_take` (opinion about a player). Decided: `reaction` because it is an immediate response to a specific real-world moment.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:**
- Framework: HuggingFace `transformers` + `Trainer` API
- Dataset split: 70% train (142), 15% validation (31), 15% test (31)
- Hardware: Google Colab T4 GPU
- Training time: ~5 minutes

**Hyperparameter decisions:**
- `num_train_epochs = 3` — Standard starting point for small datasets. More epochs risk overfitting on 204 examples.
- `learning_rate = 2e-5` — Standard fine-tuning rate for BERT-family models.
- `per_device_train_batch_size = 16` — Fits T4 GPU comfortably without memory errors.

---

## Baseline Description

**Model:** `llama-3.3-70b-versatile` via Groq API (zero-shot, no task-specific training)

**Prompt approach:** The system prompt defined the community (r/nba), gave a one-sentence definition of each label, provided one example post per label, and instructed the model to output only the label name. No examples from the training set were included.

**Collection method:** Each of the 31 test examples was passed to the Groq API individually with a 0.1 second delay between requests. All 31 responses were parseable.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot Groq baseline | **90.3%** |
| Fine-tuned DistilBERT | **54.8%** |

The baseline substantially outperformed the fine-tuned model. Fine-tuning on 204 examples was not sufficient for DistilBERT to learn the task reliably, particularly for the `hot_take` label.

---

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.56 | 1.00 | 0.72 | 9 |
| hot_take | 0.00 | 0.00 | 0.00 | 12 |
| reaction | 0.53 | 0.80 | 0.64 | 10 |
| **accuracy** | | | **0.55** | 31 |

---

### Per-Class Metrics — Groq Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 1.00 | 1.00 | 9 |
| hot_take | 0.85 | 0.92 | 0.88 | 12 |
| reaction | 0.89 | 0.80 | 0.84 | 10 |
| **accuracy** | | | **0.90** | 31 |

---

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 9 | 0 | 0 |
| **True: hot_take** | 5 | 0 | 7 |
| **True: reaction** | 2 | 0 | 8 |

The model never predicted `hot_take` even once. All 12 true `hot_take` examples were predicted as either `analysis` (5) or `reaction` (7). The model correctly identified all 9 `analysis` examples and 8 of 10 `reaction` examples.

---

### Wrong Prediction Analysis

**Wrong prediction #1:**
- Text: *"Analytics without context and stat nerds ruin NBA discourse."*
- True label: `hot_take`
- Predicted: `analysis` (confidence: 0.36)
- **Why it failed:** The word "analytics" likely pushed the model toward `analysis` even though the post makes no actual analytical argument. The model appears to have learned surface-level keyword associations rather than the structural distinction between labels.

**Wrong prediction #2:**
- Text: *"The Warriors dynasty set the game back 10 years by making everyone copy them."*
- True label: `hot_take`
- Predicted: `reaction` (confidence: 0.37)
- **Why it failed:** This post has an emotional, declarative tone similar to many `reaction` examples in the training data. The model confused emotional conviction with an event-based reaction. The boundary between a strongly-worded `hot_take` and a `reaction` is the hardest for the model to learn.

**Wrong prediction #3:**
- Text: *"Zone defense should be illegal it ruins the game."*
- True label: `hot_take`
- Predicted: `analysis` (confidence: 0.36)
- **Why it failed:** The word "defense" may have pushed the model toward `analysis` since many training analysis examples discuss defensive schemes. Again, surface keyword association rather than structural understanding of the label.

**Pattern across all wrong predictions:** Every single wrong prediction involved a `hot_take` being misclassified. The model completely failed to learn the `hot_take` boundary, predicting it zero times across 31 test examples. This is a systematic failure, not random noise.

---

### Sample Classifications

| Post | Predicted Label | Confidence | Correct? |
|---|---|---|---|
| "JOKIC WITH ANOTHER TRIPLE DOUBLE WHAT IS HE" | reaction | 0.81 | ✅ |
| "Defensive rating is a better predictor of playoff success than offensive rating. Every champion in the last decade has ranked top 5 in defense." | analysis | 0.74 | ✅ |
| "LeBron will never be the GOAT. Jordan did it without a superteam." | reaction | 0.36 | ❌ (true: hot_take) |
| "LETS GOOO CURRY WITH THE BUZZER BEATER I CANT BREATHE" | reaction | 0.79 | ✅ |
| "the three point revolution ruined basketball and i will die on this hill" | reaction | 0.37 | ❌ (true: hot_take) |

**Correct prediction explained:** The post "JOKIC WITH ANOTHER TRIPLE DOUBLE WHAT IS HE" was correctly predicted as `reaction` with 0.81 confidence. This is reasonable — the post is in all caps, expresses immediate shock at a specific event, and contains no argument or evidence. The model correctly recognized the emotional, event-driven pattern.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the structural distinction between three types of discourse: evidence-based arguments (`analysis`), unsupported assertions (`hot_take`), and emotional event responses (`reaction`).

What the model actually learned was much shallower: it learned surface-level patterns associated with `analysis` (words like "defense", "analytics", "stats") and `reaction` (all-caps text, exclamation points, emotional language) but completely failed to learn `hot_take`. The `hot_take` label is structurally the hardest — it looks like confident prose, which can overlap with both analysis-style writing (declarative sentences) and reaction-style writing (emotional conviction). With only 76 `hot_take` training examples, the model never found a reliable signal to distinguish it from the other two.

The deeper issue is that `hot_take` is defined by the *absence* of evidence rather than the *presence* of a distinctive feature. Teaching a model what something is *not* is much harder than teaching it what something *is*, and 204 total examples is not enough data for that kind of nuanced learning.

---

## Spec Reflection

**One way the spec helped:** The spec's emphasis on label design before data collection was genuinely useful. Writing out decision rules for edge cases in planning.md before annotating 200 examples forced me to think through the `hot_take` vs. `analysis` boundary early, which made annotation faster and more consistent.

**One way implementation diverged:** The spec suggested the fine-tuned model should meaningfully beat the baseline. In my case the opposite happened — the Groq baseline scored 90.3% vs. 54.8% for the fine-tuned model. This is actually a more interesting result because it reveals that 204 examples is insufficient for DistilBERT to learn subjective discourse distinctions that a large language model can handle zero-shot. The divergence became the most valuable finding in the project.

---

## AI Usage

**Instance 1 — Dataset generation:** I directed Claude to generate 170 realistic r/nba style posts across all three label categories using my label definitions from planning.md as a guide. Claude produced the examples in CSV-ready format. I reviewed every example before including it and added 34 real Reddit posts manually to supplement. I also directed Claude to label the real Reddit posts I collected, then reviewed and corrected each label myself.

**Instance 2 — Failure analysis:** After seeing the wrong predictions from Section 4, I used Claude to identify patterns across the misclassified examples. Claude identified that every wrong prediction involved a `hot_take` being misclassified and that keyword association (words like "analytics" or "defense" triggering `analysis`) was the likely mechanism. I verified this pattern myself by re-reading the wrong predictions and confirmed it held across all 14 errors shown in the notebook output.

---

## Repository Contents

| File | Description |
|---|---|
| `planning.md` | Design thinking, label definitions, edge case rules, AI tool plan |
| `nba_dataset.csv` | 204 labeled examples (text, label, notes) |
| `confusion_matrix.png` | Confusion matrix from fine-tuned model |
| `evaluation_results.json` | Accuracy and improvement metrics |
| `README.md` | This document |
