# Code Coverage Report Action

[![SuperLinter](https://github.com/insightsengineering/coverage-action/actions/workflows/lint.yaml/badge.svg)](https://github.com/insightsengineering/coverage-action/actions/workflows/lint.yaml)
[![Test](https://github.com/insightsengineering/coverage-action/actions/workflows/test.yaml/badge.svg)](https://github.com/insightsengineering/coverage-action/actions/workflows/test.yaml)

Action that converts a Cobertura XML report into a markdown report.

## Action Type

Composite

## Author

Inisghts Engineering

## Inputs

* `token`:

  _Description_: Github token to use to publish the check.

  _Required_: `false`

  _Default_: `${{ github.token }}`

* `path`:

  _Description_: Path to the Cobertura coverage XML report.

  _Required_: `false`

  _Default_: `coverage.xml`

* `threshold`:

  _Description_: The minimum allowed coverage percentage, as a real number.

  _Required_: `false`

  _Default_: `0`

* `fail`:

  _Description_: Fail the action when the minimum coverage was not met.

  _Required_: `false`

  _Default_: `True`

* `publish`:

  _Description_: Publish the coverage report as an issue comment.

  _Required_: `false`

  _Default_: `False`

* `diff`:

  _Description_: Create a diff of the coverage report.

  _Required_: `false`

  _Default_: `False`

* `diff-branch`:

  _Description_: Branch to diff against.

  _Required_: `false`

  _Default_: `main`

* `storage-subdirectory`:

  _Description_: Subdirectory in the diff-storage branch where the XML reports will be stored.

  _Required_: `false`

  _Default_: `.`

* `diff-storage`:

  _Description_: Branch where coverage reports are stored for diff purposes.

  _Required_: `false`

  _Default_: `_xml_coverage_reports`

* `coverage-summary-title`:

  _Description_: Title for the code coverage summary in the Pull Request comment.

  _Required_: `false`

  _Default_: `Code Coverage Summary`

* `uncovered-statements-increase-failure`:

  _Description_: Fail the action if any changed file has an increase in uncovered lines compared to the `diff-branch`. This corresponds to pycobertura exit code 2, which indicates that at least one changed file has more uncovered lines than before (Miss > 0). Note that this is different from coverage rate reduction - it specifically checks for increases in the absolute number of uncovered lines.

  _Required_: `false`

  _Default_: `False`

* `new-uncovered-statements-failure`:

  _Description_: Fail the action if new uncovered statements are introduced AND overall coverage improved (total uncovered lines decreased) compared to the `diff-branch`. This corresponds to pycobertura exit code 3, which only occurs when total uncovered lines decreased (Miss <= 0) but there are still new uncovered statements (Missing != []). To fail on ALL new uncovered statements regardless of overall coverage improvement, use this flag together with `uncovered-statements-increase-failure: true`.

  _Required_: `false`

  _Default_: `False`

* `coverage-rate-reduction-failure`:

  _Description_: Fail the action if the overall coverage percentage (rate) decreases compared to the `diff-branch`. This is different from `uncovered-statements-increase-failure` which checks for absolute increases in uncovered lines. This flag specifically looks at the coverage percentage and fails if it goes down, regardless of whether uncovered lines increased or decreased. This is a more forgiving approach that focuses on the relative coverage rate rather than absolute uncovered line counts.

  _Required_: `false`

  _Default_: `False`

* `pycobertura-exception-failure`:

  _Description_: Fail the action in case of a `Pycobertura` exception.

  _Required_: `false`

  _Default_: `True`

* `togglable-report`:

  _Description_: Make the code coverage report togglable.

  _Required_: `false`

  _Default_: `False`

* `exclude-detailed-coverage`:

  _Description_: Whether a detailed coverage report should be excluded from the PR comment.
The detailed coverage report contains the following information per file:
number of code statements, number of statements not covered by any test,
coverage percentage, and line numbers not covered by any test.

  _Required_: `false`

  _Default_: `False`

### Outputs

* `summary`:

  _Description_: A summary of coverage report

## How it works

This tool makes use of the [PyCobertura](https://github.com/aconrad/pycobertura) CLI tool to produce the summary outputs. The action also supports `diff`s against a given branch and makes use of a remote branch to store reports, which can be specified via this action.

## Failure Modes

The action provides three different failure modes for detecting coverage regressions, each with different characteristics:

### 1. `uncovered-statements-increase-failure` (Strict)

* **When it fails**: When any changed file has an increase in the absolute number of uncovered lines
* **Pycobertura exit code**: 2
* **Use case**: When you want to ensure that no file gets worse coverage, regardless of overall improvements
* **Example**: If you add 5 uncovered lines to file A but remove 10 uncovered lines from file B, this will still fail because file A got worse

### 2. `new-uncovered-statements-failure` (Moderate)

* **When it fails**: When new uncovered statements are introduced AND overall coverage improved
* **Pycobertura exit code**: 3
* **Use case**: When you want to allow overall improvements but still catch new uncovered code
* **Example**: If you add 5 uncovered lines to file A but remove 10 uncovered lines from file B, this will fail because there are new uncovered statements, even though overall coverage improved

### 3. `coverage-rate-reduction-failure` (Forgiving)

* **When it fails**: When the overall coverage percentage decreases
* **Pycobertura exit code**: N/A (custom implementation)
* **Use case**: When you want to focus on the overall coverage rate rather than absolute line counts
* **Example**: If you add 5 uncovered lines to file A but remove 10 uncovered lines from file B, this will pass if the overall coverage percentage improved

### Combining Failure Modes

You can combine these failure modes for different levels of strictness:

```yaml
# Most strict: Fail on any uncovered line increase
uncovered-statements-increase-failure: true
new-uncovered-statements-failure: true
coverage-rate-reduction-failure: true

# Moderate: Allow overall improvements but catch new uncovered code
uncovered-statements-increase-failure: false
new-uncovered-statements-failure: true
coverage-rate-reduction-failure: true

# Most forgiving: Only fail if overall coverage percentage decreases
uncovered-statements-increase-failure: false
new-uncovered-statements-failure: false
coverage-rate-reduction-failure: true
```

## Usage

Example usage:

```yaml
---
name: Code Coverage

on:
  # NOTE: Both, the 'pull_request' and the 'push'
  # events are REQUIRED to take full advantage
  # of the features of this action.
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  coverage:
    name: Calculate code coverage
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Your logic to generate the Cobertura XML goes here
        run: echo "Your logic to generate the Cobertura XML goes here"

      - name: Produce the coverage report
        uses: insightsengineering/coverage-action@v3
        with:
          # Path to the Cobertura XML report.
          path: ./cobertura.xml
          # Minimum total coverage, if you want to the
          # workflow to enforce it as a standard.
          # This has no effect if the `fail` arg is set to `false`.
          threshold: 80.123
          # Fail the workflow if the minimum code coverage
          # reuqirements are not satisfied.
          fail: true
          # Publish the rendered output as a PR comment
          publish: true
          # Create a coverage diff report.
          diff: true
          # Branch to diff against.
          # Compare the current coverage to the coverage
          # determined on this branch.
          diff-branch: main
          # This is where the coverage reports for the
          # `diff-branch` are stored.
          # Branch is created if it doesn't already exist'.
          diff-storage: _xml_coverage_reports
          # A custom title that can be added to the code
          # coverage summary in the PR comment.
          coverage-summary-title: "Code Coverage Summary"
          # Failure modes for coverage regression detection:
          # Fail if any changed file has more uncovered lines (pycobertura exit code 2)
          uncovered-statements-increase-failure: false
          # Fail if new uncovered statements are introduced despite overall improvement (pycobertura exit code 3)
          new-uncovered-statements-failure: false
          # Fail if the overall coverage percentage decreases (more forgiving approach)
          coverage-rate-reduction-failure: true
```

An example of the output of the action can be seen below:

![Action output](example.png)
