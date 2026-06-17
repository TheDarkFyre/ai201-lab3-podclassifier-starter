# Evaluation Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 3.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `compute_accuracy()` and
`compute_per_class_accuracy()` in `evaluate.py`.

---

## Background: What is evaluation?

After building a classifier, we need to know how well it works. Evaluation answers:
- **Overall:** What fraction of episodes did we classify correctly?
- **Per-class:** Are we better at some labels than others?

Both functions take the same inputs: a list of predicted labels and a list of
ground-truth labels, in the same order.

---

## compute_accuracy(predictions, ground_truth)

### What it does
Returns the fraction of predictions that exactly match the ground truth.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`, one per episode. |
| `ground_truth` | `list[str]` | The correct labels, in the same order as `predictions`. |

### Output

| Return value | Type | Description |
|---|---|---|
| accuracy | `float` | A value between 0.0 and 1.0. |

---

### Spec fields — fill these in before writing code

**Formula:**

```
accuracy = (number of positions where predictions[i] == ground_truth[i]) / len(ground_truth)

"Correct" means an exact string match at the same index. Divide by the
total number of episodes (length of either list — they must be the same
length).
```

---

**Step-by-step logic:**

```
1. If both lists are empty, return 0.0 (see edge case below).
2. Count the number of indices i where predictions[i] == ground_truth[i].
3. Divide that count by len(ground_truth) and return the float.
```

---

**Edge case — what if both lists are empty?**

```
Return 0.0. There are no episodes to score, so accuracy is undefined;
0.0 is a safe, unambiguous sentinel that won't mislead downstream display
or comparison logic. Dividing 0 / 0 would raise a ZeroDivisionError, so
guard explicitly.
```

---

**Worked example:**

```
predictions  = ["interview", "solo", "panel", "interview"]
ground_truth = ["interview", "solo", "solo",  "narrative"]

index 0: "interview" == "interview" → correct
index 1: "solo"      == "solo"      → correct
index 2: "panel"     == "solo"      → wrong
index 3: "interview" == "narrative" → wrong

correct = 2, total = 4
compute_accuracy() returns 2 / 4 = 0.5
```

---

## compute_per_class_accuracy(predictions, ground_truth)

### What it does
Returns accuracy broken down by each label. For each label in `VALID_LABELS`,
reports how many episodes with that ground-truth label were classified correctly.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`. |
| `ground_truth` | `list[str]` | Correct labels, in the same order. |

### Output

A `dict` keyed by label. Each value is a dict with three keys:

```python
{
    "interview": {"correct": int, "total": int, "accuracy": float},
    "solo":      {"correct": int, "total": int, "accuracy": float},
    "panel":     {"correct": int, "total": int, "accuracy": float},
    "narrative": {"correct": int, "total": int, "accuracy": float},
}
```

---

### Spec fields — fill these in before writing code

**What does "correct" mean for a given class?**

```
An episode counts as correct for class C when ground_truth[i] == C AND
predictions[i] == C. In other words: the episode truly belongs to C, and
the classifier also said C. Episodes where ground_truth[i] != C are
ignored entirely when computing C's stats.
```

---

**What does "total" mean for a given class?**

```
"total" is the number of episodes whose ground-truth label is C — not the
total number of predictions overall. It is the denominator for that class's
accuracy: how many episodes the classifier had the chance to get right for C.
```

---

**Step-by-step logic:**

```
1. Initialize a counts dict: for each label in VALID_LABELS, set
   correct=0 and total=0.
2. Loop over zip(predictions, ground_truth) to get each (predicted, truth) pair.
3. For each pair: increment counts[truth]["total"] by 1.
   If predicted == truth, also increment counts[truth]["correct"] by 1.
4. After the loop, compute accuracy for each label:
   accuracy = correct / total if total > 0 else 0.0
5. Return the dict with all four labels filled in.
```

---

**Edge case — what if a class has no examples in ground_truth (total == 0)?**

```
Set accuracy to 0.0. Division by zero is undefined, and 0.0 is the
value the docstring specifies for this case. It signals "no data" without
crashing — the bar chart in format_evaluation_report() will simply render
as empty for that class.
```

---

**Worked example:**

```
predictions  = ["interview", "interview", "solo", "panel", "panel"]
ground_truth = ["interview", "solo",      "solo", "panel", "narrative"]

idx 0: truth=interview, pred=interview → interview correct+1, total+1
idx 1: truth=solo,      pred=interview → solo      correct+0, total+1
idx 2: truth=solo,      pred=solo      → solo      correct+1, total+1
idx 3: truth=panel,     pred=panel     → panel     correct+1, total+1
idx 4: truth=narrative, pred=panel     → narrative correct+0, total+1

label       correct  total  accuracy
----------  -------  -----  --------
interview       1      1      1.0
solo            1      2      0.5
panel           1      1      1.0
narrative       0      1      0.0
```

---

## Reflection questions (discuss at the checkpoint)

1. Your overall accuracy might be decent even if one class has very low accuracy.
   Why is per-class accuracy a more informative metric than overall accuracy alone?

2. If `panel` episodes consistently get misclassified as `interview`, what does
   that tell you about your training labels or your prompt?

3. You labeled 20 training episodes and evaluated on 20 test episodes (5 per class).
   How might the evaluation results change if you had labeled 100 training episodes?
   What if you had 200 test episodes?
