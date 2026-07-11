# 🧭 The Valley Has More Than One Direction

*You've spent chapters learning to read the slope of a single curve — power rule, product rule, chain rule, quotient rule. One variable, one derivative, one clean number telling you which way "up" is. It felt complete. Then you met a model with two knobs instead of one, and the ground quietly opened up beneath the whole idea.*

---

> Least Squares defines what we want to minimize. Gradient Descent defines how we minimize it.

## The Comfortable World You're Leaving Behind

Every derivative you've taken so far has lived in a tidy, one-dimensional world.

$$f(x) = x^2$$

Differentiate with respect to $x$, get one slope, done. It's a single road, and the derivative is just the incline under your feet.

But then linear regression shows up wearing two parameters at once:

$$\hat{y} = mx + b$$

Suddenly there isn't one unknown. There are two: the **slope** $m$ and the **intercept** $b$. And the question that should be bothering you is the one you actually asked:

> *"How can Gradient Descent optimize both at the same time?"*

That single question is the entire reason partial derivatives exist. So let's earn the answer properly.

---

## The Loss Stopped Being a Function of X

Here's the part that trips almost everyone the first time. The **loss function** — the thing measuring how wrong the model is — used to feel like it depended on the data. It doesn't, not directly.

$$J(m, b)$$

Notice what's missing from that expression: $x$. The loss isn't a function of the input anymore. It's a function of the **parameters** — the very knobs you're trying to tune. Every value of $m$ and every value of $b$ produces one number: how bad the model is at that setting.

Picture that as a landscape instead of a curve:

```
             Loss
               ^
               |
            ____
         __/    \__
      __/          \__
     /                \
  ---------------------------->
             m

     b stretches into the page
```

It's not a line anymore. It's a **bowl**, sitting in three dimensions. Every point on that bowl's surface corresponds to one pair $(m, b)$, and its height is the cost at that pair. Your job is still the same as before — reach the bottom — but now you're standing *on a surface*, not walking *along a line*.

---

## Why "What's the Slope?" Stops Making Sense

Imagine you're standing somewhere on that bowl.

```
              ●   ← you, right now
```

You have two ways to move: east–west (change $m$) or north–south (change $b$). So the old question — *what's the slope here?* — doesn't even parse anymore. Slope in which direction? You have to split it into two separate, more honest questions:

- What's the slope if I only move along $m$, holding $b$ perfectly still?
- What's the slope if I only move along $b$, holding $m$ perfectly still?

Those are **partial derivatives**. The name is almost too literal — each one only tells you *part* of the story, because it's deliberately ignoring one of the directions.

$$\frac{\partial J}{\partial m} \qquad \frac{\partial J}{\partial b}$$

The first asks: *if I nudge the slope while freezing the intercept, how does the loss react?*
The second asks the mirror question about the intercept.

Bundle them together and you get the **gradient**:

$$\nabla J = \left[ \frac{\partial J}{\partial m},\ \frac{\partial J}{\partial b} \right]$$

This is the key shift in identity: the gradient isn't one slope anymore. It's a **vector** — one entry per parameter, pointing in the direction where the loss increases fastest. Gradient descent, as always, walks the *opposite* way.

If linear regression has two parameters, that's two partial derivatives. If a neural network has ten million parameters, that's ten million partial derivatives, computed the exact same way, one per knob. The idea never scales up in difficulty — only in volume, and in how much your laptop fan starts to resent you.

---

## Building the Bowl: Where the Cost Function Actually Comes From

Before going further with the gradient, it's worth slowing down on where $J$ itself comes from — because right now it's still an abstract letter.

Say you're predicting house prices from house size:

| House Size (sq ft) | House Price ($) |
|---|---|
| 800 | 120,000 |
| 1000 | 150,000 |
| 1200 | 180,000 |
| 1500 | 220,000 |
| 1800 | 260,000 |

Your eyes catch the trend instantly — bigger house, bigger price. Linear regression's whole job is to catch that trend *mathematically*, using:

$$Y = a_0 + a_1 X$$

- $a_1$ is the **slope** — how strongly the feature pushes the prediction.
- $a_0$ is the **intercept** — the baseline value when $X = 0$.

There are infinitely many lines you *could* draw through that data. So the model needs a way to score how good any particular line is. For one data point, that score starts as a plain error:

$$e = \hat{y} - y$$

But raw errors betray you. A prediction that's $20$ too high and one that's $20$ too low seem to cancel out to zero — as if the model were perfect. It clearly isn't. So every error gets **squared** first, which does two things at once: it kills the sign problem, and it punishes big mistakes disproportionately. An error of $10$ becomes a penalty of $100$; an error of $2$ becomes a penalty of $4$. A mistake five times larger becomes twenty-five times more expensive. That asymmetry is intentional — it's what keeps the model honest about large blunders.

