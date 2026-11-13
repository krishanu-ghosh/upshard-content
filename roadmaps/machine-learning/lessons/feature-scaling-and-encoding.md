---
title: Feature scaling and encoding
description: Normalization versus standardization and why it matters for some models and not others; encoding categorical variables into numbers a model can use.
roadmap: machine-learning
node: feature-scaling-and-encoding
group: working-with-data
video:
  provider: youtube
  id: "sxEqtjLC0aM"
  start: 0
  end: null
creator:
  name: Normalized Nerd
  channel_url: https://www.youtube.com/@NormalizedNerd
duration_seconds: 348
level: intermediate
prerequisites: [feature-engineering]
takeaways:
  - Models that rely on distance or on gradient-based optimisation are sensitive to the scale of their input features, so features measured on very different scales need to be brought onto a comparable footing.
  - Tree-based models split on one feature's threshold at a time and are unaffected by scale, so scaling is a step that matters enormously for some model families and not at all for others.
  - A model only ever sees numbers, so a categorical variable has to be encoded into numeric form — and the encoding chosen can itself introduce an ordering or a relationship that was never actually present in the category.
check_questions:
  - q: Why does an unscaled feature with a much larger range than the others distort a distance-based model?
    a: Distance calculations sum differences across features. A feature ranging from zero to a hundred thousand dominates that sum over one ranging from zero to one, regardless of which feature is actually more informative for the problem.
  - q: What goes wrong if a categorical variable with no natural order is encoded as plain integers?
    a: Assigning the categories 0, 1, 2 implies an ordering and a notion of distance between them — that category 2 is "more" than category 0 — which is meaningless for something like a colour or a city name, and many models will use that spurious ordering as if it were real.
deeper:
  - label: "StatQuest — One-Hot, Label, Target and K-Fold Target Encoding"
    url: https://www.youtube.com/watch?v=589nCGeWG1w
  - label: "Krish Naik — Standardization vs Normalization"
    url: https://www.youtube.com/watch?v=mnKm3YP56PY
related_formulas: []
---

## Why scale is not a cosmetic detail

Take two features measured on wildly different scales: a person's age,
ranging roughly from zero to a hundred, alongside their annual income,
ranging into the hundreds of thousands. Put them in the same model and that
gap can badly distort anything that relies on distance or on gradient-based
optimisation. A distance calculation that sums
differences across features will be dominated almost entirely by whichever
feature happens to have the larger numeric range, regardless of which one
actually carries more useful information about the problem. Bringing features
onto a comparable scale before training is how this distortion is avoided.

**Normalization** rescales a feature into a fixed range, typically zero to
one, by subtracting the minimum and dividing by the range. **Standardization**
instead rescales so the feature has a mean of zero and a standard deviation
of one, which handles outliers somewhat more gracefully since it does not
anchor to the specific minimum and maximum values present in the data.
Neither is universally correct; which to use depends on the specific model
and on whether the data's own distribution is close to normal.

## Why some models need this and others genuinely do not

**Distance-based models** compute distances directly between data points.
K-nearest neighbors, covered in its own
[lesson](/roadmaps/machine-learning/k-nearest-neighbors), is the clearest
example: an unscaled feature with a large range will dominate every distance
calculation regardless of its actual importance. **Gradient-based models**,
including linear and logistic regression and every neural network, optimise
by taking steps proportional to a gradient, and features on wildly different
scales distort that gradient, typically making training slower and less
stable. Both of these model families genuinely need scaling to perform well.

**Tree-based models** are the clear exception: decision trees, random
forests, gradient boosting. A tree splits a dataset on a threshold for one
feature at a time, such as "is this value greater than 50", and that decision
is completely unaffected by whatever scale the feature happens to be measured
on. Scaling a tree-based model's input features does nothing useful at all,
and it is a common beginner habit to scale everything reflexively regardless
of which model is actually being used.

## Encoding: making categories numeric

A model only ever operates on numbers. A categorical variable, like a
colour, a city, or a product type, has to be converted into numeric form
before training, and the method chosen matters a great deal.

**One-hot encoding** creates a separate binary column for each possible
category, with exactly one column set to one for any given row. This avoids
implying any false ordering between categories, at the cost of adding many
columns when a category has many possible values.

**Ordinal or label encoding** assigns each category a single integer. This is
appropriate only when the categories have a genuine natural order, like a
rating of low, medium, high. Used otherwise, it actively misleads, since it
implies a false numeric distance between categories that have no inherent
order at all, such as cities or colours.

**Target encoding** replaces each category with a number derived from the
target variable itself: the average outcome observed for that category in
the training data. This is powerful for categories with very many possible
values, where one-hot encoding would create an unwieldy number of columns,
but it carries a real risk of leaking information from the target into the
features if done without care, which is why it is normally computed using
only training folds during cross-validation rather than the whole dataset at
once.
