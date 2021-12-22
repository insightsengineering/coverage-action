[![SuperLinter](https://github.com/insightsengineering/coverage-action/actions/workflows/linter.yaml/badge.svg)](https://github.com/insightsengineering/coverage-action/actions/workflows/linter.yaml)
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

  _Default_: `True`

* `publish`:

  _Description_: Path to package's root.

  _Required_: `false`

  _Default_: `False`

### Outputs
None
<!-- END_ACTION_DOC -->

## Usage

Example usage:

```yaml
---
name: Code Coverage

on:
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
        uses: actions/checkout@v2

      - name: Your logic to generate the Cobertura XML goes here
        run: echo "Your logic to generate the Cobertura XML goes here"

      - name: Produce the coverage report
        uses: insightsengineering/coverage-action@v1
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
          # Github token to use to publish the PR comment
          token: ${{ secrets.GITHUB_TOKEN }}
```

An example of the output of the action can be seen [here](https://github.com/insightsengineering/coverage-action/pull/5#issuecomment-999738523).
