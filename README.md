# TakeMeter

TakeMeter is a fine-tuned text classifier that categorizes Reddit posts and comments from r/PokemonZA into four discourse-style labels — **spitting facts**, **valid but mid**, **ragebait**, and **yapping** — and compares that fine-tuned model against a zero-shot LLM baseline.

The labels are about the *style and substance* of a take, not whether it's positive or negative about the game. A glowing review can be yapping. A harsh critique can be spitting facts.

---

## Community Choice

I chose **r/PokemonZA** as the primary source because I'm personally familiar with the game and community norms, which made annotation judgment calls faster and more consistent. I also picked this window because *Pokémon Legends: Z-A* had released not long before, so discourse was dense and varied — reviews, complaints, defenses, and shitposts were all active simultaneously rather than spread thin over time.

---

## Dataset & Taxonomy

The dataset is 200 manually annotated posts/comments, perfectly balanced at 50 examples per label. Each post or comment is labeled as a single unit — if a post has a title and body, they're labeled together as one example (the title provides context for the body); a comment is labeled on its own, even though the post it's replying to may be used for context.

Split (stratified 70/15/15, `random_state=42`):

| Split | Size | Per-class |
|---|---|---|
| Train | 140 | 35 / 35 / 35 / 35 |
| Validation | 30 | — |
| Test | 30 | 8 spitting facts / 8 ragebait / 7 valid but mid / 7 yapping |

### Label definitions

**`spitting facts`** — Engages with specific, checkable details about the game (mechanics, numbers, named interactions, specific scenarios) and uses them to support a claim that could change someone's understanding. The poster demonstrates they've actually tested or closely observed what they're describing, rather than just reacting to it.

*Examples:*
1. "The combat is fast paced on the player side, but I find the AI lacking in the early parts of the game. It is 100% possible this will change in the future. You can literally spam attacks (yes, there are cooldowns) a lot faster than the NPCs do, so the fights aren't super terrible unless you're at a serious type disadvantage."
2. "Larvitar and Dratini are both rare spawns but they behave very differently when you approach them in the overworld. Walking up to a Dratini startles it and it immediately tries to run away... Larvitar however will battle you on sight."

**Uncertain case:** "The controls don't feel great in handheld mode. There's a lot of trigger button(s) usage/holding down that isn't typical of a Pokemon game and you may not be used to. I feel like it'll play and 'feel' better controlled in Docked mode. But this is more of an opinion than fact."

**`valid but mid`** — Names real features (battle system, customization, companion mechanic), but the praise or complaint echoes the common sentiment about the game rather than surfacing something new.
*Key signal: could this information be predicted just by knowing the most common opinions about this game?*

*Examples:*
1. "I wish there was more to do after the credits roll, but I had a blast with the main story."
2. "I was on the hate train when this came out, but after playing it, it's one of my favorites. My only complaint is there aren't many cool clothing options for male characters."

**Uncertain Case:** "It's mid. Played it, beat it, moved on. Not mad I bought it but not really thinking about it either." Posts that gesture at a verdict, but provide little reason or evidence as to why the user feels this way.

**`ragebait`** — Structured to provoke an angry or defensive reaction rather than to inform or persuade. Often uses exaggeration, a strawman of the opposing view, or deliberately inflammatory framing; the goal seems to be reaction (comments, downvotes, arguing) rather than getting a point across.
*Key signal: does the post seem built to make people angry or argue back, more than to make a real point?*

*Examples:*
1. "This is literally the worst Pokemon game ever made and if you disagree you haven't played a real Pokemon game."
2. "I liked the city but hated the scaffolding. The game designers need to DESIGN the game to not have copy paste nonsense... cheap and unimaginative."

**Uncertain Case:** "I'm sorry but if you think Lumiose City was 'big enough,' I genuinely don't know what to tell you." The user's post is dismissive but responding to a real ongoing debate rather than inventing a new one or providing a new opinion.

