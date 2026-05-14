# AI Detector v4

Advanced AI writing detection using LLM-native pattern analysis and student-context awareness.

## What's New in v4

### Student Assignment Context
The skill now understands that **real student formal essays maintain authentic voice markers**:
- Contractions appear naturally even in formal work (it's, can't, doesn't)
- Voice quirks persist (asides, trailing thoughts, personality)
- Perfection = suspicion in student context

### Five Key Improvements

1. **Perfection Bias Detection** — Flags essays that are TOO polished for their claimed context. In student work, formal + perfect + voiceless = suspicious.

2. **Contraction Consistency for Student Work** — Zero contractions in a claimed student essay is now explicitly flagged as suspicious. Students contract naturally.

3. **Trailing Thoughts Signal** — Incomplete sentences ("So...", "Which is...") are now recognized as strong human indicators, especially in student work.

4. **Enhanced Student Assignment False Positive Handling** — Explicit guidance: formal assignment constraints explain formality, NOT perfection. Real student work = formal + authentic.

5. **Reweighted HS English Teacher Persona** — For student work, the teacher persona carries 40% weight (doubled). Teachers catch impersonation best.

## How It Works

Six analysis layers combined:
- **Sliding Window Analysis (25%)** — Paragraph-by-paragraph predictability
- **Voice Consistency (15%)** — Formality, contractions, vocabulary drift
- **Predictability Estimation (20%)** — Token-level and sentence-level patterns
- **Pattern Detection (10%)** — AI vocabulary, structural flags, semantic repetition
- **Holistic Assessment (10%)** — Does this have genuine thought and angle?
- **Persona Panel (20%)** — Four expert perspectives (Professor, HS Teacher, Hiring Manager, Journalist)

## Output

Full detection report with:
- Final score (0-100%)
- Layer-by-layer breakdown
- Paragraph-by-paragraph analysis
- Voice consistency audit
- Top flags with exact quotes
- Verdict with confidence calibration

## Use Cases

✓ Check if student essays are authentic  
✓ Detect AI-assisted writing in assignments  
✓ Audit submitted work before grading  
✓ Verify writing samples are genuine  
✓ Identify mixed human/AI content  

## Accuracy Notes

- **0-30%**: Likely human
- **31-50%**: Mixed signals
- **51-70%**: Likely AI-assisted
- **71-100%**: Strong/overwhelming AI indicators

**Important**: The score reflects pattern-matching strength, not proof of authorship. Context matters.

## Key Insight for Educators

**The perfection trap is real.** Students told to "write formally" don't become robots. They maintain contractions, personality quirks, and authentic voice. If an essay is formal + perfect + voiceless + zero contractions = probable impersonation. The v4 skill now catches this.

---

**v4 tested on**: Authentic Grade 10 student work, overly-polished student impersonations, clear AI-generated essays, and mixed-voice student writing. All test cases validated correctly.