Average those squared errors across every training example, and you get the **cost function**, most commonly Mean Squared Error:

$$J(m, b) = \frac{1}{2m} \sum_{i=1}^{m} (\hat{y}_i - y_i)^2$$

Two details in that formula deserve their own moment, because they're exactly the kind of thing that looks arbitrary until someone shows you the mechanism.

> *"Why am I dividing by $m$?"*

Because you want the **average** error per example, not a raw total. Without dividing by $m$, a dataset of 10,000 rows would produce a cost value ten times larger than the same model trained on 1,000 rows — not because the model got worse, but purely because there's more data to sum over. Dividing by $m$ keeps the scale of the loss (and its gradient) independent of dataset size, which makes the learning rate far easier to choose sensibly. Nothing breaks if you skip this step — it's a convenience, not a correctness requirement.

> *"The 2 in the denominator is basically to cancel with the 2 that comes after the derivative — but why? When I differentiate, that 2 in the power comes in front of the equation and defines how the function changes, doesn't it?"*

You've actually already spotted the mechanism yourself. Watch what happens when you differentiate a squared term:

$$\frac{d}{dx}(x^2) = 2x$$

That factor of $2$ falls out automatically, every time you differentiate a square. So if the cost function is written with a $\frac{1}{2}$ sitting in front of it:

$$\frac{1}{2}(\hat{y}_i - y_i)^2$$

then differentiating produces a $2$ that cancels the $\frac{1}{2}$ exactly, leaving a clean gradient with no leftover constant cluttering the update rule. Here's the part that matters most: that $2$ **does** define how the squared function changes locally — you're right about that — but multiplying an entire function by a constant only *scales* its gradient uniformly. It doesn't move the location of the minimum even slightly. The valley floor stays exactly where it was; the $\frac{1}{2}$ just tidies up the arithmetic on the way there.

---

## Descending the Bowl, One Coordinated Step at a Time

With the cost function built, the update rule for each parameter looks almost identical to the single-variable case you already know:

$$m := m - \alpha \frac{\partial J}{\partial m} \qquad b := b - \alpha \frac{\partial J}{\partial b}$$

The minus sign is still doing the same job it always did — the gradient points uphill, so you walk the other way. The learning rate $\alpha$ still controls stride length, not direction.

Let's make it concrete. Say the current parameters are $m = 2$, $b = 5$, and at that exact point the partial derivatives come out to:

$$\frac{\partial J}{\partial m} = 8 \qquad \frac{\partial J}{\partial b} = -2$$

With $\alpha = 0.1$:

$$m_{\text{new}} = 2 - 0.1(8) = 1.2$$
$$b_{\text{new}} = 5 - 0.1(-2) = 5.2$$

The slope moved down. The intercept moved up. Same step, opposite directions — because their gradients pointed opposite ways.

> *"So here, each term has its own gradient descent — am I right, or how does it actually work?"*

Close, but the framing matters. It isn't five separate little gradient descents running in isolation. It's **one** gradient descent, computing **many** partial derivatives from the **same** shared position on the bowl, then applying all the updates together.

Here's the distinction that actually matters: both partial derivatives above were computed *at the same point*, $(m, b) = (2, 5)$. You don't update $m$ first and then compute $\partial J/\partial b$ using the *new* value of $m$ — that would mean the two gradients came from two different points on the surface, and the whole thing would lose its footing.

Think of it as driving a strange vehicle with two steering wheels — one for left-right, one for up-down. Every second, the navigation system tells you:

```
Move left by 0.8 units.
Move forward by 0.2 units.
```

You don't take one trip for "left" and a separate trip for "forward." You apply both movements *together*, arriving at one new position. That's exactly what

$$a_j := a_j - \alpha \frac{\partial J}{\partial a_j} \quad \text{for every } j$$

is doing — one simultaneous update, across every parameter, computed from one shared snapshot of the loss surface.

---

## How the Model Actually "Decides" a Weight

Scale the idea up. A house price model rarely stops at one feature:

$$Y = a_0 + a_1X_1 + a_2X_2 + a_3X_3 + a_4X_4$$

Nothing about the mechanism changes — the parameter vector just gets longer. Gradient descent computes one partial derivative per weight:

$$\frac{\partial J}{\partial a_0}, \ \frac{\partial J}{\partial a_1}, \ \dots, \ \frac{\partial J}{\partial a_4}$$

and updates all five together. But this raises the question you asked next, and it's the one that actually reveals how learning happens at all:

> *"I know $a_i$ is the weight, but how does the model decide the value for the weight? And what happens to a weight while gradient descent is busy fitting the other $a_i$'s?"*

