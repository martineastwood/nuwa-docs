# Publishing to PyPI

Automatically publish your wheels to PyPI using GitHub Actions with trusted publishing.

## Trusted Publishing (Recommended)

PyPI Trusted Publishing is a secure way to publish without storing API tokens in your GitHub repository.

### Setup

1. **Configure PyPI Trusted Publishing**

   Go to https://pypi.org/manage/account/publishing/ and add a new publisher:

   - **PyPI Project Name**: Your package name
   - **Owner**: Your GitHub username/organization
   - **Repository name**: Your repository name
   - **Workflow name**: `publish.yml` (or your workflow filename)
   - **Environment name**: (optional) Leave blank for all environments

2. **Create Publish Workflow**

   Create `.github/workflows/publish.yml`:

   ```yaml
   name: Publish to PyPI

   on:
     push:
     tags:
       - "v*"

   jobs:
     build_wheels:
       name: Build wheels on ${{ matrix.os }}
       runs-on: ${{ matrix.os }}
       strategy:
         matrix:
           os: [ubuntu-latest, windows-latest, macos-latest]

       steps:
         - uses: actions/checkout@v4

         - name: Build wheels
           uses: martineastwood/nuwa-build-action@v1

         - uses: actions/upload-artifact@v4
           with:
             name: cibw-wheels-${{ matrix.os }}-${{ strategy.job-index }}
             path: ./wheelhouse/*.whl

     publish:
       name: Publish to PyPI
       needs: [build_wheels]
       runs-on: ubuntu-latest
       permissions:
         id-token: write  # Required for trusted publishing

       steps:
         - uses: actions/download-artifact@v4
           with:
             pattern: cibw-wheels-*
             path: dist
             merge-multiple: true

         - name: Publish to PyPI
           uses: pypa/gh-action-pypi-publish@release/v1
   ```

3. **Tag and Push**

   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

   The workflow will automatically build and publish your package.

## Manual Publishing

If you prefer to publish manually or use an API token:

### Using API Token

1. **Create PyPI Token**

   Go to https://pypi.org/manage/account/token/ and create a new API token.

2. **Add to GitHub Secrets**

   - Go to your repository Settings → Secrets and variables → Actions
   - Create a new secret named `PYPI_API_TOKEN`
   - Paste your PyPI API token

3. **Update Workflow**

   ```yaml
   publish:
     name: Publish to PyPI
     needs: [build_wheels]
     runs-on: ubuntu-latest

     steps:
       - uses: actions/download-artifact@v4
         with:
           pattern: cibw-wheels-*
           path: dist
           merge-multiple: true

       - name: Publish to PyPI
         uses: pypa/gh-action-pypi-publish@release/v1
         with:
           password: ${{ secrets.PYPI_API_TOKEN }}
   ```

## TestPyPI Publishing

To test your publishing workflow, use TestPyPI first:

```yaml
publish:
  name: Publish to TestPyPI
  needs: [build_wheels]
  runs-on: ubuntu-latest
  permissions:
    id-token: write

  steps:
    - uses: actions/download-artifact@v4
      with:
        pattern: cibw-wheels-*
        path: dist
        merge-multiple: true

    - name: Publish to TestPyPI
      uses: pypa/gh-action-pypi-publish@release/v1
      with:
        repository-url: https://test.pypi.org/legacy/
```

### Setup TestPyPI Trusted Publishing

1. Go to https://test.pypi.org/manage/account/publishing/
2. Add a new publisher (same as PyPI, but for TestPyPI)
3. Use `repository-url` in your workflow

## Source Distributions

Include source distributions for completeness:

```yaml
build_sdist:
  name: Build source distribution
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.x"

    - name: Install nuwa-build
      run: pip install nuwa-build

    - name: Build sdist
      run: python -m build --sdist

    - uses: actions/upload-artifact@v4
      with:
        name: cibw-sdist
        path: dist/*.tar.gz

publish:
  needs: [build_wheels, build_sdist]
  # ... rest of publish job
```

## Complete Workflow Example

```yaml
name: Build and Publish

on:
  push:
    tags:
      - "v*"

jobs:
  build_wheels:
    name: Build wheels on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v4

      - name: Build wheels
        uses: martineastwood/nuwa-build-action@v1
        with:
          nim-version: "2.0.0"
        env:
          CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
          CIBW_SKIP: "pp* *-musllinux_*"
          CIBW_ARCHS_MACOS: "x86_64 arm64"

      - uses: actions/upload-artifact@v4
        with:
          name: cibw-wheels-${{ matrix.os }}-${{ strategy.job-index }}
          path: ./wheelhouse/*.whl

  build_sdist:
    name: Build source distribution
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Install nuwa-build
        run: pip install nuwa-build

      - name: Build sdist
        run: python -m build --sdist

      - uses: actions/upload-artifact@v4
        with:
          name: cibw-sdist
          path: dist/*.tar.gz

  publish:
    name: Publish to PyPI
    needs: [build_wheels, build_sdist]
    runs-on: ubuntu-latest
    permissions:
      id-token: write

    steps:
      - uses: actions/download-artifact@v4
        with:
          pattern: cibw-*
          path: dist
          merge-multiple: true

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
```

## Release Checklist

Before pushing a release tag:

- [ ] Version updated in `pyproject.toml`
- [ ] Changelog updated
- [ ] Tests passing locally
- [ ] Trusted publishing configured on PyPI
- [ ] Workflow file created and tested
- [ ] Tag created: `git tag v1.0.0`
- [ ] Tag pushed: `git push origin v1.0.0`
