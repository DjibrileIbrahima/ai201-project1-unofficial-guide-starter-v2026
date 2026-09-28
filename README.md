# The Unofficial Guide

Djibrile Ibrahima - advice_threads

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
This project answers questions using the advice_threads corpus, a set of student forum threads where someone asks a question and other students reply with advice they learned firsthand. It handles the practical, unofficial side of campus life: what to wear in winter, which parking lots sell out, how many times you can use pass/fail, or what a commuter locker costs. When you ask a question, the system finds the most relevant replies, writes a short answer from them, and names the thread it came from. If a question isn't covered by the threads, such as sports trivia or medical advice, the system refuses to answer instead of guessing.


## Chunking Strategy

**Chunk size:** No fixed character count. One reply per chunk, with the thread's question line on top (105–254 characters, 175 on average).
**Overlap:** None.

My documents are short. The longest file is only 812 characters. Each file is one question followed by a few replies. Each reply is its own piece of advice, so I made each reply its own chunk. The old chunker cut every 800 characters. Sometimes that split a reply in half. It also made one chunk that was only 2 characters long. A reply on its own can be unclear, so I added the thread question to the top of every chunk. One downside is that the vote counts are lost.

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `thread_bike_commute.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is a bike worth it for a 20 minute walk commute?

Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.
```

**Chunk 2** — source: `thread_first_gen.txt#1` — produced by: `chunker.py::split_documents`

```
THREAD: Anything specific for first-generation students?

The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.
```

**Chunk 3** — source: `thread_laptop_specs.txt#2` — produced by: `chunker.py::split_documents`

```
THREAD: How much laptop do I actually need for CS courses?

I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.
```

**Chunk 4** — source: `thread_parking.txt#1` — produced by: `chunker.py::split_documents`

```
THREAD: Worth getting a parking permit?

Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.
```

**Chunk 5** — source: `thread_sleep_schedule.txt#1` — produced by: `chunker.py::split_documents`

```
THREAD: Everyone says fix your sleep. Does it actually matter?

The library being open until 2am is a trap. It's a resource, not a schedule.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question: Which parking lots sell out in August?**

**Answer:**

```
  (best distance 0.290, cutoff 0.55)

The west lots sell out in about three days in August (thread_parking.txt).

Sources retrieved: thread_internship_timing.txt, thread_parking.txt, thread_winter_advice.txt
```

**My relevance cutoff: 0.55**

I ran my five test questions and the five `OUT_OF_SCOPE` questions through `python app.py retrieve` and recorded the best distance for each. The two groups didn't overlap. In-corpus questions scored 0.29–0.44 and out-of-scope questions scored 0.82–0.90, so any cutoff between 0.44 and 0.82 separates them. I set it at 0.55, inside the gap but closer to the in-corpus side. For a guide like this, refusing a real question costs less than answering from unrelated threads. The out-of-scope questions still pulled back real chunks (for example, a laptop thread for the ibuprofen question), so a loose cutoff would give the model thin, irrelevant material. The risk runs the other way. My winter question already drifted to 0.44 because "what should I wear" is worded differently from the thread's "what do I need?", so a real question worded even more loosely could land above 0.55 and be refused.


<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
| What should I wear in winter? | Yes | 0.4409 |
| Does fixing your sleep matter, yes or no? | Yes | 0.3189 |
| Which parking lots sell out in August? | Yes | 0.2898 |
| How many times can I use the pass/fail option? | Yes | 0.3312 |
| How much is the commuter lounge locker in the student center? | Yes | 0.3240 |
| What is the capital of Mongolia? | No | 0.8990 |
| How do I change the oil in a diesel engine? | No | 0.9047 |
| Who won the 1994 World Cup? | No | 0.8982 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8189 |
| How do I write a for loop in Rust? | No | 0.8606 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1. I asked Claude to write split_documents for my corpus. It split each thread on the reply markers so every reply became its own chunk, with the thread question on top. I checked it with python app.py chunks and went from 26 chunks (one only 2 characters long) to 75 clean ones. I kept its trade-off of dropping vote counts, since my questions don't depend on them.**

**2.  I asked Claude what relevance cutoff to use. It recommended 0.63, the midpoint between my in-corpus (max 0.44) and out-of-scope (min 0.82) distances. I chose 0.55 instead, because I'd rather refuse a real question than answer from unrelated threads.**

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
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Every chunk reads as a complete thought | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Exact facts come through | 6 of 6 | 2 of 2 | 2 of 2 | 2 of 2 | MET |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

Real output from `results/run_2026-09-28_0223_before.md`, produced by `run_eval.py::main` (retrieval by `store.py::search`, chunks from `chunker.py::split_documents`), run 1 unless noted.

**Criteria 1 and 2:** right thread retrieved, source named:

```
Which parking lots sell out in August? — run 1
- Best distance: 0.2898 (passed the gate)
- Sources retrieved: thread_internship_timing.txt, thread_parking.txt, thread_winter_advice.txt

The west lots sell out in about three days in August (thread_parking.txt).
```

**Criterion 3:** produced by `run_eval.py::check_out_of_scope`, cutoff 0.55. Refused 5 of 5:

```
| What is the capital of Mongolia? | 0.899 | refused |
| How do I change the oil in a diesel engine? | 0.905 | refused |
| Who won the 1994 World Cup? | 0.898 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.819 | refused |
| How do I write a for loop in Rust? | 0.861 | refused |
```

**Criterion 4:** from `python app.py chunks -n 5`, produced by `chunker.py::split_documents` (all 5 are in Sample Chunks above):

```
Chunk 4  |  source: thread_parking.txt#1  |  produced by: chunker.py::split_documents
THREAD: Worth getting a parking permit?

Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.
```

**Criterion 5:** exact figures:

```
How much is the commuter lounge locker in the student center? — run 1
The locker in the student centre commuter lounge costs $20 a year (from thread_commuting.txt).

How many times can I use the pass/fail option? — run 1
Based on the documents, you can use the pass/fail option twice per year and eight times across your degree (*thread_pass_fail.txt*).
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | The right thread came back for all 5 questions in all 3 runs, 5 of 5 against a target of 4. |
| 2 | Every answer names a source | MET | All 15 answers named the source file. |
| 3 | Gate stops out-of-corpus questions | MET | The gate refused 5 of 5. The closest was ibuprofen at 0.819, well above my 0.55 cutoff. |
| 4 | Every chunk reads as a complete thought | MET | All 5 chunks from `app.py chunks -n 5` have the thread question and one full reply. |
| 5 | Exact facts come through | MET | "$20" appeared in 3 of 3 runs. Pass/fail run 1 said "twice per year" instead of "two per year". I counted it because the figure is still correct, and this criterion is about wrong numbers, not wording. |

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
