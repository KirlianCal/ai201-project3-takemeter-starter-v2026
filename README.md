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

**Moment 3**

- *What I asked for:* Help reading my confusion matrices and per-label results, and picking the one change to make for the improvement run.
- *What came back:* An explanation that my first model was predicting only `help_request` because it had too few training steps, and a suggestion to raise epochs from 3 to 10.
- *What I changed:* I made only that one change, re-ran all three seeds, and wrote the verdicts myself from `results_three_seeds_after.json`.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both models were scored on the same 23 held-out posts (`test_split.csv`, seed 42). The baseline is `facebook/bart-large-mnli`, zero-shot, using my definitions from `label_definitions.txt`.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.565 | 0.609 | +0.043 |
| Macro F1 | 0.482 | 0.189 | -0.293 |
| F1 — `experience` | 0.000 | 0.000 | +0.000 |
| F1 — `for_sale` | 0.400 | 0.000 | -0.400 |
| F1 — `help_request` | 0.727 | 0.757 | +0.029 |
| F1 — `how_to` | 0.800 | 0.000 | -0.800 |

**What I predicted before I looked:** Before running the baseline, I predicted that my trained model would come very close to the baseline but not beat it. I made this prediction after seeing my own trained results (accuracy 0.609, macro F1 0.189, mostly guessing `help_request`), but before seeing any baseline numbers.

<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:** On accuracy the two are nearly tied: the trained model is 14/23 and the baseline is 13/23, a difference of one post, which on 23 posts is within noise. My prediction was right that they'd be close, but the trained model edged ahead by one post. On macro F1 the baseline is far ahead (0.482 vs 0.189). The trained model's accuracy comes almost entirely from `help_request`, which is about 60% of my data, and it scored 0.000 on the other three labels. The baseline did the opposite. It got `how_to` right (F1 0.800, on only 2 posts) but over-predicted `for_sale`: it called 12 posts `for_sale` and only 3 were, including all four `experience` posts and several plain questions. So for my data, fine-tuning on 106 examples added almost nothing over a general model reading my definitions, and it lost on the labels that matter least in volume. The test split is also too small (`experience` 4, `how_to` 2, `for_sale` 3) for any of these per-label numbers to be reliable.
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

---

## Run Log — Before