The model never "decides" a weight the way a person turns a dial toward a value they already have in mind. It starts every weight at some arbitrary point — often all zeros — and then repeatedly asks the data one question: *if I nudged this particular weight slightly, would the total error go up or down?* That question, answered for every weight, is exactly what the gradient is.

If increasing $a_1$ would shrink the loss, its gradient pushes $a_1$ upward. If decreasing $a_3$ would shrink the loss, its gradient pushes $a_3$ downward. The data, filtered through the loss function, is what supplies the direction — the algorithm never invents one from nowhere.

And the weights are never fitted in isolation, because they aren't independent — they interact through the same shared prediction:

$$\hat{y} = a_0 + a_1X_1 + a_2X_2 + a_3X_3 + a_4X_4$$

Change one weight, and the prediction shifts, which shifts the loss, which shifts the gradients computed for *every other weight* on the next pass. So while gradient descent is "busy" adjusting $a_3$, it isn't ignoring $a_1$ — both are being read off the *same* current state, at the *same* moment, from the *same* surface. If two features like $X_1$ and $X_2$ happen to carry overlapping information, the model doesn't have to lean entirely on one; responsibility can split between them, with one weight rising while another falls, and the total loss still dropping. That's joint optimization — every weight learned in the context of all the others, never one at a time.

---

## The Stride Length: Learning Rate, Revisited in Two Dimensions

The learning rate $\alpha$ hasn't changed jobs — it still decides how far you move once the gradient has told you which way to go.

**Too small**, and progress looks like this:

```
.  .  .  .  .  .
```

Technically correct, agonizingly slow.

**Too large**, and the walk turns into an oscillation across the valley:

```
Minimum
      x
<--------->
```

The steps overshoot the bottom on every pass, and the loss can bounce indefinitely instead of settling — or diverge outright.

**Well-tuned**, and the descent looks the way it's supposed to:

```
      .
    .
  .
 .
x
```

Smooth, shrinking steps, converging on the floor of the bowl. And that shrinking happens somewhat automatically near the minimum — as the surface flattens out, the gradient's magnitude naturally gets smaller, so even a fixed $\alpha$ produces smaller and smaller updates on its own. A small gradient is a hint that you're near flat ground — though not a guarantee that the flat ground you found is the *lowest* flat ground on the whole surface. Bowls in higher dimensions are allowed to be dishonest like that.

---

## Three Ways to Walk: Batch, Stochastic, Mini-Batch

One more layer sits underneath all of this — *how much data* gets used to compute the gradient at each step.

- **Batch Gradient Descent** uses every training example for each update. Stable, smooth, but expensive when the dataset is enormous.
- **Stochastic Gradient Descent (SGD)** uses a single random example per step. Fast and cheap per update, but noisy — the path toward the minimum zigzags rather than gliding.
- **Mini-Batch Gradient Descent** splits the difference, using small batches (32, 64, 128 examples at a time). It's the version most deep learning frameworks reach for by default, trading a little stability for a lot of speed.

None of these change *what* the gradient means. They only change *how much evidence* you consult before taking each step downhill.

---

## 🧠 ML Bridge

This is the exact machinery sitting underneath every model you've trained or will train — including the stroke prediction work you've been building. Every time that model adjusts its weights during training, it isn't doing anything mystical: it's computing a partial derivative for each feature's weight, bundling them into a gradient, and taking one coordinated step downhill. Add more features, and you're not changing the algorithm — you're just handing it a longer gradient vector to compute. Backpropagation in a neural network is this same loop, applied layer by layer, with the chain rule stitching the partial derivatives together across the network. The bowl just grows more dimensions than anyone could ever actually visualize.

---

## 🌀 Deepest Realization

The single-variable derivative you spent chapters mastering was never wrong — it was just a special case. A model with one parameter lives on a curve, and a curve only has one direction to have a slope in. The moment a second parameter enters the picture, "the slope" stops being a well-defined question, and the only honest way forward is to ask it twice — once per direction — then act on both answers *at the same time, from the same point*. That's the whole leap from ordinary derivatives to partial derivatives: not a harder rule, but a more honest question, asked once for every knob the model has.

---

> ✅ **Final Takeaway**
> *A gradient isn't one slope pretending to be several — it's several honest, separate slopes, each asking "what if only I changed?", collected into a single vector and walked all at once. Every parameter learns in the full context of every other parameter, because they were never separate questions to begin with.*

---

> *Next up: what happens when the bowl stops being a bowl — saddle points, plateaus, and why "the gradient is nearly zero" doesn't always mean "you've found the bottom."*
---

<img width="1024" height="1536" alt="Gradient descent" src="https://github.com/user-attachments/assets/6b3a9b3b-33fe-45bd-aea6-62eae863097f" />

---
