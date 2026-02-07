# Featuristic

"Because feature engineering should be a science, not an art."

## Overview

Featuristic uses **Genetic Algorithms** to automate feature engineering and feature selection for machine learning. Built with [Nuwa Build](../../nuwa-build/), it demonstrates the power of Nim for high-performance Python extensions.

## How It Works

Featuristic uses **symbolic regression** to intelligently derive interpretable mathematical formulas from your data:

1. **Creates diverse formulas** using operators like `add`, `subtract`, `sin`, `tan`, `square`, `sqrt`
2. **Evaluates importance** by measuring correlation with the target variable
3. **Evolves population** using genetic algorithms (selection, recombination, mutation)
4. **Generates features** that exhibit strong predictive power

Example formula: `(square(feature_1) - abs(feature_2)) * feature_3`

## Features

- **Genetic Feature Synthesis** - Automated feature creation using symbolic regression
- **Genetic Feature Selection** - Identify optimal feature subsets
- **Interpretable Formulas** - Every generated feature includes its mathematical expression
- **Parallel Processing** - Multi-core support via `n_jobs=-1`
- **Early Termination** - Stops when improvements plateau

## Quick Start

```bash
pip install featuristic
```

```python
import featuristic as ft
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split

# Load data
X, y = ft.fetch_cars_dataset()
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)

# Synthesize new features
synth = ft.GeneticFeatureSynthesis(
    n_features=5,
    population_size=200,
    max_generations=100,
    early_termination_iters=25,
)
synth.fit(X_train, y_train)

# Transform data
X_train_new = synth.transform(X_train)
X_test_new = synth.transform(X_test)

# Train model
model = LinearRegression()
model.fit(X_train_new, y_train)
```

## Feature Selection

Use genetic algorithms to find the optimal feature subset:

```python
def objective_function(X, y):
    model = LinearRegression()
    scores = cross_val_score(model, X, y, cv=3, scoring="neg_mean_absolute_error")
    return scores.mean() * -1

selector = ft.GeneticFeatureSelector(
    objective_function,
    population_size=200,
    max_generations=100,
)
selector.fit(X_train_new, y_train)

X_selected = selector.transform(X_train_new)
```

## Inspect Formulas

View the mathematical expressions behind generated features:

```python
info = synth.get_feature_info()
print(info["formula"].iloc[0])
# Output: -(abs((cube(model_year) / horsepower)))
```

## Built with Nuwa

Featuristic is a real-world example of using the Nuwa toolchain:

- **High-performance computation** - Nim's speed for genetic algorithms
- **Easy Python interop** - Seamless integration with scikit-learn
- **Zero-configuration builds** - `nuwa build` handles everything

## Full Documentation

For detailed documentation, API reference, and more examples, visit [featuristic.co.uk](https://www.featuristic.co.uk/).

## License

MIT
