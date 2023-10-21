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
Baz = "1, 2, 3"
```
then this package will modify it to:
```toml
[compat]
julia = "1.6"
Foo = "~1.2.3"
Bar = "=0.1.2"
Baz = "~1.0.0"
```

## Usage

```yaml
- uses: cjdoris/julia-downgrade-compat-action@v1
  with:
    # Comma-separated list of packages to not downgrade. This should include any standard
    # libraries because these have versions tied to the Julia version.
    # Example: Pkg, TOML
    # Default: ''
    skip: ''

    # When strict, a compat entry like "1.2.3" becomes "=1.2.3" so that exactly v1.2.3 is
    # installed. When not strict, it becomes "~1.2.3" so that patch upgrades are allowed
    # (v1.2.*). This entry can be 'true' (strict), 'false' (not strict) or 'v0' (strict for
    # "0.*.*" and not strict otherwise).
    # Default: 'v0'
    strict: ''
```

## Example

For example, here is the action being used as part of a standard Julia test workflow:
```yaml
jobs:
  test:
    strategy:
      matrix:
        version: ['1', '1.6']
    steps:
      - uses: actions/checkout@v3
      - uses: julia-actions/setup-julia@v1
        with:
          version: ${{ matrix.version }}
      - uses: cjdoris/julia-downgrade-compat-action@v1
        if: ${{ matrix.version == '1.6' }}
        with:
          skip: Pkg,TOML
      - uses: julia-actions/julia-buildpkg@v1
      - uses: julia-actions/julia-runtest@v1
```

The action requires Julia to be installed, so must occur after `setup-julia`. It runs just
before `julia-buildpkg` so that the Project.toml is modified before installing any packages.

In this example, we are running the test suite with the latest version of Julia and
also Julia 1.6, corresponding to `matrix.version`. The `if:` entry only runs the downgrade
action when it is Julia 1.6 running. This means we get one run using latest Julia and
latest packages, and one run using Julia 1.6 and old packages.

The `skip:` input says that we should not attempt to downgrade `Pkg` or `TOML`.

## Supported compat entries

Compats like `1`, `1.2`, `1.2.3`, `^1.2.3`, `~1.2.3`, `=1.2.3`, `1.2.3, 2.3.4` are all supported.

Compats like `1.2.3 - 1.2.5` are not supported.

For list compats like `1.2.3, 2.3.4`, all but the first entry is ignored. Therefore you should put the lowest entry first.
