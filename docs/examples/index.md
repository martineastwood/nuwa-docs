# Examples

Start with **nuwa-example**. Featuristic’s Nim port is still on a branch and is not a released Nuwa demo.

## Available examples

- **[Nuwa Example](nuwa-example/)** — template-style project: `nuwa develop`, tests, and a publish workflow. This is the one to clone.
- **[Featuristic (Nim port)](featuristic/)** — larger genetic-programming library. Source is the [`nim` branch](https://github.com/martineastwood/featuristic/tree/nim) only. It is not on PyPI; `pip install featuristic` is the existing Python package.

## Getting started

1. Clone [nuwa-example](https://github.com/martineastwood/nuwa-example)
2. Install a Nim compiler and `pip install nuwa-build`
3. Run `nuwa develop` (installs `nimble-deps`)
4. Run `python example.py` or `pytest`

## Contributing examples

Projects that are useful as examples:

- Can be built with current nuwa-build from the default branch or a tagged release
- Include tests and a short README
- Do not require an unreleased product branch unless that is stated clearly
