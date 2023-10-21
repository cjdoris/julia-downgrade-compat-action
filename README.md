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

Run this action after installing Julia (e.g. with `setup-julia`) and before installing Julia
dependencies (e.g. with `julia-buildpkg`).

For example:
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
          skip: [Pkg, TOML]
      - uses: julia-actions/julia-buildpkg@v1
      - uses: julia-actions/julia-runtest@v1
```

In this example, we are running the test suite with the latest version of Julia and
also Julia 1.6, corresponding to `matrix.version`. The `if:` entry only runs the downgrade
action when it is Julia 1.6 running. This means we get one run using latest Julia and
latest packages, and one run using Julia 1.6 and old packages.

The `skip:` input says that we should not attempt to downgrade `Pkg` or `TOML`.

## Inputs

- `skip` is a list of packages to prevent from being downgraded. It should include any
  standard libraries, because these are tied to the Julia version.

- `strict` can be `true`, `false` or `v0` (default is `v0`):
  - `true`: a compat like `1.2.3` becomes `=1.2.3` so that exactly only v1.2.3 is installed.
  - `false`: then it becomes `~1.2.3`, allowing a patch upgrade (v1.2.*).
  - `v0`: is strict only for v0.*.*, so `1.2.3` becomes `~1.2.3` but `0.1.2` becomes `=0.1.2`.

## Supported compat entries

Compats like `1`, `1.2`, `1.2.3`, `^1.2.3`, `~1.2.3`, `=1.2.3`, `1.2.3, 2.3.4` are all supported.

Compats like `1.2.3 - 1.2.5` are not supported.

For list compats like `1.2.3, 2.3.4`, all but the first entry is ignored. Therefore you should put the lowest entry first.
