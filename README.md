# TakeMeter — r/ValorantCompetitive Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in the
r/ValorantCompetitive subreddit. Given a post or comment, the model predicts
whether it is structured **analysis**, a confident **hot take**, or an immediate
**reaction** to a match or roster event.

---

## Community Choice

**r/ValorantCompetitive** is a subreddit dedicated to professional Valorant
esports — VCT matches, team rosters, player performance, and high-level meta
discussion. I chose this community because I play Valorant at Ascendant–Radiant
rank and regularly follow VCT team rosters and results, which makes me a strong
annotator for this task.

The discourse is genuinely varied in quality: some posts make structured
arguments backed by match data and verifiable evidence, others are confident
opinions stated without support, and others are pure emotional reactions to
match results. These distinctions are meaningful to people in this community —
regulars can immediately tell the difference between someone who actually watched
a match and someone just reacting to the score. That makes this a real and
interesting classification task rather than a toy problem.

---

## Label Taxonomy

### `analysis`
The post makes a structured argument grounded in specific evidence from pro play
— match results, agent/map picks, player stats, team compositions, tournament
format, or roster logic. The reasoning could be verified or challenged by someone
else using the same evidence.

**Example 1:**
"Loud's success at Masters is built on Aspas playing Chamber as a lurk anchor
rather than an entry fragger — it forces opponents to always leave someone on
flank, which opens up their 4-man executes on the other side."

**Example 2:**
"The reason NA keeps underperforming at international events is structural — VCT
Americas scheduling compresses the season too much for teams to do meaningful
mid-year adjustments."

---

### `hot_take`
A bold, confident opinion about a team, player, or the meta stated without
supporting evidence. The claim might be correct, but the post asserts rather than
argues. Common tells: "is overrated," "will never win," "best in the world,"
absolutist language.

**Example 1:**
"TenZ is the most overrated player in VCT history. He's never shown up when it
actually mattered."

**Example 2:**
"Sentinels will never win an international event no matter who they sign. It's an
org culture problem."

---

### `reaction`
An immediate emotional response to a specific match result, roster move, or
announcement. The post expresses excitement, disappointment, or disbelief — not a
broader argument. It only makes sense right after the event.

**Example 1:**
"I cannot believe NRG just lost that. They had a 10-4 half lead. I'm actually
sick."

**Example 2:**
"WAIT THEY SIGNED HIM?? This roster is going to be insane."

---

## Data Collection

**Source:** r/ValorantCompetitive — public posts and top-level comments collected
manually by browsing the subreddit across multiple thread types including match
threads, post-match threads, discussion posts, patch note threads, and roster news
threads.

**Labeling process:** Each example was read individually and assigned one label
using the definitions above. A notes column flagged uncertain cases. No bulk
skimming was used.

**Label distribution:**

| Label | Count | % |
|-------|-------|---|
| analysis | 93 | 42% |
| hot_take | 73 | 33% |
| reaction | 56 | 25% |
| **Total** | **222** | **100%** |

---

### Three Difficult-to-Label Examples

**1. Structured argument with unverified claims**
> "Lev having 4 players being able to play Omen made me think that's why they
> are the best team in the world right now... aspas would NEVER play anything but
> his 2 agents and that's why the team sucks."

This post has logical structure and cites a specific observation, which looks like
`analysis`. But the claims are asserted from observation rather than backed by
verifiable data (pick rates, match results, stats). **Decision:** `hot_take` —
the test is verifiability, not length or logical form.

**2. Reaction that slides into a hot take**
> "I can't believe they lost that match — they've been choking on big stages all
> year, this team just doesn't have what it takes internationally."

Starts as an emotional reaction to a specific event but ends with a general claim
about the team's character that extends beyond the moment. **Decision:** `hot_take`
— the post's main point is a general claim that goes beyond the immediate event.

**3. Hot take with one decorative stat**
> "Derke is the best player in the world right now — his ACS at this tournament
> has been unreal."

Cites a stat, which looks like evidence, but it's a single number dropped to sound
credible rather than part of a real argument. **Decision:** `hot_take` — a single
stat with no reasoning built around it does not constitute analysis.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:**
- 70/15/15 train/validation/test split (155 / 33 / 34 examples)
- 8 epochs
- Learning rate: 2e-5
- Batch size: 16
- Weight decay: 0.01
- Warmup steps: 50

