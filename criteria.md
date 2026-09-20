# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:** One of my five questions (PHYS 130 workload) is close in
wording to other course-workload posts in my corpus, so I expect retrieval to
occasionally surface a similar-but-wrong course instead of the right one —
4 of 5 leaves room for that without treating it as a pass on anything.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:** This is 5 of 5, not 4 of 5, because the grounding
instruction in generate.py explicitly tells the model to name the filename
every time, and the chunks it's given always carry a source field — nothing
in my pipeline should let an answer skip this, so anything less than 5 of 5
would mean the instruction itself is being ignored, not just a hard question.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

**Why this target:** When I measured this in Milestone 4, my five in-corpus
questions all landed under 0.38 best distance, and my five out-of-scope
questions all landed over 0.78 — a clean 0.4-wide gap with the 0.6 cutoff
sitting in the middle. Given that gap, I'd actually expect 5 of 5, but I'm
keeping the target at 4 of 5 since I only tested five out-of-scope questions
and don't want to claim more certainty than five data points support.
---

## 4. Chunks stay on one topic

For at least 4 of 5 sampled chunks (`python app.py chunks -n 5`), the chunk
covers exactly one venue, course, or policy — not two unrelated topics
folded into one chunk.



**Why this target:**

My documents are already short (most under 800
characters), so the fallback chunker doesn't split most of them at all —
each post becomes one chunk. That's fine when a post is genuinely one
topic (like North Kitchen), but a few posts pack a heading, a caveat, and
a logistics note together (like the laundry post), and I want to check
that bundling doesn't blur into mixing two different subjects.


---

## 5. Specific numbers come through correctly

For at least 4 of 5 test questions that ask about a number (a cost, a wait
time, an hour range), the answer states the exact same number that appears
in the source chunk — not a rounded, garbled, or invented figure.



**Why this target:**

Almost every document in campus_life carries a
concrete number — $1.50 for laundry, 30 minutes for a Friday wait, 7 hours
a week for PHYS 130 — and a plausible-sounding wrong number (say, "$1"
instead of "$1.50") would be a worse failure than a vague answer, because
it's easy to trust and hard to notice.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
