# julia-downgrade-compat-action

This is a GitHub action for Julia packages which modifies Project.toml so that the oldest
compatible versions of dependencies are installed.

This can be used to ensure your tests pass on old versions of packages. If they fail, this
suggests you need to increase the compat lower bounds.

For example, if your Project.toml has this compat entry:
```toml
[compat]
julia = "1.6"
Foo = "1.2.3"
Bar = "0.1.2"
```
then this package will modify it to:
```toml
[compat]
julia = "1.6"
Foo = "~1.2.3"
Bar = "=0.1.2"
```

## Usage

Run this action after installing Julia (e.g. with `setup-julia`) and before installing Julia
dependencies (e.g. with `julia-buildpkg`). Something like:

```yaml
jobs:
  test:
    steps:
      - uses: actions/checkout@v3
      - uses: julia-actions/setup-julia@v1
      - uses: cjdoris/julia-downgrade-compat-action@v1
      - uses: julia-actions/julia-buildpkg@v1
      - uses: julia-actions/julia-runtest@v1
```
