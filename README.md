# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

This project classifies posts from the VWVortex MK6 Golf/GTI subforum (public, readable without a login). TakeMeter sorts each post by what it is doing: asking for help (`help_request`), reporting what happened or how the author feels about their car (`experience`), giving instructions for others to follow (`how_to`), or advertising a product the author is selling (`for_sale`).

**Stretch feature:** a fourth label, `for_sale`, because the subforum has a classifieds section and vendor promo posts that none of the other three labels fit.

---

## Label Taxonomy

### `help_request`

**Definition:**
The author asks the community for an answer or diagnosis they don't have, whether it's a single fact or the cause of a problem described through symptoms.

**Example 1:**
> Hi Guys Mk6 TDi [VIN removed]
> would someone be so kind as to confirm the coolant capacity.
> [...]
> My haynes manual gives a capacity of 5.6 Lt but that's for a 1.4 engine
> Kindest Regards Mike

**Example 2:**
> I am experiencing low voltage on My HPFP on my 2010 MK6 CBFA, I tested voltage and I am at 2.8 volts on the connector [...] I have tried swapping fuse 23, j757 and putting on a new connector with no luck, any suggestions on what to check next would be greatly appreciated.

### `experience`

**Definition:**
The author shares how they felt about, or what happened with, their vehicle or something they installed, changed, or bought for it, such as being satisfied, regretting it, or finding it easy or annoying, without asking for help or giving instructions.

**Example 1:**
> Just did the climatronic retrofit on my MK6 GTI. Everything works except Dual zone. Very easy to do, shouldn't take more than a few hours.

**Example 2:**
> Shifting into third i was blown away with the improvement. [...] I just wish that the gti came like this from the factory. Ill report back with extended driving impressions.

### `how_to`

**Definition:**
The author gives instructions a reader could follow to install, replace, adjust, or fix something, whether that's a full step list or a single specific adjustment.

**Example 1:**
> Needed to complete the swap:
> -OEM xenon housings
> -Ballasts
> -Bulbs
> [...]
> Start with removing the grille, (4) screws at the top and gently remove each of the tabs at the bottom of the grille.

**Example 2:**
> The only adjustment I could find was a white hex adjuster on the topside of each light. [...] Turning this gold screw counter clockwise will raise the beam higher.

### `for_sale` (stretch label)

**Definition:**
The author is advertising a product they are selling.

**Example 1:**
> FOR SALE -INJEN Air intake off my 2012 Golf 2.5L. I have the complete Injen intake off my 2012 Golf 2.5l installed in 2015. Original Box, instructions, filter/water cover and all the clamps/bolts needed. ... $150.00 I can ship ground for an additional $10.00.

**Example 2:**
> Garage Clean-out Free Parts in MA. All from a 2012 Golf TDI 4Dr; cleaning out my garage. Local pickup only, absolutely no shipping! ... Brand new Curt tow hitch with ball mount and ball ...

### The hardest boundary

**Which two labels:** `experience` and `how_to`

**The decision rule I used every time (checked in this order):**
0. If the author is advertising a product they are selling, it's `for_sale`. A post by someone trying to buy a part is not `for_sale`; it's `help_request`.
1. Otherwise, if the post's main purpose is to get an answer or opinion from others, it's `help_request`, even if it also describes what the author did.
2. Otherwise, if it contains instructions a reader could follow, it's `how_to`, even if the author also says how they felt.
3. Otherwise, if it mostly shares what happened or how the author felt about the product or job, it's `experience`.

---

## The Dataset

**Where the posts came from:** Public threads in the VWVortex MK6 Golf/GTI subforum, copied by hand. This includes the classifieds/for-sale threads and vendor promotion posts. I limited the data to MK6 cars; a Tiguan post that slipped in was removed.

**Departure from "keep whole posts":** The rows in `labels.csv` are verbatim excerpts, not whole posts: the title plus the key sentences, with long quoted replies, signatures and image captions cut. The model reads only the first 128 tokens (about 90 words) of each post anyway, so full posts would have been cut off at training time regardless. No wording was rewritten; sentences were only left out.

**How many I got and why:** 152 rows, under the 200 target. I used the project's stop rule: I stopped collecting and labeled what I had rather than rush 50 more.