**Key hyperparameter decision:** The default 3-epoch run produced majority class
collapse — the model predicted `analysis` for every example because it is the most
frequent label (42% of data). I added **class weighting** to the loss function to
penalize errors on underrepresented labels more heavily, and increased epochs from
3 to 8 to give the model more time to learn the harder boundaries. Class weights
were calculated as `total / (num_labels × class_count)` per label.

---

## Baseline

**Model:** `llama-3.3-70b-versatile` via Groq API (zero-shot, no task-specific
training)

**Prompt approach:** The system prompt included the community name, a one-sentence
definition of each label with one example post per label, and an instruction to
output only the label name. The model was given no training examples.

**Prompt used:**
```
You are classifying posts and comments from r/ValorantCompetitive, a subreddit
dedicated to professional Valorant esports.

analysis: The post makes a structured argument grounded in specific evidence
from pro play — match results, player stats, agent picks, team compositions,
or roster logic. The reasoning could be verified or challenged by someone else.

hot_take: A bold, confident opinion about a team, player, or the meta stated
without supporting evidence. The post asserts rather than argues.

reaction: An immediate emotional response to a specific match result, roster
move, or announcement. The post expresses excitement, disappointment, or
disbelief — not a broader argument.

Respond with ONLY the label name.
Valid labels: analysis, hot_take, reaction
```

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | **0.647** |
| Fine-tuned DistilBERT | **0.559** |
| Difference | -0.088 (regression) |

### Per-Class Metrics

**Baseline (Groq):**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 0.36 | 0.53 | 14 |
| hot_take | 0.80 | 0.73 | 0.76 | 11 |
| reaction | 0.47 | 1.00 | 0.64 | 9 |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.58 | 0.79 | 0.67 | 14 |
| hot_take | 1.00 | 0.09 | 0.17 | 11 |
| reaction | 0.50 | 0.78 | 0.61 | 9 |

### Confusion Matrix (Fine-Tuned Model)

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|---------------------|---------------------|---------------------|
| **True: analysis** | 11 | 0 | 3 |
| **True: hot_take** | 6 | 1 | 4 |
| **True: reaction** | 2 | 0 | 7 |

See `confusion_matrix.png` for the visual version.

---

### Three Wrong Predictions — Analysis

**Wrong prediction #1**
> "There is nothing you or I can constructively criticize about the play of a
> team that is this successful. Maybe they have some issues closing out big games
> but that's for them to know and work on, not really something that's possible
> to diagnose as an outsider."
>
> True: `hot_take` | Predicted: `analysis` | Confidence: 0.39

