<!-- BEGIN_ACTION_DOC -->
# Code Coverage Report Action

### Description
Action to convert a Cobertura coverage report to HTML/Markdown.
### Action Type
Composite

### Author
Roche

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