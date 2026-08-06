# The Three Ways of Being Wrong

You already met the houses.

| House | Actual (Lakh ₹) | Predicted | Error |
|-------|------------------|-----------|-------|
| A | 40 | 42 | -2 |
| B | 55 | 52 | +3 |
| C | 70 | 74 | -4 |
| D | 90 | 88 | +2 |

You already learned that adding these errors straight up gives you a lie: `-1`, averaged to `-0.25`, as if the model were basically perfect. You already learned the first fix — **take the absolute value, ignore the direction, just measure the distance** — and that fix had a name: **MAE**.

But MAE was never the whole story. It was step one of three. And the other two steps exist for a very specific reason:

> *"Why can't we just average these errors?"*

That question doesn't stop at MAE. It follows you into MSE, and then into RMSE. Each metric is a different answer to the same original problem — **how do you summarize four (or four million) mistakes into one honest number?**

---

## Squaring Instead of Absoluting

MAE said: strip the sign, keep the size.

**MSE (Mean Squared Error)** says something slightly different: *don't just strip the sign — punish the size.*

$$
MSE = \frac{1}{n}\sum_{i=1}^{n}(Actual_i - Predicted_i)^2
$$

Same four houses. Same four errors. But now, instead of taking `|error|`, you square it.

| House | Error | Squared Error |
|-------|-------|----------------|
| A | -2 | 4 |
| B | +3 | 9 |
| C | -4 | 16 |
| D | +2 | 4 |

$$
MSE = \frac{4+9+16+4}{4} = \frac{33}{4} = 8.25
$$

Compare that to MAE's `2.75`. Same houses. Same errors. Wildly different number.

**Why the jump?** Because squaring doesn't treat all mistakes equally anymore. A miss of 4 lakhs doesn't count "twice as much" as a miss of 2 lakhs — it counts **four times as much** (`4² = 16` vs `2² = 4`). Squaring is not a bigger ruler. It's a magnifying glass that grows stronger the bigger the mistake gets.

> *Isn't that unfair to the model?*

Not unfair — **intentional**. Think about what kind of mistake you'd actually want a model to be more afraid of. A model that's consistently off by 2 lakhs everywhere is *tolerable*. A model that's occasionally off by 40 lakhs is *dangerous*. MSE is built to be more scared of the second kind. It doesn't just record big errors — it **amplifies** them so the model, during training, feels real pressure to fix them.

### The Cost You Already Know

If this formula feels familiar, it should. You've already stood inside it — this *is* the cost function from Ordinary Least Squares, the same one gradient descent spends its entire life trying to minimize:

$$
J(\theta) = \frac{1}{n}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2
$$

MSE was never a new idea introduced for "evaluation." It was the loss function wearing a different hat. When you derived OLS by setting the derivative of the squared-error sum to zero, you were minimizing MSE. When gradient descent takes its small careful steps downhill, the "hill" it's descending **is** the MSE surface.

That's not a coincidence — it's the whole reason MSE exists as a *loss function* and not just a *report card*.

---

## The Problem MAE Had — Solved by Accident

Remember the sharp corner?

```
     |
   \ | /
    \|/
-----●-----
```

The absolute value function has no defined slope exactly at zero — two different slopes arrive from the left and right, and gradient descent, which needs a single clean derivative to know which way to step, gets confused standing on that peak.

Squaring erases that corner completely.

$$
f(x) = x^2 \quad \Rightarrow \quad f'(x) = 2x
$$

There's no ambiguity here. At every single point — including zero — the slope is smooth, continuous, and well defined.

```
      \         /
       \       /
        \     /
         \   /
          \ /
           ●
        (smooth)
```

This is *why* MSE became the default loss function for so much of regression. Not because error-squaring is a "better philosophy" of wrongness, but because the resulting curve is a smooth bowl a gradient can walk down without ever hitting a corner it can't read.

---

## The Price MSE Pays: Losing the Units

Here's where MSE quietly breaks something MAE never broke.

Go back to the LPA example. If MAE = 1.5, you could say, in plain human language: *the model is off by 1.5 LPA on average.* Clean. Interpretable. Done.

Now try that with MSE. Say MSE = 8.25, on data measured in **Lakh ₹**.

> *8.25 what?*

Not lakhs. **Lakh-squared.** Because you squared the errors, you also squared the *unit*. And "lakh-squared" isn't a quantity a human being has any intuition for. You can't tell a stakeholder "the model is off by 8.25 square-lakhs" and expect it to mean anything, the same way you can't describe your height in square-meters and expect someone to picture you standing in a doorway.

MSE solved MAE's differentiability problem — and in doing so, created a brand-new problem: **it stopped speaking the language of the original data.**

