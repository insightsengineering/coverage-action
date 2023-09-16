[![SuperLinter](https://github.com/insightsengineering/coverage-action/actions/workflows/lint.yaml/badge.svg)](https://github.com/insightsengineering/coverage-action/actions/workflows/lint.yaml)
[![Test](https://github.com/insightsengineering/coverage-action/actions/workflows/test.yaml/badge.svg)](https://github.com/insightsengineering/coverage-action/actions/workflows/test.yaml)

<!-- BEGIN_ACTION_DOC -->
# Code Coverage Report Action

### Description

Action that converts a Cobertura XML report into a markdown report.
### Action Type

Composite

### Author

Insights Engineering

### Inputs

* `token`:

  _Description_: Github token to use to publish the check.

  _Required_: `false`

  _Default_: `${{ github.token }}`

* `path`:

  _Description_: Path to the Cobertura coverage XML report.

  _Required_: `false`

  _Default_: `.`

* `threshold`:

  _Description_: The minimum allowed coverage percentage, as a real number.

  _Required_: `false`

  _Default_: `0`

* `fail`:

  _Description_: Fail the action when the minimum coverage was not met.

  _Required_: `false`

  _Default_: `true`

* `publish`:

  _Description_: Publish the coverage report as an issue comment.

  _Required_: `false`

  _Default_: `false`

* `diff`:

  _Description_: Create a diff of the coverage report.

  _Required_: `false`

  _Default_: `false`

* `diff-branch`:

  _Description_: Branch to diff against.

  _Required_: `false`

  _Default_: `main`

* `diff-storage`:

  _Description_: Branch where coverage reports are stored for diff purposes.

  _Required_: `false`

  _Default_: `_xml_coverage_reports`

* `coverage-reduction-failure`:

  _Description_: Fail the action if code coverage decreased compared to the `diff-branch` is decreased.

  _Required_: `false`

  _Default_: `false`

* `new-uncovered-statements-failure`:

  _Description_: Fail the action if any new uncovered statements are introduced compared to the `diff-branch`.

  _Required_: `false`

  _Default_: `false`

* `pycobertura-exception-failure`:

  _Description_: Fail the action in case of a `Pycobertura` exception.

  _Required_: `false`

  _Default_: `true`

* `togglable-report`:

  _Description_: Make the code coverage report togglable.

  _Required_: `false`

  _Default_: `false`

### Outputs

None
<!-- END_ACTION_DOC -->

## How it works

This tool makes use of the [PyCobertura](https://github.com/aconrad/pycobertura) CLI tool to produce the summary outputs. The action also supports `diff`s against a given branch and makes use of a remote branch to store reports, which can be specified via this action.

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
        uses: insightsengineering/coverage-action@v2
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
```

An example of the output of the action can be seen below:

![Action output](example.png)
