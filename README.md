# The Unofficial Guide

**Anuja Shukla** — corpus: `campus_life`


> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

This is a question-answering system for the `campus_life` corpus — 88 short
posts about student life, covering things like course workloads, dining hall
wait times, housing logistics, add/drop deadlines, and dorm-specific costs
like laundry. You ask it a plain question and it retrieves the most relevant
chunks, answers using only what's in those chunks, and names the source file.
If you ask something the corpus doesn't cover, it says so instead of guessing.

## Chunking Strategy

**Chunk size:** paragraph-based, not fixed-character — split on blank-line paragraph breaks, with a 60-character minimum merge (any paragraph shorter than that gets folded into the next one, so a bare heading never becomes its own useless fragment).

**Overlap:** none. Paragraphs already break at natural boundaries, so there's no risk of slicing a sentence in half the way a fixed character window can.

**Result:** the fallback chunker's 800-character window never fired — no post in campus_life reaches 800 characters, so it produced 88 documents -> 88 chunks, one per post, average 317 characters (shortest 178, longest 549). My paragraph-based chunker produces 177 chunks, average 156 characters (shortest 61, longest 397).

**Why:** reading the posts in Milestone 1, most are genuinely one topic and belong as one chunk, but a few pack more than one idea into a single post — the Morrow House post covers both laundry cost and noise levels, for instance. Splitting on paragraphs lets those separate ideas become separate, retrievable chunks instead of one chunk that answers two unrelated questions half as well.

**Tradeoff I found:** a few short heading paragraphs (a course name, a building name) end up as their own chunk, and the paragraph after them loses that context on its own. For example, one chunk from `course_cs_340_exams.txt` just says "Start the term project in week three, not week eight" without naming CS 340 anywhere in that chunk — the course name is in the chunk before it. I decided this was an acceptable cost since the source filename is still cited alongside every answer, so the context isn't actually lost to the person reading the answer, even though it's split across chunks.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```


**Chunk 2** — source: `course_stat_150.txt#0` — produced by: `chunker.py::split_documents`

```
STAT 150 Applied Statistics

Transferred in last year, so take this with a grain of salt. Format is flipped: watch the recordings, class time is problem sets. Assessment: three equally weighted midterms, no final. No curve, but the lowest midterm is dropped.
```


**Chunk 3** — source: `housing_morrow_house.txt#3` — produced by: `chunker.py::split_documents`

```
Laundry costs $1.50 wash, $1.25 dry, coin or card. On noise: loud until about 1am on weekends, no enforced quiet hours.
```


**Chunk 4** — source: `course_cs_340_exams.txt#1` — produced by: `chunker.py::split_documents`

```
Start the term project in week three, not week eight; everyone learns this the hard way.
```


This one doesn't name CS 340 on its own — the course name is in the paragraph before it (`#0`). See the tradeoff noted above in Chunking Strategy.


**Chunk 5** — source: `health_center.txt#1` — produced by: `chunker.py::split_documents`

```
Counselling is separate, in the same building, and has its own intake process with a shorter wait than people expect — usually three or four days for a first session.
```

"The same building" refers to the health center named in the previous chunk (`#0`) — same issue as Chunk 4.



## Sample Answer

**Question:** How much does it cost to wash and dry laundry in Morrow House?

**Answer:**

```
In Morrow House, laundry costs $1.50 for a wash and $1.25 for a dry.

Source: housing_morrow_house_laundry.txt
```

Sources retrieved: `housing_calder_annexe.txt`, `housing_fenwick_court.txt`, `housing_innisfree_hall.txt`, `housing_morrow_house_laundry.txt`, `housing_old_brewhouse.txt`


**My relevance cutoff:** 0.6 (the starter default — I measured my own distances and found the default already sits cleanly in the gap, so I kept it rather than changing it for the sake of changing it).

| Question | In corpus? | Best distance |
|---|---|---|
| What happens if I drop a class after week two? | Yes | 0.374 |
| When can I change my meal plan tier, and what happens if I downgrade? | Yes | 0.192 |
| How long is the wait at Verrill Street Grill on Friday evenings? | Yes | 0.147 |
| How much does it cost to wash and dry laundry in Morrow House? | Yes | 0.170 |
| How many hours a week does PHYS 130 typically take? | Yes | 0.204 |
| What is the capital of Mongolia? | No | 0.787 |
| How do I change the oil in a diesel engine? | No | 0.923 |
| Who won the 1994 World Cup? | No | 0.847 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.824 |
| How do I write a for loop in Rust? | No | 0.877 |

My five in-corpus questions all landed under 0.38. All five out-of-scope questions landed over 0.78. That's a gap of about 0.4 with nothing in it, so 0.6 sits comfortably in the middle without me having to tune it — a real finding, since it means this corpus is easy for the embedding model to separate cleanly, not something I got right by luck.



## How I Used AI

**1.** I asked Claude to write a chunking function based on what I noticed
in Milestone 1 — that my posts are short and the 800-character fallback
never splits them. It gave me a plain paragraph splitter, but the first
version I ran produced useless one-line chunks for bare headings like
"STAT 150 Applied Statistics" with nothing under them. I added the
60-character minimum-merge myself so a short heading paragraph gets folded
into the next one instead of becoming its own fragment.

**2.** I asked Claude to check my Milestone 2 acceptance criteria by having
it try to test each one using only the sentence as written. It couldn't
generate a fully unambiguous test for my chunk-quality criterion ("covers
exactly one venue, course, or policy") since judging "one topic" still
requires some human judgment at the edges — I kept the criterion but noted
that limitation myself rather than rewriting it to sound more precise than
it actually is.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
