# Changelog

## Unreleased

- Enable GitHub Pages deploy on push to `main`.
- Align all pages with the tested matrix: CPython 3.10–3.14 and native Linux x86_64/aarch64, macOS x86_64/arm64, and Windows x86_64 builds; no free-threaded / PyPy / musllinux.
- Document `nuwa-build` 0.5.3 extras (`watch`, `notebook`) and `nuwa_sdk@0.4.4` pins.
- Point the published site at `https://martineastwood.github.io/nuwa-docs/` (GitHub Pages). `getnuwa.com` is not this project.
- Correct Action platform docs, including native Linux ARM64 and preserved caller hooks.
- Document the Action's `package-dir` input.
- Document file-based stub metadata (`nuwaStubDir`) instead of stdout-only.
- Treat Featuristic as unreleased 2.0 on the `nim` branch (not PyPI 1.1.0).
- Document NumPy buffer views (`asNumpyArray`): consumer-only, C vs Fortran, 1D vs ND indexing.