**`yapping`** — Doesn't engage with any specific, checkable detail about the game, and doesn't stake out a real verdict either — it talks around the game (the poster's habits, unrelated tangents, pure filler) rather than about it.
*Key signal: if you removed everything that isn't actually about this game's content or quality, would anything be left?*

*Examples:*
1. "Like I'm not saying it's good or anything but I will gladly buy a new Pokemon game bc I js really like the franchise."
2. "Tbh I don't really care just cause I still enjoy the games and never really found an issue with em."

**Uncertain Case:** A short, calm post that's hard to distinguish from Valid but Mid because it lacks intensity.

Each label's documented "uncertain case" was used to keep annotation consistent at the boundary.

---

## Data Collection & Labeling Process

**Source:** Posts and comments were pulled from multiple Pokémon-adjacent subreddits — primarily r/PokemonZA, with additional examples from r/LegendsZA, r/pokemon, and r/nintendo — to get a wider range of discourse styles than a single subreddit would produce, while keeping all examples focused on Legends Z-A discussion.

**Collection method:** I manually browsed each subreddit — no script or API pull was used. I read through posts and comments directly and copied over the ones that contained an actual opinion or take on the game, skipping pure questions, memes without commentary, and off-topic content.

**Labeling process:** I labeled all 200 examples myself, targeting roughly 50 examples per label to keep the dataset balanced. I used the taxonomy's key-signal questions (e.g., "could this be predicted from common opinions?" for valid but mid) as a consistency check during labeling, especially at label boundaries.

**Label distribution:** perfectly balanced, 50 examples per label (200 total), stratified 70/15/15 into train/val/test (35/35/35/35 train, 30 val, 8/8/7/7 test — see split table above).

### Difficult-to-Label Examples

**Example 1:** "I actually think 7/10 is very fair and feels about right to me. At first the monotonous look of Lumiose City and the Wild Zones threw me off but after [...]"
- **Labeled as:** `spitting facts`
- **Why it was hard:** the post opens with a numeric rating, which on its own reads like a generic verdict closer to `valid but mid`. What pushed it to `spitting facts` was the second half — the poster describes a specific change in their own assessment over time ("at first... but after...") tied to a named location (Lumiose City, the Wild Zones), which is the "tested or closely observed" signal the taxonomy asks for. The boundary case is essentially: is the rating doing the work, or is the observed detail behind it doing the work? I judged the latter.

**Example 2:** "Tbh the gaming community has just been force hating a lot of games lately. I am so tired of ppl listening to these content creators like their word is gospel..."
- **Labeled as:** `ragebait`
- **Why it was hard:** this post has no concrete claim about the game itself at all — by the `yapping` key signal ("if you removed everything not about the game, would anything be left?"), almost nothing is left, which made `yapping` tempting. But the post is structured to needle a specific group ("ppl listening to these content creators") rather than ramble without direction — it's aimed *at* someone in a way built to provoke pushback, which is the actual `ragebait` signal ("built to make people angry or argue back"). I weighted the directed, provocative framing over the lack of game-specific content and labeled it `ragebait`. (The fine-tuned model got this one wrong, predicting `yapping` at only 0.32 confidence — which tracks with how genuinely close this boundary is.)

**Example 3:** "I'm taking my time with it, but I'm enjoying it so far. I do like how they have changed some things from legends arceus. Just dont go into it thinking..."
- **Labeled as:** `yapping`
- **Why it was hard:** the post references a real comparison point (changes from *Legends Arceus*), which made `valid but mid` a real candidate — naming an actual feature/change is exactly what that label looks for. But the post never specifies *what* changed or whether it's good or bad; it stays at the level of "I'm enjoying it, some things changed" without committing to a verdict or a checkable claim. Per the `yapping` key signal, stripping out the vague enthusiasm and the dangling "just don't go into it thinking..." leaves nothing concrete about the game's content, so I labeled it `yapping` over `valid but mid`.

---

## Methodology

**Fine-tuned model:** `distilbert-base-uncased` with a 4-way classification head, tokenized at `max_length=256`.

**Hyperparameters:** the notebook's defaults (3 epochs, batch size 16) are tuned for datasets of 100–500 examples. With only 140 training examples spread across 4 classes (35/class), I increased to **4 epochs** and reduced the **train batch size to 8**, to give the randomly-initialized classification head more gradient updates and more optimizer steps per epoch to learn from a small dataset. Learning rate (2e-5), weight decay (0.01), and warmup steps (50) were left at the provided defaults.

**Baseline:** zero-shot classification using `llama-3.3-70b-versatile` via the Groq API, prompted with the full label definitions and one example per class, temperature 0. The baseline saw 30/30 parseable responses on the test set.

**System prompt used:**

```
SYSTEM_PROMPT = """
You are classifying Reddit posts and comments from the Pokemon Fandom.
Assign each post to exactly one of the following categories.
spitting facts: A post engages with specific, checkable details about the game (mechanics, numbers, named interactions, specific scenarios) and uses them to support a claim that could change someone's understanding.
Example: "The combat is fast paced on the player side, but I find the AI lacking in the early parts of the game. It is 100% possible this will change in the future. You can literally spam attacks (yes, there are cooldowns) a lot faster than the NPCs do, so the fights aren't super terrible unless you're at a serious type disadvantage."
valid but mid: A post names real features (battle system, customization, companion mechanic), but the praise or complaint echoes the common sentiment about the game rather than surfacing something new.
Example: "I love how much more challenging it has been than past games, I love the format, I love the setting, the story was great and was longer than most pokemon games to get through for me personally. Now I'm getting absolutely decimated trying my hand at online Z-A and am just starting mega dimension and loving both of those."
ragebait: A post is structured to provoke an angry or defensive reaction rather than to inform or persuade.
Example: "I liked the city but hated the scaffolding. The game designers need to DESIGN the game to not have copy paste nonsense."
yapping: A post doesn't engage with any specific, checkable detail about the game, and doesn't actually stake out a real verdict on it either — it talks around the game (the poster's habits, unrelated tangents, pure filler) rather than about it.
Example: "Like I'm not saying it's good or anything but I will gladly buy a new pokemon game bc I just really like the franchise."
Respond with ONLY the label name.
Do not explain your reasoning.
Valid labels:
spitting facts
valid but mid
ragebait
yapping
"""
```

**Collection process:** for each of the 30 test-set examples, the post/comment text was inserted into a user message, sent to the Groq API (`llama-3.3-70b-versatile`) at temperature 0, and the single-label response was parsed directly (the system prompt enforces label-only output). All 30 responses were parseable without retry.

---

## Evaluation Report

### Overall accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq, llama-3.3-70b-versatile) | **0.767** |
| Fine-tuned DistilBERT | **0.433** |

Fine-tuning produced a **33.3-point regression** relative to the zero-shot baseline. This is the headline result of this project and is discussed in depth in the Reflection section below — the short version is that 140 training examples were not enough for a from-scratch classification head to learn an intent-based distinction that a 70B-parameter model could infer directly from the taxonomy text alone.

### Per-class metrics — Baseline (Groq)

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| spitting facts | 1.00 | 0.62 | 0.77 | 8 |
| yapping | 0.75 | 0.86 | 0.80 | 7 |
| ragebait | 0.78 | 0.88 | 0.82 | 8 |
| valid but mid | 0.62 | 0.71 | 0.67 | 7 |
| **macro avg** | 0.79 | 0.77 | 0.76 | 30 |
| **weighted avg** | 0.79 | 0.77 | 0.77 | 30 |

### Per-class metrics — Fine-tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| spitting facts | 0.62 | 1.00 | 0.76 | 8 |
| yapping | 0.40 | 0.57 | 0.47 | 7 |
| ragebait | **0.00** | **0.00** | **0.00** | 8 |
| valid but mid | 0.25 | 0.14 | 0.18 | 7 |
| **macro avg** | 0.32 | 0.43 | 0.35 | 30 |
| **weighted avg** | 0.32 | 0.43 | 0.36 | 30 |

The fine-tuned model never once correctly predicted ragebait on the test set, despite ragebait having a full 35 training examples — the same count as every other class.

### Confusion matrix — Fine-tuned model (test set, n=30)

| True ↓ \ Predicted → | spitting facts | yapping | ragebait | valid but mid |
|---|---|---|---|---|
| **spitting facts** | 8 | 0 | 0 | 0 |
| **yapping** | 0 | 4 | 2 | 1 |
| **ragebait** | 1 | 5 | 0 | 2 |
| **valid but mid** | 4 | 1 | 1 | 1 |

(See `confusion_matrix.png` for the rendered heatmap version of this same data.)

---

## Error Analysis

Of 30 test examples, the fine-tuned model got **17 wrong (43.3% error rate)**. The errors are not evenly spread across the label space — they cluster into two dominant, directional patterns.

### Pattern 1: Ragebait → Yapping is the single biggest failure mode

5 of 8 ragebait posts were predicted as yapping, and the model's overall ragebait recall is 0.00. Two representative examples:

> **#8** — *"Pretty low standards creeped in this community. Sad."*
> True: `ragebait` · Predicted: `yapping` (confidence: 0.32)

> **#6** — *"Im ngl, the game looks terrible, I wouldnt doubt if most of the reviews are paid off. But I feel like the ppl that act like it's good are forcing themselves too hard"*
> True: `ragebait` · Predicted: `yapping` (confidence: 0.33)

**Which labels are confused:** ragebait → yapping, directionally, in 5 of the 6 ragebait errors.

**Why this boundary is hard:** the taxonomy distinguishes these labels by *intent* — ragebait is "structured to provoke," yapping "doesn't engage with any specific detail and doesn't stake out a verdict." But both labels' surface form looks similar when a post is short: short + low-detail + dismissive. #8 is two blunt sentences with no checkable claim — by the yapping definition's own "if you removed everything that isn't about the game, would anything be left?" test, it's tempting to call it yapping too. The actual distinguishing signal is that #8 is aimed *at someone* (the implied "you" who has low standards) in a way built to needle, not to ramble — the spec's own framing of ragebait as response-shaped rather than content-shaped. #6 adds a second failure on top: a sarcastic, cynical tone ("paid off," "forcing themselves") that reads structurally as meandering rather than as a deliberate jab, because it doesn't use the inflammatory/strawman markers (ALL CAPS, "if you disagree you haven't played a real game") from the taxonomy's own ragebait examples.

**Labeling problem or data/boundary problem:** the original taxonomy labels for these examples are internally consistent with the written definitions (both are genuinely "structured to provoke," not filler) — so this isn't an annotation inconsistency. It's a **boundary the model never learned**, almost certainly because 35 ragebait training examples skewed toward the more obvious, inflammatory style (the taxonomy's own provided examples are both long-form and explicitly combative), leaving the model with no anchor for *short, blunt, sarcastic* ragebait. With confidences sitting at 0.32–0.34 (barely above the 0.25 chance baseline for 4 classes), the model isn't confidently wrong here — it's guessing, which confirms this is a true data-coverage gap rather than a mislearned but confident rule.

**What would fix it:** more ragebait training examples that are short and low-effort rather than long and rhetorically elaborate, plus a few explicit sarcasm-flagged examples so the model has something to anchor "dismissive tone aimed at a person" against, separate from "rambling tangent aimed at nothing."

### Pattern 2: Valid but mid → Spitting facts, and it's confident

4 of 7 valid-but-mid posts were predicted as spitting facts, including the single highest-confidence error in the entire test set:

> **#2** — *"I have three main issues with it: The campaign is tedious. It's just constant grinding until you unlock the next story mission. It feels unnecessarily padded, especially since the story itself is actu..."*
> True: `valid but mid` · Predicted: `spitting facts` (confidence: **0.67**)

> **#14** — *"On top of my head: - The textures are often low quality, clashing and/or unpolished - The game design is repetitive and uninteresting - There is a complete lack of art direction..."*
> True: `valid but mid` · Predicted: `spitting facts` (confidence: 0.58)

**Which labels are confused:** valid but mid → spitting facts, directionally, in 4 of the 6 valid-but-mid errors, and the two highest-confidence misclassifications in the whole report.

**Why this boundary is hard:** both #2 and #14 use itemized, declarative-sounding structure ("I have three main issues," bulleted complaints), which superficially resembles the spitting-facts taxonomy example's structure (also a list of specific claims: "It is 100% possible... You can literally spam attacks"). But the actual distinguishing test in the taxonomy is whether the claim is something a reader could verify or test themselves, versus whether it just echoes the common consensus opinion about the game (the taxonomy's own "key signal" question for valid but mid). "The campaign is tedious" and "textures are low quality" are common, predictable complaints stated with declarative confidence — they pass the *form* test for spitting facts (specific-sounding, itemized) but fail the *substance* test (they don't surface anything a reader didn't already expect to hear about this game).

**Labeling problem or data/boundary problem:** this looks like a genuine boundary/data problem rather than an annotation error. The labels are correct under the spec's own "strip the tone" and "could this be predicted by knowing common opinions" tests — but those tests require evaluating the *content* of each claim against community consensus, which a from-scratch DistilBERT head trained on 140 examples has no real way to learn; it has much easier access to the surface cue "bulleted, declarative list = facts."

**What would fix it:** more spitting-facts examples that are conversational/unstructured (so the model can't use list-formatting as a proxy), and more valid-but-mid examples that are itemized but still generic, to explicitly break the correlation between "looks like a list of facts" and "is a list of facts."

### A third, smaller pattern: sarcasm reads as sincere argument

> **#15** — *"I genuinely don't know what people think when they say that the city feels alive. Like did they play a completely different game? Did they buy the wrong game and thought it was ZA or something? The ci..."*
> True: `ragebait` · Predicted: `spitting facts` (confidence: 0.50)

This is a mocking, rhetorical-question structure aimed at provoking ("did they buy the wrong game"), but its question-as-argument shape gets read by the model as genuine claim-making rather than as a jab — the same underlying sarcasm-blindness as Pattern 1, just landing on the opposite wrong label. This reinforces that sarcasm/tone detection, not just label-pair confusion, is a standalone weakness independent of which two labels are involved.

---

## Sample Classifications

| # | Text (truncated) | True label | Predicted | Confidence | Correct? |
|---|---|---|---|---|---|
| 1 | "I actually think 7/10 is very fair and feels about right to me. At first the monotonous look of Lumiose City and the Wild Zones threw me off..." | spitting facts | spitting facts | 0.67 | ✅ |
| 2 | "Sue me for thinking a game belonging to the biggest franchise in history could be better lmao" | yapping | yapping | 0.32 | ✅ |
| 3 | "I have three main issues with it: The campaign is tedious. It's just constant grinding until you unlock the next story mission..." | valid but mid | spitting facts | 0.67 | ❌ |
| 4 | "Pretty low standards creeped in this community. Sad." | ragebait | yapping | 0.32 | ❌ |
| 5 | "Im ngl, the game looks terrible, I wouldnt doubt if most of the reviews are paid off. But I feel like the ppl that act like it's good are forcing themselves too hard" | ragebait | yapping | 0.33 | ❌ |

**Why #1 is a reasonable prediction:** the post gives a specific numeric rating (7/10), names a concrete location (Lumiose City, Wild Zones), and describes a change in the poster's own assessment over time ("at first... but after a while") — this is exactly the "tested or closely observed" pattern the spitting facts definition asks for, so a high-confidence correct call here makes sense.

**Worth noting on #2:** the model got this one right, but at 0.32 confidence — barely above the 0.25 random-chance floor for a 4-class problem. This is a useful caution against reading "accuracy" as "understanding": a correct prediction made with near-uniform probability across all four classes isn't evidence the model has learned what yapping actually is, just that it landed on the right guess.

---

## Reflection: What the model captured vs. what I intended it to capture

The taxonomy was designed around **intent and epistemic substance** — is a claim checkable, is a post built to provoke, is an opinion just consensus restated, is there any actual content once you strip the filler. None of those are surface-level textual properties; they require judging what a claim is *doing*, not just what words it uses.

What the fine-tuned model actually learned looks much closer to a set of **structural and lexical proxies** that happen to correlate with the true labels often enough to get decent recall on the easiest class, but collapse everywhere the proxy and the true label diverge:

- **Itemized/declarative structure → "spitting facts,"** regardless of whether the itemized claims are actually novel/checkable (Pattern 2). The model overfit to *format* (bullets, "I have three issues," numbered complaints) rather than to *substance*.
- **Short + low-elaboration → "yapping,"** regardless of whether the post is rambling filler or a blunt jab (Pattern 1). The model conflated "doesn't say much" with "doesn't engage," missing that ragebait can be just as short as yapping — the difference is whether it's aimed at provoking a reaction, which has no clean lexical signature.
- **Sarcasm and rhetorical tone are essentially invisible to the model** — both Pattern 1 and Pattern 3 trace back to the same root cause: tone-as-intent doesn't show up as a learnable signal in 35 examples per class for a lightweight encoder head trained from scratch.

The clearest evidence that this is a **data scarcity problem rather than a flawed taxonomy** is the baseline comparison: the same four label definitions, handed to a 70B-parameter model as instructions rather than learned from 140 labeled examples, produced 76.7% accuracy with reasonable balance across all four classes (worst per-class F1 of 0.67). The taxonomy itself is learnable — DistilBERT with this little data and this few epochs over a from-scratch head simply didn't have enough signal to separate intent-based categories that share a lot of surface vocabulary. If I were to keep iterating, the fix is not redefining the labels but substantially growing — and diversifying the *style* of — the ragebait and valid-but-mid training examples specifically, since those are the two classes where the model's structural shortcuts diverge most sharply from the intended boundary.

---

## Spec Reflection

**How the spec helped:** the taxonomy's "Key Signal" questions for each label (e.g., for valid but mid: *"could this be predicted just by knowing the most common opinions about this game?"*) gave me a concrete, repeatable test to apply during annotation, instead of relying on a vaguer positive/negative gut read. This was especially useful at the valid-but-mid/spitting-facts boundary, where two posts can use nearly identical vocabulary and only differ in whether the claim is genuinely new information — having a written test reduced the chance I'd label similar posts inconsistently across the 200-example set.

**Where my implementation diverged:** the notebook's default hyperparameters (3 epochs, batch size 16) are recommended for datasets of 100–500 examples, but with only 140 training examples split across 4 classes (35 each), I judged that the head needed more optimizer steps to learn anything at all from so little data, so I raised epochs to 4 and lowered batch size to 8. In hindsight, given the test results, this divergence wasn't the actual bottleneck — the real constraint was example count and diversity per class, not training schedule. A more useful divergence next time would be collecting more examples per class rather than tuning epochs/batch size on a fixed 140-example train set.

A second, smaller divergence: the baseline's system prompt only includes one example per label and no key-signal questions or uncertain-case framing, while the full taxonomy I annotated against includes two examples, a key-signal question, and an uncertain-case note per label. I simplified the prompt to keep it concise for the API call, but in hindsight the baseline's strong, balanced performance (worst per-class F1 of 0.67) suggests the 70B model didn't need the extra scaffolding I relied on as a human annotator — it inferred the intent-based distinctions from far less guidance than I had.

---

## AI Usage Disclosure

1. **Label definitions and the 200-post taxonomy** were written entirely by me, without AI assistance. No AI tool was used to draft, suggest, or revise the label definitions, key signal questions, or uncertain-case framing.

2. **Pre-labeling and stress-testing during annotation:** after I had already labeled examples myself, I had an AI tool pre-label and generate synthetic boundary-case examples to stress-test my taxonomy. I reviewed its output against my own labels, and where it disagreed or surfaced a case I hadn't considered, I used that to revise specific labels and sharpen the edge-case rules in the taxonomy (e.g., the "uncertain case" notes for each label were refined this way).

3. **System prompt and hyperparameter evaluation:** I asked an AI tool to evaluate the quality of the Groq classification system prompt and to suggest which training hyperparameters to adjust for better fine-tuning results given my dataset size. Based on its suggestions, I changed `num_train_epochs` from 3 to 4 and `per_device_train_batch_size` from 16 to 8, reasoning that the classification head needed more gradient updates given only 140 training examples. I kept the learning rate, weight decay, and warmup steps at the provided defaults.

4. **Error pattern analysis:** I gave an AI tool (Claude) my list of wrong predictions from the fine-tuned model and asked it to help surface patterns — which label pairs were most confused, whether sarcasm or post length correlated with errors, and whether specific failure clusters pointed to a labeling problem versus a data/boundary problem. The Pattern 1, 2, and 3 analyses in the Error Analysis section above, and the Reflection section, were developed through this process: I provided the raw wrong-prediction text and confidence scores, and the AI tool proposed the directional groupings and root-cause hypotheses, which I then reviewed and incorporated into this README rather than writing the error analysis from scratch unassisted.
