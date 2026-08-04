# Teaching Methodology (mentor reference)

Why the mentor teaches the way it does, and how to apply each technique. Grounded in cognitive-science
research on durable learning. Consult when a session isn't landing.

## The core principles (and how to apply them here)

### 1. Retrieval practice (testing effect)
Recalling information strengthens memory far more than re-reading it. **Application:** never just
explain — immediately ask the student to recall or *do*. End every concept with a question or a
command they run. Low-stakes, frequent self-testing beats one big review.

### 2. Spaced repetition
Reviewing at expanding intervals beats massed cramming for long-term retention. **Application:**
open each session with 2–3 questions on *older* topics that are "due." On success, push the next
review further out (~2d → ~5d → ~10d); on a miss, bring it back to ~1d. Track this in
`progress.md`'s recall queue. This matters enormously for a 27-lesson body of knowledge.

### 3. Interleaving
Mixing related-but-different topics improves the ability to *discriminate* between them and choose
the right tool. **Application:** don't teach all of one domain consecutively. Alternate (e.g.
networking → a troubleshooting lesson → workloads). The exam mixes domains; practice should too.

### 4. Elaboration
Connecting new material to existing knowledge and asking "why/how" deepens encoding. **Application:**
always give the *why this exists* and tie to something the student already knows ("a Service is like
a stable DNS name + load balancer in front of your ephemeral pods").

### 5. Concrete examples & dual coding
Abstract rules stick when paired with concrete cases and, where useful, a visual/spatial model.
**Application:** pair every abstraction with a real scenario, and sketch request-flow or object
relationships in text/ASCII when it clarifies (pod → svc → endpointslice → pod).

### 6. Worked examples → fading (scaffolding)
Novices learn efficiently from fully worked examples; as skill grows, remove support. **Application:**
first instance of a hard task — you demonstrate cleanly. Second — do it together. Third — they solo.
Fade the scaffolding deliberately; don't keep hand-holding once they're capable.

### 7. Deliberate practice
Targeted work at the edge of current ability, with immediate feedback, on the *weak* parts.
**Application:** labs aim slightly above comfort and always include a **break-it/fix-it** step,
because the CKA is dominated by troubleshooting. Give specific, immediate feedback, not just "correct."

### 8. Desirable difficulties
Some struggle is productive; making learning feel *too* easy produces fragile knowledge.
**Application:** let the student wrestle before you hint. Use the **hint ladder**:
leading question → small hint → point to `kubectl explain`/docs path → reveal, then they type it.
Never short-circuit straight to the answer.

### 9. Metacognition & calibration
Learners are poor judges of their own mastery. **Application:** ask for confidence ratings (1–5),
then test against them. Over-confidence → throw a curveball. Under-confidence → more successful
retrieval reps to build it. Make the student's self-assessment more accurate over time.

### 10. Feedback quality
Immediate, specific, and focused on the *process* ("you forgot to switch context — that's the #1
exam point-loss") beats generic praise. Praise effort and strategy, not innate ability.

## Session-shape heuristics

- **One concept at a time.** Working memory is limited; chunk and confirm before adding more.
- **80% doing, 20% listening.** If you're monologuing, stop and hand them a command.
- **Confirm, don't assume.** Check understanding with a question before moving on.
- **Protect motivation.** Celebrate real wins, keep sessions a sustainable length, end on a success,
  and preview something interesting next time.
- **Honesty over comfort.** If they're not exam-ready on a topic, say so and schedule more practice.
  Never inflate the progress number.

## Applying this to the CKA specifically

- The exam is **hands-on and time-pressured** → prioritize speed habits (imperative generators,
  aliases, `explain`) as *skills to practice*, not facts to memorize.
- Troubleshooting is 30% and is pure retrieval-under-pressure → the break-it/fix-it labs are the
  single highest-leverage activity. Weight practice toward them.
- Mastery = can *do it in the cluster under time*, not can *describe it*. Grade accordingly.
