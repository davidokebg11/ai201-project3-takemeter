# TakeMeter — Planning Document

## Community
I chose r/nba (the NBA subreddit) because it is one of the most active sports
communities on Reddit, with thousands of posts daily ranging from game reactions
to deep statistical breakdowns. The discourse is varied enough to be interesting
because the same event — a player's performance, a trade, a loss — can generate
purely emotional responses, bold unsupported opinions, and carefully reasoned
arguments all in the same thread. This makes it a strong fit for a classification
task: the distinctions between label types are real, recognizable to community
members, and consistently present in the data.

## Labels

### `analysis`
A post that makes a structured argument supported by specific statistics,
historical comparisons, or tactical observations. The evidence is concrete and
verifiable — not just vibes.

**Example 1:**
"Jayson Tatum's playoff efficiency drops significantly in elimination games —
his TS% goes from 58% in the regular season to 51% in must-win situations over
the last 3 years. That's a real and consistent pattern."

**Example 2:**
"The Nuggets switch everything defensively because Jokic's lateral quickness
is good enough to contain guards on the perimeter. It's not laziness — it's
optimized for their personnel."

---

### `hot_take`
A bold, confident opinion stated without meaningful supporting evidence.
The post asserts a claim — sometimes provocative — but doesn't argue for it.
The claim might even be correct, but the post makes no real case.

**Example 1:**
"LeBron is overrated. Always has been. Couldn't win without a superteam."

**Example 2:**
"The NBA is unwatchable now. Nobody plays defense anymore and the refs decide
every game."

---

### `reaction`
An immediate emotional response to a specific recent event — a game, a play,
a trade announcement. Little to no argument. The post is expressing a feeling
in the moment.

**Example 1:**
"LETS GOOO STEPH CURRY IS UNREAL 🔥🔥🔥"

**Example 2:**
"I can't believe they just traded him. Season is over. I'm done."

---

## Hard Edge Cases

**The hardest boundary: `analysis` vs `hot_take`**

Ambiguous post example:
"LeBron is overrated — his playoff win rate against top-seeded opponents
is below .500."

This could be `hot_take` (accusatory framing, bold claim) or `analysis`
(cites a specific stat).

**Decision rule:** If the post provides specific, verifiable evidence that
would support the claim even if you removed the opinion framing, label it
`analysis`. If the evidence is vague, cherry-picked, or decorative — just
enough to sound credible but not genuinely reasoning — label it `hot_take`.
The one-stat post above uses the stat for effect rather than as part of a
real argument → `hot_take`.

**The second hard boundary: `hot_take` vs `reaction`**

Both can be short and emotional. The difference: `hot_take` makes a general
claim about a player/team/league. `reaction` is tied to a specific moment or
event happening right now.

---

## Data Collection Plan

- **Source:** r/nba on Reddit — post titles and comment text from recent threads
- **Target:** ~70 examples per label (210 total) to maintain balance
- **Method:** Manual copy-paste from Reddit into a CSV spreadsheet
- **If a label is underrepresented:** Go into game-thread comments for more
  `reaction` examples, or post-game discussion threads for more `analysis`

---

## Evaluation Metrics

I will use **accuracy**, **per-class F1**, and a **confusion matrix.**

Accuracy alone isn't enough because the dataset could be imbalanced —
a model that always predicts `hot_take` could score high accuracy while
being useless. Per-class F1 catches this by showing precision and recall
for each label individually. The confusion matrix reveals which specific
label pairs are being confused, which is more actionable than a single number.

---

## Definition of Success

The fine-tuned model should achieve:
- Overall accuracy ≥ 70% on the test set
- No single class with F1 below 0.55
- Meaningful improvement over the zero-shot Groq baseline

A classifier meeting these thresholds would be genuinely useful as a
community moderation or post-quality tool.

---

## AI Tool Plan

**Label stress-testing:** I will give Claude my label definitions and ask it
to generate 10 posts that sit at the boundary between `analysis` and
`hot_take` to test whether my definitions hold up before I annotate 200 examples.

**Annotation assistance:** I may use an LLM to pre-label batches of 20–30
examples at a time using my definitions, then review and correct every label
myself before saving. All pre-labeled examples will be marked in a notes column.

**Failure analysis:** After fine-tuning, I will paste my misclassified examples
into Claude and ask it to identify common patterns — then verify those patterns
myself before writing them into the evaluation report.
