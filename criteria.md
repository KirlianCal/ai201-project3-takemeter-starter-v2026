# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy

The trained model achieves an overall accuracy of at least 0.70 on the held-out test set for all three seeds (42, 7, and 2024).

**Why this target:** There are three separate labels, so a uniform random guesser would get ~33% accuracy. A 70% target requires the model to learn patterns in how VWVortex members ask for help, share experiences, and explain repairs, while allowing for informal language and some overlapping post purposes.

---

## 2. Per-label performance

Each of the three labels achieves an F1 score of at least 0.55 on the held-out test set for all three seeds.

**Why this target:** The `experience` and `how_to` labels can overlap when someone describes a modification and explains how they completed it. The `help_request` label can also be difficult to distinguish from posts that describe a problem without directly asking a question. A 0.55 minimum gives every label a meaningful performance floor without assuming that all three will be equally easy to learn.

---

## 3. Balance

No label accounts for more than 65% of the final labelled dataset.

**Why this target:** In my reading of the MK6 subforum, most threads are people asking questions, so I expect `help_request` to be the largest label if I collect without steering. If most examples are `help_request`, the model could learn to favour questions and perform poorly on posts describing modifications or providing instructions. Each label needs enough examples for the model to learn its own patterns rather than rely on the most common class.

---

## 4. Consistency across training seeds

The difference between the highest and lowest overall accuracy across seeds 42, 7, and 2024 must be no greater than 0.10.

**Why this target:** The model is trained on a relatively small, specialised dataset of VWVortex posts, so changing the seed may affect which examples it learns from and how well it generalises. A maximum spread of 10 percentage points is a practical tolerance for this small dataset while still requiring reasonably stable results.

---

## 5. Confidence

Among test-set predictions made with a confidence of at least 0.80, at least 80% must be correct for each seed. Confidence means the model's highest predicted class probability for a post. If a seed has fewer than 5 test predictions at or above 0.80, that seed has no verdict for this criterion and counts as missed.

**Why this target:** Forum posts sometimes mix personal experience with repair instructions, making a confident prediction potentially misleading when a post crosses the `experience`/`how_to` boundary. Requiring at least 80% accuracy among high-confidence predictions would make confidence scores more useful when deciding which classifications can be accepted automatically and which posts need human review.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