This is exactly the gap the third metric was built to close.

---

## RMSE: Undoing the Damage

**RMSE (Root Mean Squared Error)** does something almost embarrassingly simple. It takes MSE, and just... square-roots it back.

$$
RMSE = \sqrt{MSE} = \sqrt{\frac{1}{n}\sum_{i=1}^{n}(Actual_i - Predicted_i)^2}
$$

$$
RMSE = \sqrt{8.25} \approx 2.87
$$

That's it. That's the entire trick. You get to keep everything MSE was good at — the smooth differentiable curve, the extra punishment for big mistakes — but the moment you're done training and you want to *report* the number to a human, you take the square root and the unit comes back to Lakh ₹.

$$
\text{MAE} = 2.75 \quad|\quad \text{MSE} = 8.25 \quad|\quad \text{RMSE} \approx 2.87
$$

Notice RMSE and MAE now live in the same neighborhood again — both in lakhs, both interpretable — but RMSE (2.87) sits slightly *above* MAE (2.75). That gap isn't noise. It's RMSE quietly telling on the model: *"There's an outlier in here somewhere, and it's dragging my number up more than MAE would ever admit."*

> *So RMSE is just MAE with better manners?*

Not quite — RMSE is **MSE with a translator**. It inherits MSE's sensitivity to large errors (square first, so big mistakes still get amplified), but it undoes the unit distortion at the very last step (root at the end, so the final number is human-readable again). MAE never amplifies. RMSE amplifies internally, then hides the amplification behind a square root before showing you the result.

---

## Standing All Three Side by Side

```
         MAE                    MSE                    RMSE
   |error| , average      error² , average       √(MSE)

   ✔ same unit as data    ✘ squared unit          ✔ same unit as data
   ✔ robust to outliers   ✘ dominated by outliers  ✘ still sensitive
   ✘ not differentiable   ✔ smooth, differentiable ✔ smooth (via MSE)
     at zero                                          then rooted
```

Three metrics, one lineage. MAE asks the honest question first. MSE trades interpretability for a mathematically usable slope. RMSE trades back — it keeps MSE's sensitivity to big mistakes but restores the readability MAE had from the start.

None of the three is "the correct one." They're three different lenses on the same four wrong predictions, and which lens you pick depends on what you're more afraid of: an uninterpretable number, or an outlier hiding quietly inside your average.

---

## The Final Comparison

Put the three side by side on the exact same four houses, and the numbers alone tell most of the story:

| Metric | Formula | Value | Unit | Outlier Behavior | Differentiable? |
|--------|---------|-------|------|--------------------|------------------|
| **MAE** | $\frac{1}{n}\sum \lvert Actual - Predicted \rvert$ | 2.75 | Lakh ₹ | Robust — grows linearly | ✘ not at zero |
| **MSE** | $\frac{1}{n}\sum (Actual - Predicted)^2$ | 8.25 | Lakh ₹² | Fragile — grows quadratically | ✔ smooth everywhere |
| **RMSE** | $\sqrt{MSE}$ | 2.87 | Lakh ₹ | Still fragile — inherits MSE's amplification | ✔ smooth (via MSE) |

Read that table as three answers to three different jobs:

- **MAE** — the metric you show a stakeholder who wants an honest, unglamorous "how far off, on average."
- **MSE** — the metric you hand to an optimizer, because it needs a slope more than it needs to make sense to a human.
- **RMSE** — the metric you use when you want MSE's discipline but need to say the number out loud in a meeting.

### What Each One Lacks

No metric here is complete on its own — each has a specific blind spot the other two don't fully cover.

**MAE lacks a usable gradient.**
It treats every error with equal weight, which sounds fair, but it means MAE genuinely cannot tell the difference between a model that's consistently a little wrong and a model that's occasionally catastrophically wrong — both can produce the same average. And because the modulus function has no defined slope at zero, it can't be optimized directly by plain gradient descent without extra machinery (subgradients, smoothing tricks). It's honest, but it's not eager to improve itself.

**MSE lacks interpretability, and lacks fairness to outliers.**
The moment you square the error, you square the unit — "lakh-squared" means nothing to anyone reading a report. And the quadratic penalty, which is exactly what makes it optimizer-friendly, is a double-edged sword: one single bad data point (a mislabeled house, a sensor glitch, a genuine anomaly) can dominate the entire metric and quietly convince you the model is worse than it actually is on the data that matters.

**RMSE lacks true independence from MSE's flaw.**
It looks like it fixed everything — same unit as the data, still differentiable — but the square root only undoes the *unit* distortion, not the *sensitivity* distortion. The outlier that inflated MSE already did its damage before the root was ever taken. RMSE reports that damage in a friendlier unit; it doesn't remove it.

