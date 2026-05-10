# AI Detector v3

A Claude skill that analyzes text for signs of AI-generated writing and produces a detailed detection report with scoring.

Unlike basic AI detectors that just scan for word lists, this skill uses Claude's own capabilities as a language model to estimate how predictable the text is, check for voice consistency, and score each paragraph independently — similar to how tools like Turnitin work, but without needing a separate classifier model.

## How It Works

The skill runs text through six analysis layers, each weighted and combined into a final score.

### Layer 1 — Sliding Window (25%)

Every paragraph gets scored independently. For each one, Claude asks itself: "If I were given the first sentence as a prompt, how closely does the rest match what I would generate?" Paragraphs get color-coded (green/yellow/orange/red) so you can see exactly which sections look human and which don't.

This is the closest thing to Turnitin's per-sentence highlighting you can get without a trained classifier.

### Layer 2 — Voice Consistency (15%)

Tracks formality level, contraction usage, vocabulary complexity, and pronoun shifts across the entire document. This is what catches the most common real-world pattern: AI-generated text with a few human paragraphs edited in (or vice versa).

If someone writes six paragraphs of formal, contraction-free prose and then drops a casual opinion paragraph at the end with "I think" and "honestly", this layer catches that mismatch.

### Layer 3 — Predictability Estimation (20%)

The closest approximation to perplexity scoring possible without token-level log probabilities. Claude evaluates each sentence and asks whether it would have written the same thing given the context. Also measures burstiness (sentence length variance), sentence opening diversity, and default phrasing density.

### Layer 4 — Pattern Detection (10%)

Traditional heuristic scanning for known AI writing patterns. Three tiers of AI vocabulary (from dead giveaways like "delve" and "tapestry" down to mild signals like "significant" and "notably"), structural patterns (rule of three, uniform paragraph length, summary conclusions), semantic repetition, information density, and source/citation fingerprints.

Also includes model fingerprinting — identifies whether text likely came from ChatGPT, Claude, or Gemini based on each model's specific habits (citation format, vocabulary preferences, structural tendencies).

### Layer 5 — Holistic Assessment (10%)

A gut-check pass where Claude steps back and evaluates: Does this text have an actual angle? Are there specific details that couldn't be predicted from the topic alone? Could I reconstruct this entire text from just the title? Does it feel like someone had something to say, or like someone was assigned to write about a topic?

### Layer 6 — Persona Panel (20%)

The text gets evaluated by four simulated expert perspectives, each catching different things:

| Persona | Focus |
|---------|-------|
| **Harvard Professor** | Depth of understanding, argument structure, citation quality |
| **High School English Teacher** | Age-appropriate voice, student writing patterns, authenticity |
| **Hiring Manager** | Genuine intent vs template-filling, specificity of claims |
| **Investigative Journalist** | Source quality, factual claims, narrative originality |

Each persona scores independently. When they disagree significantly, it usually means the text is a mashup of human and AI writing.

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

Scores are pattern-match percentages, not probabilities. The skill never claims certainty — it frames results as likelihood based on detected patterns.

## Output

The full report includes:

- Layer-by-layer score breakdown with weights and contributions
- Paragraph-by-paragraph color-coded table showing which sections flagged and why
- Voice consistency analysis (formality range, contraction usage, vocabulary drift, pronoun shifts)
- Persona panel table with each expert's score and reasoning
- Top 3-5 strongest signals with exact quotes from the text
- Source analysis (when citations are present)
- Plain-English verdict with model identification if applicable

A **quick mode** is also available for fast yes/no checks — just the score, the strongest signal, and a model ID if relevant.

## False Positive Handling

The skill includes explicit adjustments for groups that commonly trigger false positives:

- **ESL writers** — May use formal vocabulary from textbook English, avoid contractions, and stick to safe sentence structures. Vocabulary and contraction flags get discounted.
- **Neurodivergent writers** — May prefer systematic structure and consistent formatting. Uniform paragraph length alone isn't treated as a flag without supporting signals.
- **Naturally formal writers** — Consistently high formality is less suspicious than inconsistent formality. Legal, medical, and academic professionals often carry their professional voice everywhere.
- **Constrained assignments** — Students told "don't use first person" or "write formally" will produce text that looks more AI-like. Structural flags matching assignment constraints get discounted.

All adjustments are noted in the verdict.

## What It Catches

- AI vocabulary across three tiers (24 dead giveaways, 30 strong signals, 24 mild signals)
- 13 structural patterns (from superficial -ing phrases to perfect document symmetry)
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

## Installation

Download the `.skill` file and install it in Claude. The skill triggers automatically when you ask Claude to check if text is AI-generated, run AI detection, or similar phrases.

## Usage

Paste any text and ask:

- "Is this AI?"
- "Run AI detection on this"
- "Check if this passes AI detection"
- "AI score this"
- "Quick check — is this AI?"

The skill handles the rest.

## License

MIT
