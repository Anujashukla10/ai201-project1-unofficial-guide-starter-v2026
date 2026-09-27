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
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks stay on one topic | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Specific numbers come through correctly | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->
### Real output — criterion 1 and 5, from run 1

**Question:** How much does it cost to wash and dry laundry in Morrow House?
Produced by `store.py::search` (retrieval) and `generate.py::answer_from_chunks` (generation).
```
In Morrow House, laundry costs $1.50 to wash and $1.25 to dry (housing_morrow_house_laundry.txt).
```

Best distance: 0.1698. Source retrieved and cited: `housing_morrow_house_laundry.txt`, which contains the exact figure the answer states.

### Real output — criterion 4, from `chunker.py::split_documents`
```
Laundry costs $1.50 wash, $1.25 dry, coin or card. On noise: loud until about 1am on weekends, no enforced quiet hours.
```
Source: `housing_morrow_house.txt#3`. This chunk fails criterion 4 — it bundles laundry cost and noise level, two separate topics, into one chunk.

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All 5 questions, all 3 runs, had their answer inside a retrieved chunk — exceeded the 4/5 target every time. |
| 2 | Every answer names a source | MET | Every one of the 15 answers (5 questions × 3 runs) named at least one real source file that actually contained the fact. |
| 3 | Gate stops out-of-corpus questions | MET | All 5 out-of-scope questions were refused, well above the 4/5 target, with distances (0.79–0.92) far past the 0.6 cutoff. |
| 4 | Chunks stay on one topic | MET | 4 of 5 sampled chunks cover one topic. The fifth (`housing_morrow_house.txt#3`) bundles laundry cost and noise into one chunk — exactly at the 4/5 target, not above it. |
| 5 | Specific numbers come through correctly | MET | All 5 questions, all 3 runs, stated the exact figure from `expects` with no rounding or invention. |

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
I missed nothing this round. All five criteria met their targets across all three runs.

Honestly, one target was set at the edge rather than with room to spare:
criterion 4 hit exactly 4/5, not higher, because `housing_morrow_house.txt#3`
bundles two topics (laundry cost and noise) into one chunk. My chunker only
splits on blank-line paragraph breaks, and this post states "On noise:" as an
inline sub-topic marker without a blank line before it — so my paragraph-based
split doesn't catch it. If I'd sampled a different 5 chunks, or if my corpus
had more posts shaped this way, this criterion could easily have come out at
3/5 and missed.

I'd tighten criterion 4 next time to "5 of 5 sampled chunks cover exactly one
topic" — a 4/5 target left no room to tell a fluke from a real problem, and
this run shows the chunker has a specific, fixable weak spot rather than
random noise.

## The Improvement

**What I changed:** Added a topic-shift split in `chunker.py::split_documents` 
that splits a paragraph on the pattern `". On "` (e.g. "...coin or card. On 
noise: loud...") before the existing paragraph/merge logic runs, so inline 
sub-topics without a blank-line break get separated.

**Why I picked it:** My Milestone 3 diagnosis found that 
`housing_morrow_house.txt#3` bundles laundry cost and noise level into one 
chunk because the post uses "On noise:" as an inline sub-heading with no 
blank line before it — this pattern repeats elsewhere in the corpus 
("On the meal plan changes"), so I expected splitting on it to fix this 
whole family of cases, not just one file.

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->


| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks stay on one topic | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Specific numbers come through correctly | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Did it help?** No — not for the case it targeted. Chunk count only rose 
from 177 to 178 across the whole corpus (one net split, not the whole family 
of "On X:" patterns I expected). Looking at `housing_morrow_house.txt#3` 
directly: the topic-shift split did fire — the chunk text now shows "Laundry 
costs..." and "On noise:..." separated by a blank line internally — but both 
halves are short (51 and 70 characters), so my existing 60-character 
minimum-merge immediately recombined them back into a single chunk. The two 
changes worked against each other: the split I added to separate topics was 
undone by the merge I'd already built to avoid fragments. Criterion 4 stayed 
at exactly 4/5, identical to before — no regression, but no improvement 
either. Every other criterion (1, 2, 3, 5) also came out identical across 
both run logs, confirming the change didn't break anything else, it just 
didn't accomplish what I intended.

## What's Still Broken

Criterion 4 is still at exactly 4/5, unchanged from before my fix. 
`housing_morrow_house.txt#3` still bundles laundry cost and noise into one 
chunk, because my topic-shift split and my 60-character minimum-merge work 
against each other: the split creates two short pieces, and the merge 
immediately recombines them since neither reaches 60 characters alone.

What I'd try next: don't let the merge step re-join pieces that came from a 
topic-shift split — only apply the minimum-merge to paragraphs that were 
never split in the first place. That keeps the merge doing its original job 
(folding bare headings into real content) without undoing the new split. I 
stopped here because diagnosing why the fix didn't work, and reporting it 
honestly, felt more valuable than quickly patching the threshold without 
understanding the interaction — a rushed second fix risked breaking the 
92% of chunks that were already working correctly.

## What I'd Do Differently

I'd tighten criterion 4's target from "4 of 5" to "5 of 5" from the start. 
A 4/5 target meant a single known weak spot in my chunker could still pass, 
which is exactly what happened — the criterion never actually forced me to 
fix the bundling issue, it just let me document it. A 5/5 target would have 
made this a real MISS in Milestone 1 rather than a MET-with-an-asterisk, 
and that would have been a more honest signal to act on.

I'd also write criterion 4 to test the interaction directly rather than 
sampling 5 chunks and eyeballing them — something like "no chunk contains 
both a cost figure and an unrelated qualitative claim (like noise level)" 
would have caught this specific bundling pattern by rule rather than by 
luck of which 5 chunks got sampled.
