# AI Detector v4

![Version](https://img.shields.io/badge/version-4.0-blue?style=flat-square)
![Claude](https://img.shields.io/badge/Claude-Opus_4.6+-blueviolet?style=flat-square&logo=anthropic)
![Skill](https://img.shields.io/badge/type-Claude_Skill-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Layers](https://img.shields.io/badge/analysis_layers-6-red?style=flat-square)
![AI Patterns](https://img.shields.io/badge/patterns_tracked-120+-yellow?style=flat-square)
![Language](https://img.shields.io/badge/language-English-lightgrey?style=flat-square)
![GitHub stars](https://img.shields.io/github/stars/LAOUUUUU/Ai-detector-Claude-skill-?style=flat-square)
![GitHub forks](https://img.shields.io/github/forks/LAOUUUUU/Ai-detector-Claude-skill-?style=flat-square)
![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgithub.com%2FLAOUUUUU%2FAi-detector-Claude-skill-&countColor=%23263759&style=flat-square)

A Claude skill that analyzes text for signs of AI-generated writing and produces a detailed detection report with scoring.

Unlike basic AI detectors that just scan for word lists, this skill uses Claude's own capabilities as a language model to estimate how predictable the text is, check for voice consistency, and score each paragraph independently — similar to how tools like Turnitin work, but without needing a separate classifier model.

## What's New in v4

v4 sharpens detection on the two cases v3 was weakest at: **student impersonation** (adult-polished text passed off as a student's) and **humanized AI** (AI text run through a humanizer to beat detectors).

- **Perfection-bias detection** — In student/casual context, formal + flawless + voiceless + zero contractions is now a flag, not a free pass. Real students writing formally keep their voice; robots don't.
- **Contraction-absence flag** — Zero contractions anywhere in a casual or claimed-student piece is suspicious on its own (unless the assignment banned them).
- **Trailing-thoughts signal** — Sentences that trail off or restart ("So...", "Right now they're just... stuck.") are recognized as a strong *human* indicator and pull scores down.
- **Humanized-AI residue layer** — Catches AI that was de-formalized: suspiciously complete topic coverage, even information density, "inserted" casual markers, parallel structure that survived a vocabulary swap, and uniform find-replace contractions. Requires 3+ corroborating signals before it adds weight.
- **Punctuation & formatting fingerprints** — Em-dash density (one of the strongest current ChatGPT tells), curly quotes, inline arrows, and bold-label-colon lists. Scored on density, never on a single occurrence, and discounted in Markdown/technical contexts.
- **Reweighted persona panel** — For student work, the High School English Teacher persona now carries 40% (the rest 20% each); teachers catch impersonation best. Standard text keeps all four equal.
- **Refreshed vocabulary & model fingerprints** — Added current-era tells ("boasts", "it's worth noting", "dive into", "game-changer", "look no further", etc.) and updated ChatGPT / Claude / Gemini fingerprints for the GPT-4o/5, Claude 4.x, and Gemini 2.x era.
- **Confidence band + length-graduated confidence** — Scores now report a ± band, and detection confidence scales with text length (unreliable < 100 words, reduced 100–299, normal 300+).
- **More false-positive guards** — Added explicit handling for translated text and domain/technical writing on top of the existing ESL, neurodivergent, naturally-formal, and instructed-writing adjustments.

The full skill source is in [`ai-detector/SKILL.md`](ai-detector/SKILL.md); the installable package is [`ai-detector.skill`](ai-detector.skill).

## How It Works

The skill runs text through six analysis layers, each weighted and combined into a final score.

### Layer 1 — Sliding Window (25%)

Every paragraph gets scored independently. For each one, Claude asks itself: "If I were given the first sentence as a prompt, how closely does the rest match what I would generate?" Paragraphs get labeled by AI likelihood (Low/Mid/High/Very High) so you can see exactly which sections look human and which don't. v4 adds trailing thoughts and self-interrupted sentences as a strong human signal.

This is the closest thing to Turnitin's per-sentence highlighting you can get without a trained classifier.

### Layer 2 — Voice Consistency (15%)

Tracks formality level, contraction usage, vocabulary complexity, and pronoun shifts across the entire document. This is what catches the most common real-world pattern: AI-generated text with a few human paragraphs edited in (or vice versa). v4 adds an explicit flag for *complete* contraction absence in casual or student work.

If someone writes six paragraphs of formal, contraction-free prose and then drops a casual opinion paragraph at the end with "I think" and "honestly", this layer catches that mismatch.

### Layer 3 — Predictability Estimation (20%)

The closest approximation to perplexity scoring possible without token-level log probabilities. Claude evaluates each sentence and asks whether it would have written the same thing given the context. Also measures burstiness (sentence length variance), sentence opening diversity, and default phrasing density. v4 flags the specific gap where rhythm looks human but the content of every sentence is still fully predictable — a sign the surface was edited but the substance was generated.

### Layer 4 — Pattern Detection (10%)

Traditional heuristic scanning for known AI writing patterns. Three tiers of AI vocabulary (from dead giveaways like "delve" and "tapestry" down to mild signals like "significant" and "notably"), structural patterns (rule of three, uniform paragraph length, summary conclusions, perfection penalty), semantic repetition, information density, source/citation fingerprints, and — new in v4 — punctuation/formatting fingerprints and a humanized-AI residue block.

Also includes model fingerprinting — identifies whether text likely came from ChatGPT, Claude, or Gemini based on each model's specific habits (citation format, vocabulary preferences, punctuation, structural tendencies).

### Layer 5 — Holistic Assessment (10%)

A gut-check pass where Claude steps back and evaluates: Does this text have an actual angle? Are there specific details that couldn't be predicted from the topic alone? Could I reconstruct this entire text from just the title? Does it feel like someone had something to say, or like someone was assigned to write about a topic — or like AI that was lightly de-formalized?

### Layer 6 — Persona Panel (20%)

The text gets evaluated by four simulated expert perspectives, each catching different things:

| Persona | Focus |
|---------|-------|
| **Harvard Professor** | Depth of understanding, argument structure, citation quality |
| **High School English Teacher** | Age-appropriate voice, student writing patterns, authenticity |
| **Hiring Manager** | Genuine intent vs template-filling, specificity of claims |
| **Investigative Journalist** | Source quality, factual claims, narrative originality |

Each persona scores independently. When they disagree significantly, it usually means the text is a mashup of human and AI writing. For student work, the English Teacher persona is weighted at 40% (the rest 20% each), since teachers are best at spotting adult-sounding impersonation.

## Scoring

```
Final = (Sliding Window × 0.25) + (Voice Consistency × 0.15) + (Predictability × 0.20)
      + (Pattern Detection × 0.10) + (Holistic × 0.10) + (Persona Panel × 0.20)
```

| Range | Label |
|-------|-------|
| 0-15% | Likely human-written |
| 16-30% | Mostly human, minor flags |
| 31-50% | Uncertain, mixed signals |
| 51-70% | Likely AI-assisted |
| 71-85% | Strong AI indicators |
| 86-100% | Almost certainly AI-generated |

Scores are reported with a ± confidence band that widens for shorter text. They are pattern-match percentages, not probabilities. The skill never claims certainty — it frames results as likelihood based on detected patterns.

## Output

The full report includes:

- Layer-by-layer score breakdown with explanations (Score = how strongly AI was detected, Weight = how much the layer matters, Contribution = actual points added to final score)
- Paragraph-by-paragraph breakdown with AI likelihood labels (Low/Mid/High/Very High) showing which sections flagged and why
- Voice consistency analysis (formality range, contraction usage, vocabulary drift, pronoun shifts)
- Persona panel table with each expert's score and reasoning, noting the weighting used
- Top 3-5 strongest signals with exact quotes from the text
- Source analysis (when citations are present)
- Plain-English verdict with model identification and a humanized-AI note if applicable

A **quick mode** is also available for fast yes/no checks — just the score, the strongest signal, and a model ID if relevant.

## False Positive Handling

The skill includes explicit adjustments for groups that commonly trigger false positives:

- **ESL writers** — May use formal vocabulary from textbook English, avoid contractions, and stick to safe sentence structures. Vocabulary and contraction flags get discounted.
- **Neurodivergent writers** — May prefer systematic structure and consistent formatting. Uniform paragraph length alone isn't treated as a flag without supporting signals.
- **Naturally formal writers** — Consistently high formality is less suspicious than inconsistent formality. Legal, medical, and academic professionals often carry their professional voice everywhere.
- **Domain / technical writing** — Documentation and READMEs use headers, bullets, and bold-label-colon items natively, so the formatting fingerprints get discounted; necessary term reuse isn't read as semantic repetition.
- **Translated text** — Translation artifacts (literal phrasing, flat rhythm) can mimic AI, so predictability and burstiness flags get discounted.
- **Constrained assignments (the perfection paradox)** — Students told "don't use first person" or "write formally" produce more AI-like text, so structural flags matching the constraint get discounted. But an assignment explains formality, not *perfection* — real student work is formal **and** still has a human fingerprint.

All adjustments are noted in the verdict.

## What It Catches

- AI vocabulary across three tiers (27 dead giveaways, 41 strong signals, 28 mild signals)
- 14 structural patterns (from superficial -ing phrases to perfect document symmetry and the student-context perfection penalty)
- Punctuation and formatting fingerprints (em-dash density, curly quotes, inline arrows, bold-label-colon lists)
- Humanized-AI residue (even coverage, inserted casual markers, surviving structure, find-replace contractions)
- Semantic repetition (same idea restated across multiple paragraphs)
- Low information density (lots of words, barely any actual content)
- ChatGPT citation fingerprints (utm_source tags, bracketed markdown citations)
- Model-specific writing patterns for ChatGPT, Claude, and Gemini
- Voice inconsistency from human+AI mashups
- Uniform statistical properties (sentence length, paragraph length, opening word diversity)

## What It Can't Do

- **True perplexity scoring** — Would need token-level log probabilities from the model API, which aren't exposed. The predictability estimation is a proxy, not the real thing.
- **Trained classifier** — Tools like Turnitin are fine-tuned on millions of labeled human vs AI text samples. This skill uses heuristics and Claude's own judgment instead.
- **Database comparison** — Can't compare against a corpus of previously submitted work like Turnitin does for plagiarism.
- **Non-English text** — Calibrated for English only. Accuracy drops for other languages.
- **Short text** — Under 100 words, detection is unreliable and the skill says so.
- **Prove authorship** — It detects patterns associated with AI; it cannot prove who or what wrote something.

## Installation

Download the `ai-detector.skill` file and install it in Claude. The skill triggers automatically when you ask Claude to check if text is AI-generated, run AI detection, or similar phrases.

## Usage

Paste any text and ask:

- "Is this AI?"
- "Run AI detection on this"
- "Check if this passes AI detection"
- "AI score this"
- "Was this run through a humanizer?"
- "Quick check — is this AI?"

The skill handles the rest.

## License

MIT