Three seeds (42, 7, 2024), CPU, PyTorch 2.14.1+cpu, base model `distilbert-base-uncased`, 3 epochs, learning rate 2e-5, batch size 16, 128 tokens. Results saved in `results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 on every seed | 0.609 | 0.609 | 0.609 | MISSED |
| 2. Per-label F1 | ≥ 0.55 for all four labels, every seed | lowest 0.000 | lowest 0.000 | lowest 0.000 | MISSED |
| 3. Balance | no label > 65% of the dataset | 59.9% (`help_request`) | 59.9% | 59.9% | MET |
| 4. Seed consistency | accuracy spread ≤ 0.10 | 0.609 | 0.609 | 0.609 (spread 0.000) | MET (but see Verdicts) |
| 5. Confidence | ≥ 80% correct among predictions with confidence ≥ 0.80, at least 5 such predictions per seed | 0 such predictions (max confidence 0.584) | 0 (max 0.469) | 0 (max 0.509) | MISSED |

Per-label F1 on each of the three seeds was identical: `help_request` 0.757, `experience` 0.000, `how_to` 0.000, `for_sale` 0.000. Macro F1 was 0.189 on every seed.

### Confusion matrix

Seed 42 test split (23 posts). Rows are true labels, columns are the model's guesses.

| true \ predicted | help_request | experience | how_to | for_sale |
|---|---|---|---|---|
| **help_request** | 14 | 0 | 0 | 0 |
| **experience** | 4 | 0 | 0 | 0 |
| **how_to** | 2 | 0 | 0 | 0 |
| **for_sale** | 3 | 0 | 0 | 0 |

**My biggest off-diagonal number, and what it means:** The largest single cell is 4 real `experience` posts called `help_request`, with 0 going the other way. But that's partly an accident of counts: every post that isn't `help_request` (4 `experience`, 2 `how_to`, 3 `for_sale`, 9 in all) was called `help_request`. The model never predicted any other label, on any of the three seeds. So the boundary it failed to learn is not one pair but all of them: it learned the majority label (60% of the data) and nothing else. Training loss was still falling at epoch 3 (about 1.28 to 1.09), which suggests it is undertrained as well as short of data.

---

## Verdicts and Diagnoses

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.70 on all three seeds | MISSED | 0.609 on seeds 42, 7 and 2024. Below 0.70 on every seed. |
| 2 | Per-label F1 | ≥ 0.55 for all four labels on all three seeds | MISSED | `help_request` 0.757, but `experience`, `how_to` and `for_sale` were 0.000 on every seed. |
| 3 | Balance | No label above 65% of the dataset | MET | Largest label is `help_request` at 91/152 = 59.9%. This is a property of the data file, so it is the same on every seed. |
| 4 | Seed consistency | Accuracy spread ≤ 0.10 across seeds | MET (vacuously) | Spread was 0.000, so the target is met as written. But it is met only because the model gave the identical answer, "always `help_request`", on every seed. It is not evidence that the model is stable in any useful sense. |
| 5 | Confidence | ≥ 80% correct among predictions with confidence ≥ 0.80, with at least 5 such predictions per seed | MISSED | Zero predictions reached 0.80 on any seed (highest confidence 0.584, 0.469, 0.509). By my own rule, a seed with fewer than 5 such predictions has no verdict and counts as missed. |

**Diagnoses**

**Criteria 1 and 2** The model predicts `help_request` for every post. The confusion matrix shows all 9 non-`help_request` test posts landing in `help_request`, and the result is identical on all three seeds. Accuracy of 0.609 is exactly the share of `help_request` in the test split (14/23). Likely cause is a combination of too few examples and too little training: the minority labels had only 11 to 18 training examples each, and with 106 training rows, batch size 16 and 3 epochs the model made only about 21 updates. Training loss was still falling at epoch 3 (about 1.28 to 1.09), so it had not finished learning. I know this from the matrix, the identical-seed result and the loss curve.

**What my agreement report can and can't tell me.** I matched the staff labels on 30 of 30 posts.  But that check used the staff taxonomy on a different forum, not my own four labels on my own 152 rows, so it does not rule out noisy labels at my `experience`/`how_to` or `help_request`/`experience` boundaries. The matrix can't settle it either, because the model never learned anything beyond the majority label. So my most supported diagnosis is "undertrained on too little data", and a labeling problem remains untested. One more piece of evidence: the zero-shot baseline, which saw none of my posts, got F1 0.800 on `how_to` and 0.400 on `for_sale`, so at least some of my distinctions are learnable from the definitions. With only 2 and 3 test posts on those labels this is weak evidence.

**Criterion 5 (confidence): same cause.** The model never learned to separate the labels, so its probabilities stayed close to the class frequencies. No prediction cleared 0.80, so this criterion could not be tested.

**Criterion 4 (consistency):** met as written, but I am not counting it as a real success. It says only that a model that always guesses the same label is consistent.

---

## Agreement Report

**Agreement rate:** 30 / 30 = 100%

**How I labeled the 30 posts:** I labeled them according to the staff taxonomy in `data/staff_taxonomy.md` (`analysis`, `hot_take`, `reaction`) and saved them as `my_staff_labels.csv` before asking my TF for `staff_labels.csv`. I noticed the posts fell into three blocks of ten, and I also checked each post against the staff rules. Because I saw the pattern, my 100% agreement may partly reflect the order of the set and not only my application of the rules.

**Disagreements**

There were none, so there is nothing to sort into piles (a), (b) or (c).

**What the pattern in my disagreements tells me:** With 30 out of 30 there is no pattern of disagreement to read. That may mean the staff rules are tight enough to apply the same way, or that this set was easier than advertised. This check used the staff taxonomy on a fictional running forum, so it tells me I can apply a written rule I did not write. It says nothing direct about whether I applied my own four labels consistently across my 152 VWVortex rows, and my own `experience`/`how_to` and `help_request`/`experience` boundaries remain untested.


---

## The Improvement

**What I changed:** One training setting: epochs, from 3 to 10. Nothing else changed ( i.e. same labels, same data, same learning rate 2e-5, batch size 16, 128 tokens, same three seeds 42, 7 and 2024).

**Which diagnosis pointed at it:** The diagnosis for criteria 1 and 2 was that the model was undertrained: it made only about 21 updates in 3 epochs, training loss was still falling, and it predicted `help_request` for every post. More epochs targets that problem directly.

### Run Log — After

Three seeds (42, 7, 2024), CPU, PyTorch 2.14.1+cpu, base model `distilbert-base-uncased`, **10 epochs**, learning rate 2e-5, batch size 16, 128 tokens. Results saved in `results_three_seeds_after.json`.

| Criterion           | Target                                                       | Seed 42                  | Seed 7                  | Seed 2024                  | Verdict      |
|---------------------|--------------------------------------------------------------|--------------------------|-------------------------|----------------------------|--------------|
| 1. Overall accuracy | ≥ 0.70 on every seed                                         | 0.652                    | 0.826                   | 0.913                      | MISSED       |
| 2. Per-label F1     | ≥ 0.55 for all four labels, every seed                       | lowest 0.000 (2 labels)  | lowest 0.000 (1 label)  | lowest 0.667 (`experience`)| MISSED       |
| 3. Balance          | no label > 65% of the dataset                                | 59.9%                    | 59.9%                   | 59.9%                      | MET          |
| 4. Seed consistency | accuracy spread ≤ 0.10                                       | 0.652                    | 0.826                   | 0.913 (spread 0.261)       | MISSED       |
| 5. Confidence       | ≥ 80% correct at confidence ≥ 0.80, ≥ 5 predictions per seed | not measured             | not measured            | not measured               | NOT MEASURED |

Per-label F1 after the change:

| Label          | Seed 42 | Seed 7 | Seed 2024 |
|----------------|---------|--------|-----------|
| `help_request` | 0.778   | 0.875  | 0.933     |
| `experience`   | 0.000   | 0.000  | 0.667     |
| `how_to`       | 0.667   | 1.000  | 1.000     |
| `for_sale`     | 0.000   | 1.000  | 1.000     |
| Macro F1       | 0.361   | 0.719  | 0.900     |

Confusion matrix, seed 2024 (the best seed). Rows are true labels, columns are guesses:

| true \ predicted | help_request | experience | how_to | for_sale |
|------------------|--------------|------------|--------|----------|
| **help_request** | 14           | 0          | 0      | 0        |
| **experience**   | 2            | 2          | 0      | 0        |
| **how_to**       | 0            | 0          | 2      | 0        |
| **for_sale**     | 0            | 0          | 0      | 3        |

**Did it help, and how do I know:** Yes, but not enough to meet my own targets. Accuracy rose on all three seeds (0.609 before; 0.652, 0.826 and 0.913 after) and macro F1 rose from 0.189 to 0.361, 0.719 and 0.900. Validation loss at the end of training also fell, from 1.05–1.12 to 0.49–0.71. On seeds 7 and 2024 the model now gets `how_to` and `for_sale` right. Results depends heavily on the seed: seed 42 barely moved off "always `help_request`", and the spread across seeds went from 0.000 (fake stable) to 0.261, well over my own 0.10 target. Criteria 1, 2 and 4 are still missed. All numbers come from a 23-post test split with 2 to 4 posts per minority label, so each per-label number can move by a lot from one post.

---

## What's Still Broken

- **Criterion 1 (accuracy ≥ 0.70 on every seed):** Seed 42 is still 0.652. I would try a higher learning rate or more epochs, and add training examples. Unit allows for one change.
- **Criterion 2 (every label F1 ≥ 0.55):** `experience` is the problem label: F1 0.000, 0.000 and 0.667. I would add more `experience` examples (18 in training) and re-check my labels on that boundary.
- **Criterion 4 (spread ≤ 0.10):** The spread of 0.261 probably says that 106 training posts and a 23-post test split are too small for a stable measure. I would collect more data and use a larger test split.
- **Criterion 5 (confidence):** Not measured for the improvement run, so there is no verdict. I would run the confidence check on the new models.

**The gap between what I meant my labels to capture and what the model learned:** I meant `experience` to capture posts that report what happened or how the author felt about something, importantly without a question or instructions, but the model for the most part treats them as `help_request`: across the three seeds, 10 of the 12 `experience` test posts were called `help_request`, and no `help_request` post was called `experience`. So it has not learned the difference between a post that tells about a story or experince and one that asks for advice, and the matrix cannot say whether that is too few examples (18 in training) or inconsistent labeling at that boundary.



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