**How I labeled them:** 21 rows were labeled cold (no AI help, just my definitions) and are marked `cold` in the note column. Other rows were labeled with AI help; see "Pre-labelling disclosure" under How I Used AI.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `help_request` | 91 | 59.9% |
| `experience` | 26 | 17.1% |
| `for_sale` | 20 | 13.2% |
| `how_to` | 15 | 9.9% |
| **Total** | 152 | 100% |

The largest label is under the 70% cap, but `help_request` is still about 60% of the data, so a model that guessed it every time would score about 60%.

**Three hard cases**

**1.**
> *The post:* "Osram lamp module. Hi, I'm from Poland. The module in my Golf 6 with Osram headlights has burned out. I can't find it in Poland. Does anyone have one for sale or know where to buy one?"
>
> *Could have been:* `for_sale` or `help_request`
>
> *I chose `help_request`, because:* the words "for sale" appear, but the author is the buyer asking where to find a part. Nobody is advertising a product. This is why rule 0 says "selling", not "mentions sales". A second post, "Roof Rack Pieces" (looking for clamping parts, "Is anyone selling these pieces?"), got the same call.

**2.**
> *The post:* "Sabotage?. I've had this shop, look at my MK6 GTI twice now, they did my water pump and valve cover gasket... fuel trims are +40% right now at idle... Do I just sell the car?"
>
> *Could have been:* `experience` or `help_request`
>
> *I chose `help_request`, because:* most of the post tells a story about what happened with the shop, but it ends with direct questions about what to do next. Rule 1 says the main purpose decides, and the purpose is to get advice.

**3.**
> *The post:* "Just did the climatronic retrofit on my MK6 GTI. Everything works except Dual zone. Very easy to do, shouldn't take more than a few hours. ... If anyone has questions feel free to ask"
>
> *Could have been:* `experience` or `how_to`
>
> *I chose `experience` (labeled cold), because:* the author says it was easy and reports what worked, but never gives steps a reader could follow. It's how the job went, not how to do it. This is the `experience` / `how_to` boundary my decision rule is built around.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, seed 42.

**Hardware:** CPU only (PyTorch 2.14.1+cpu), Python 3.13.7 on Windows, 15.6 GB RAM. Training took under a minute.

**Anything I changed from the defaults, and why:** Nothing.

**Split sizes:** 106 train / 23 validation / 23 test (stratified 70/15/15).

| Label | Train | Val | Test |
|---|---|---|---|
| `help_request` | 63 | 14 | 14 |
| `experience` | 18 | 4 | 4 |
| `how_to` | 11 | 2 | 2 |
| `for_sale` | 14 | 3 | 3 |

Three labels have fewer than 8 examples in the test split (`experience` 4, `how_to` 2, `for_sale` 3), so their metrics will jump around between seeds.

**What the run produced:** `results.json` and `test_split.csv` are committed. On the 23 test posts, accuracy was 0.609 and macro F1 was 0.189. Per-label F1 was 0.757 for `help_request` and 0.000 for the other three, which looks like the model predicting `help_request` for almost every post. I'm saving the diagnosis for next unit.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I pasted forum posts and attachments and asked Claude to turn each into a `labels.csv` row: the title plus the key sentences, verbatim, with a one-line note.
- *What came back:* Rows with proposed labels and notes, and flags on posts that sat on a boundary.
- *What I changed:* I corrected it where I disagreed. I told Claude to remove a non-MK6 (Tiguan) post, and for several batches I supplied the label myself (`help_request`, `how_to`, `experience`) instead of using its guess.

**Moment 2**

- *What I asked for:* Help when the notebook failed to load `labels.csv` (`EmptyDataError`) even though the file loaded fine from the terminal.
- *What came back:* A diagnostic cell to print the working directory and file size, and a suggestion to point the notebook at the file's full path.
- *What I changed:* I ran the diagnostic and set `CSV` to the full path. That loaded all 152 rows and the run went through.

**Pre-labelling disclosure:**
- 21 rows were labeled cold by me, with no AI help, and are marked `cold` in the note column (experience 10, how_to 6, for_sale 3, help_request 2).
- Claude built every row's text as an excerpt from posts I supplied, and proposed a label for the rows where I did not state one. 
- For the rest, I gave the label for each batch myself.
- I read through the file but did not independently re-derive every AI-proposed label, so some of those rows may carry its judgment calls.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**