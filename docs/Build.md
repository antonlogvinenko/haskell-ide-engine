## Build system

`haskell-ide-engine` is built using the library `shake`. The build descriptions are defined in the file `install.hs`.

Previously, `haskell-ide-engine` was built using a `Makefile` on Unix systems and a `PowerShell` script on Windows. By replacing both scripts by a Haskell-based solution, the build process is unified.

### Design goals

The design of the build system has the following main goals:

* works identically on every platform
* has minimal run-time dependencies:
    - a working installation of `stack`
    - `git`
* is completely functional right after simple `git clone`
* one-stop-shop for building all executables required for using `hie` in IDEs.

See the project's `README` for detailed information about installing `hie`.

### Tradeoffs

#### `shake.yaml`

A `shake.yaml` is required for executing the `install.hs` file.

* It contains the required version of `shake`.
* In contrast to the other `*.yaml` it does not contain the submodules, which is necessary for `stack` to work even before the submodules have been initialized.

It is necessary to update the `resolver` field of the `shake.yaml` if the script should run with a different `GHC`.

#### `install.hs` installs a GHC

Before the code in `install.hs` can be executed, `stack` installs a `GHC`, depending on the `resolver` field in `shake.yaml`. This is necessary if `install.hs` should be completely functional right after a fresh `git clone` without further configuration.

This may lead to an extra `GHC` to be installed by `stack` if not all versions of `haskell-ide-engine` are installed.

#### `stack` is a build dependency

Currently, it is not possible to build all `hie-*` executables automatically without `stack`, since the `install.hs` script is executed by `stack`.

Other parts of the script also depend on `stack`:

* finding the local install-dir `stack path --local-bin`
* finding and installing different `ghc` versions

#### `install.hs` executes `cabal install Cabal`

`ghc-mod` installs `cabal-helper` at runtime depending on the `ghc` used by the project, which can take a long time upon startup of `hie`. The `install.hs` script speeds up this process by calling `cabal install Cabal` upon build.

Hopefully, this behaviour can be removed in the future.
