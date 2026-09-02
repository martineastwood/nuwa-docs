# Featuristic (Nim port, unreleased)

The Nuwa-based rewrite of Featuristic lives on the **`nim` branch** as **2.0.0 (unreleased)**. `pip install featuristic` and [featuristic.co.uk](https://www.featuristic.co.uk/) are still **1.1.0** (pure Python).

Use [nuwa-example](../nuwa-example/) if you want a project you can clone and build today.

## What it is

Featuristic automates feature engineering and feature selection with genetic algorithms and symbolic regression. The `nim` branch compiles the hot path with [Nuwa Build](../../nuwa-build/).

That branch is development-only: APIs can change, and it is not on PyPI.

## Source

```text
https://github.com/martineastwood/featuristic/tree/nim
```

```bash
git clone https://github.com/martineastwood/featuristic.git
cd featuristic
git checkout nim
# Requires Nim on PATH, CPython 3.10+, and nuwa-build 0.5.3+
pip install "nuwa-build>=0.5.3"
nuwa develop
```

Do not treat this as a supported getting-started path for Nuwa.

## Built with Nuwa

When it ships, this port is meant to show:

- compiled Nim for the genetic-algorithm loop
- Python / scikit-learn facing API (`fitness_function` for custom synthesis losses; `objective_function` / `metric` for selection)
- `nuwa develop` / `nuwa build` for the extension