This post uses complete sentences and a reasoned structure that reads like
analysis. The model picked up on the logical framing ("not really something that's
possible to diagnose") and classified it as analysis. But there is no verifiable
evidence — it's an opinion about what fans should and shouldn't say. This reveals
a core weakness: the model is responding to **sentence structure** rather than
**verifiability**.

**Wrong prediction #2**
> "gotta say this is the worst tournament for me because it's just team A stomps
> team B and team B stomps team C and C stomps A and so on."
>
> True: `hot_take` | Predicted: `analysis` | Confidence: 0.36

This post gives a reason for its opinion (cyclical results), which gives it the
surface appearance of an argument. The model may have learned that posts with
causal language ("because") tend to be analysis. But the claim is entirely
subjective and unverifiable. This is another case of the model learning
**linguistic markers** (because, therefore, which means) rather than the
distinction between assertion and evidence.

**Wrong prediction #3**
> "sova gets buffed undirectly cause now breaking senti utils becomes more
> important and we're also gonna see more sova with fracture out and sunset in"
>
> True: `analysis` | Predicted: `reaction` | Confidence: 0.35

This is a genuine analysis post — it reasons through indirect meta implications
of a patch on map pool and agent pick rates. The model labeled it `reaction`,
likely because it is short and casual in tone. This shows the model struggles
with **short analysis posts** that don't look like formal arguments even though
they contain verifiable reasoning.

---

### Sample Classifications

The following examples were run through the fine-tuned model:

| Post (truncated) | Predicted Label | Confidence |
|-----------------|-----------------|------------|
| "Loud's success at Masters is built on Aspas playing Chamber as a lurk anchor rather than an entry fragger..." | analysis | 0.61 |
| "TenZ is the most overrated player in VCT history. He's never shown up when it actually mattered." | hot_take | 0.58 |
| "I cannot believe NRG just lost that. They had a 10-4 half lead. I'm actually sick." | reaction | 0.71 |
| "Tejo is probably the only initiator that is op if his util does regenerate. Having 3 mollies a round is kinda op..." | analysis | 0.55 |
| "WHAT IN THE WORLD HAPPENED TO PRX" | reaction | 0.74 |

The first correctly-predicted example ("Loud's success at Masters...") is
reasonable because the post names a specific tactical pattern (Chamber as lurk
anchor), explains the mechanical reason it works (forces flank coverage), and
connects it to a broader strategic consequence (opens 4-man executes). These are
the exact features the `analysis` label is designed to capture.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the distinction between **verifiable reasoning** and
**unverified assertion**. What it actually learned was closer to a distinction
between **long structured text** (classified as analysis), **short emotional
text** (classified as reaction), and almost nothing reliable for **hot_take**.

The confusion matrix makes this clear: `hot_take` has an F1 of 0.17 and the model
only correctly identified 1 out of 11 hot takes. Hot takes were split almost
evenly between being mislabeled as `analysis` (6 cases) and `reaction` (4 cases).
This makes sense — hot takes that use complete sentences look like analysis, and
hot takes that are short and punchy look like reactions.

The deeper issue is that the verifiability distinction I was trying to capture is
**semantic**, not **syntactic**. DistilBERT is good at picking up on surface
patterns — word choice, sentence length, punctuation, emotional language — but the
difference between "this post cites evidence" and "this post just sounds like it
cites evidence" requires understanding what the words are actually claiming, which
is much harder for a model trained on 155 examples.

The zero-shot baseline outperforming fine-tuning is itself revealing. A large
language model with no task-specific training was better at this task than a
fine-tuned small model, which suggests the labels require genuine language
understanding rather than pattern matching. With more data (500+ examples) and a
stronger base model, fine-tuning would likely win.

---

## Spec Reflection

**One way the spec helped:** The requirement to define a decision rule for every
edge case before annotating forced me to write the verifiability test ("could
someone verify or challenge this claim using external evidence?") before I labeled
a single example. This made my annotation far more consistent than it would have
been otherwise and gave me a precise answer for the hardest cases.

**One way implementation diverged from the spec:** The spec suggested fine-tuning
would meaningfully outperform the zero-shot baseline. In my case it did not — the
fine-tuned model regressed by 8.8 percentage points. This happened because 222
examples with class imbalance is not enough for DistilBERT to learn a distinction
that requires semantic understanding rather than surface pattern matching. If I
were doing this again I would collect 400+ examples with a more balanced
distribution (targeting 33% per label rather than 42/33/25).

---

## AI Usage

**1. Label stress-testing:** I gave Claude my three label definitions and asked it
to generate boundary cases between `hot_take` and `reaction`. It produced examples
like "I can't believe they lost — they've been choking all year" which I couldn't
cleanly classify. This forced me to write the decision rule: if the post makes a
general claim that extends beyond the immediate event, label it `hot_take`. I used
this rule dozens of times during annotation.

**2. Annotation assistance:** I pasted batches of 20–40 raw comments into Claude
and asked it to label each one using my definitions. I reviewed and corrected
every pre-assigned label before accepting it. Approximately 180 of 222 examples
were pre-labeled this way and then reviewed. All pre-labeled examples are marked
in the `notes` column of the CSV as implicitly assisted. Cases where I overrode
the pre-label are noted explicitly.

**3. Failure analysis:** After fine-tuning I pasted the 15 wrong predictions into
Claude and asked it to identify common patterns. It identified the structural
language pattern (the model responding to sentence structure rather than
verifiability) and the short-post problem. I verified both patterns by re-reading
the examples myself and confirmed them before writing the analysis above.
