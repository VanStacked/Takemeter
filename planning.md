# TakeMeter Planning — r/ValorantCompetitive

## Community Choice

r/ValorantCompetitive is a subreddit dedicated to professional Valorant esports —
VCT matches, team rosters, player performance, and high-level meta discussion.
I chose this community because I play Valorant at Ascendant–Radiant rank and
regularly follow VCT team rosters and results, which makes me a strong annotator
for this task. The discourse is varied in quality: some posts make structured
arguments backed by match data, others are confident opinions with no support,
and others are pure emotional reactions to match results. These distinctions are
meaningful to people in this community — regulars can immediately tell the
difference between someone who watched the match and someone just reacting to
a score.

## Label Taxonomy

### `analysis`
**Definition:** The post makes a structured argument grounded in specific evidence
from pro play — match results, agent/map picks, player stats, team compositions,
tournament format, or roster logic. The reasoning could be verified or challenged
by someone else using the same evidence.

**Example 1:**
"Loud's success at Masters is built on Aspas playing Chamber as a lurk anchor
rather than an entry fragger — it forces opponents to always leave someone on
flank, which opens up their 4-man executes on the other side."

**Example 2:**
"The reason NA keeps underperforming at international events is structural —
VCT Americas scheduling compresses the season too much for teams to do meaningful
mid-year adjustments."

---

### `hot_take`
**Definition:** A bold, confident opinion about a team, player, or the meta stated
without supporting evidence. The claim might be correct, but the post asserts
rather than argues. Common tells: "is overrated," "will never win," "best in the
world," absolutist language.

**Example 1:**
"TenZ is the most overrated player in VCT history. He's never shown up when it
actually mattered."

**Example 2:**
"Sentinels will never win an international event no matter who they sign.
It's an org culture problem."

---

### `reaction`
**Definition:** An immediate emotional response to a specific match result, roster
move, or announcement. The post expresses excitement, disappointment, or
disbelief — not a broader argument. It only makes sense right after the event.

**Example 1:**
"I cannot believe NRG just lost that. They had a 10-4 half lead. I'm actually sick."

**Example 2:**
"WAIT THEY SIGNED HIM?? This roster is going to be insane."

---

## Hard Edge Cases

### Edge case 1 — Reaction that slides into a hot take
**Example:**
"I can't believe they lost that match — they've been choking on big stages all
year, this team just doesn't have what it takes internationally."

**Why it's hard:** The post starts as an emotional reaction to a specific event
but ends with a general claim about the team's character that goes beyond the
moment.

**Decision rule:** If the post's main point is a general claim about a team or
player that extends beyond the immediate event, label it `hot_take`. If it's
purely about the moment with no broader claim, label it `reaction`.

### Edge case 2 — Hot take with one decorative stat
**Example:**
"Derke is the best player in the world right now — his ACS at this tournament
has been unreal."

**Why it's hard:** The post cites a stat, which looks like evidence, but it's
a single number dropped to sound credible rather than part of a real argument.

**Decision rule:** A single stat dropped to sound credible, with no actual
argument built around it, is still `hot_take`. If the post uses multiple data
points and explains the *why*, label it `analysis`.

### Edge case 3 — Structured argument with unverified claims
**Example:**
"Lev having 4 players being able to play Omen made me think that's why they
are the best team in the world right now... aspas would NEVER play anything
but his 2 agents and that's why the team sucks."

**Why it's hard:** The post has logical structure and cites specific examples,
which looks like `analysis`. But the claims are asserted from observation rather
than backed by verifiable data (pick rates, match results, stats).

**Decision rule:** Ask — could someone *verify or challenge* the core claim
using external evidence? If the post would require real data to support it
and doesn't provide that data, label it `hot_take` even if it has structure.
Structure alone is not analysis. The test is verifiability, not length or
logical form.

---

## Data Collection Plan

**Source:** r/ValorantCompetitive — public posts and top-level comments.
I will collect manually by browsing the subreddit and copy-pasting into a CSV.

**Target distribution:**
- `analysis`: ~70 examples
- `hot_take`: ~70 examples
- `reaction`: ~70 examples
(Total: ~210 examples, leaving a small buffer)

**Collection strategy by label:**
- `reaction`: Collect from recent VCT match threads — top-level comments are
  almost pure reaction and one busy thread yields 30–40 examples quickly.
- `hot_take`: Browse Discussion flair posts — look for titles with "unpopular
  opinion," claims about players being overrated, or replies in Roster News
  threads.
- `analysis`: Filter by long-form Discussion posts asking "why" or "how" —
  posts referencing specific maps, agent compositions, or round economics.

**If a label is underrepresented after 200 examples:** I will search the subreddit
specifically for that label type — e.g., for more `analysis` posts, filter by
tournament discussion threads; for more `reaction`, look at match threads from
recent VCT events.

**CSV format:** Three columns — `text`, `label`, `notes`.
The `notes` column will flag any examples I was uncertain about during labeling.

---

## Evaluation Metrics

I will use the following metrics:

- **Overall accuracy** — to get a single number for direct baseline vs.
  fine-tuned comparison.
- **Per-class F1** — because accuracy alone hides class-level failures.
  If my model learns `hot_take` well but completely misses `reaction`, accuracy
  won't show that. F1 balances precision and recall for each class.
- **Confusion matrix** — to identify which specific label pairs the model
  confuses and in which direction. I expect `hot_take` and `reaction` to be
  the hardest pair to separate, since a reaction post that slides into a general
  claim looks very similar to a hot take at the surface level.

Accuracy alone is insufficient here because the task has three classes and
the interesting failure modes are class-specific, not global.

---

## Definition of Success

I will consider this classifier genuinely useful if:
- Fine-tuned model accuracy exceeds the zero-shot baseline by at least 10
  percentage points
- Per-class F1 ≥ 0.65 for all three labels
- No single label has F1 below 0.50 (a model that can't learn one boundary
  at all is not useful)

A classifier meeting these thresholds could realistically be used to auto-tag
posts in a community tool or flag low-quality discourse for moderators.

---

## AI Tool Plan

### Label stress-testing
I will give Claude my three label definitions and all three edge case rules and
ask it to generate 10 posts that sit at the boundary between `hot_take` and
`reaction` — the hardest pair. If it produces posts I can't cleanly classify,
I will tighten my definitions before annotating 200 examples.

### Annotation assistance
I will use an LLM to pre-label batches of ~50 examples at a time by providing
my label definitions and asking for one label per post. I will review and
correct every pre-assigned label before accepting it. I will track which examples
were pre-labeled in the `notes` column of my CSV (marked as "pre-labeled") and
disclose this in my AI usage section.

### Failure analysis
After fine-tuning, I will paste all misclassified examples into Claude and ask
it to identify common patterns — post length, sarcasm, specific label pairs,
recency of the event referenced. I will then verify those patterns myself by
re-reading the examples before writing my evaluation report.

---

## Notes / Running Decisions

### 2025-06-23
- Finalized three labels: `analysis`, `hot_take`, `reaction`
- Decided against a `complaint` label — r/ValorantCompetitive skews toward
  engaged fans discussing pro play, not ranked frustration venting
- Decided against a `question/help` label — that content is rare on this sub
  compared to r/Valorant
- Core design principle: the test for `analysis` vs `hot_take` is
  verifiability, not length or logical structure