**And all three share one blind spot none of them can fix alone:**
None of them tell you if 2.87 lakhs of error is *good* or *bad*. Is that impressive for house prices ranging from 40–90 lakhs? Or is it embarrassing? MAE, MSE, and RMSE are all **absolute** measures — numbers in isolation, with no sense of scale, no baseline to compare against, no answer to *"good compared to what?"* That question — how much of the data's own variance the model actually explains — is exactly the gap the next metric in your notes, R², exists to close.

---

## The Deepest Realization

The real insight here isn't about formulas. It's this:

**Every regression metric is a trade-off between two things that can't both be maximized at once — mathematical convenience for the optimizer, and human interpretability for the reader.**

MAE picks interpretability, pays for it with a broken derivative.
MSE picks the smooth derivative, pays for it with a nonsense unit.
RMSE tries to have both, and mostly succeeds — but never fully escapes MSE's outlier sensitivity, because a square root can't undo *which* errors got amplified before it, only the unit they're expressed in.

There is no metric that is simply "better." There's only a metric that matches what you're willing to trade away.

---

> *"Every model is wrong in some amount. The metric you choose doesn't decide whether it's wrong — it decides which kind of wrongness you're willing to see."*

---

## Putting All Three Side by Side, Properly

Not the quick ASCII glance from earlier — the full comparison, dimension by dimension, using the same four houses throughout.

| Dimension | MAE | MSE | RMSE |
|---|---|---|---|
| **Formula** | $\frac{1}{n}\sum \lvert Actual-Predicted \rvert$ | $\frac{1}{n}\sum (Actual-Predicted)^2$ | $\sqrt{\frac{1}{n}\sum (Actual-Predicted)^2}$ |
| **Value on the house data** | 2.75 | 8.25 | 2.87 |
| **Unit** | Lakh ₹ (same as target) | Lakh ₹² (squared, meaningless to a human) | Lakh ₹ (same as target) |
| **How it treats an error of size *e*** | Grows linearly — a 2× bigger error counts 2× as much | Grows quadratically — a 2× bigger error counts 4× as much | Grows sub-quadratically overall, because the final root softens MSE's amplification — but still more than MAE |
| **Sensitivity to outliers** | Low — one huge miss nudges the average, doesn't dominate it | High — one huge miss can dwarf every other error combined | Medium-high — inherits MSE's amplification, then the root pulls the final number back toward realistic scale |
| **Differentiable at zero?** | No — sharp corner in the graph, breaks clean gradient-based optimization | Yes — smooth parabola everywhere | Yes almost everywhere, but behaves poorly right near $MSE=0$ since the derivative of a square root blows up there |
| **Typically used as...** | An evaluation metric, and a loss function when robustness to outliers matters | The default *loss function* for regression training (this is literally the OLS cost function) | An evaluation metric — reported to humans after training is already done |
| **What it answers** | "On average, how far off is the model, in real-world terms?" | "How steep is the penalty surface I need my optimizer to descend?" | "How far off is the model, in real-world terms, while still respecting large mistakes more than small ones?" |
| **Best read when...** | You want a number a non-technical stakeholder can immediately understand | You're inside the training loop, comparing candidate models, or deriving gradients | You want MAE's interpretability but suspect outliers are hiding in the data |
| **Worst read when...** | Large errors are dangerous and need to be flagged, not averaged away gently | You need to explain the number to someone outside the model | Used alone, without also checking MAE — a high RMSE with a low MAE is itself a diagnostic signal |

### The one-line version of each

- **MAE** — *"How wrong am I, on average, treating every mistake the same?"*
- **MSE** — *"How wrong am I, if big mistakes should hurt disproportionately more than small ones — and I don't care that the unit stops making sense?"*
- **RMSE** — *"How wrong am I, if big mistakes should hurt more, but I still want to explain the final number out loud?"*

### Reading Them Together, Not Apart

The real diagnostic power isn't in any single one of these three — it's in the **gap** between MAE and RMSE.

```
RMSE ≈ MAE   →  errors are roughly uniform, no big outliers
RMSE >> MAE  →  a few large errors are hiding inside the average
```

On the house data: `RMSE (2.87) vs MAE (2.75)` — a small gap, meaning the errors are fairly evenly spread, with no single house wrecking the model's credibility. If house C's error had been `-40` instead of `-4`, MAE would have crept up modestly, but RMSE would have jumped dramatically — and that widening gap is exactly how you'd catch the outlier without even looking at the raw data table again.
---

<img width="1024" height="1536" alt="Regression Metrics" src="https://github.com/user-attachments/assets/e3e70303-7114-40e4-8e7f-a88e4326fdd6" />

---
