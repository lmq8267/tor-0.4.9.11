# Repository Guidelines

A contributor guide for Tor (`tor-0.4.9.11`). See also the in-tree hacking docs at `doc/HACKING/`.

## Project Structure & Module Organization

Tor is a C daemon built with the GNU Autotools (`configure.ac`, `Makefile.am`).

- `src/lib` — low-level utility libraries: containers, crypto, buffers, compression, encoding, time, dispatch.
- `src/core` — core per-process logic: mainloop, OR protocol, crypto subsystem.
- `src/feature` — feature subsystems: HS, relay, dirauth/dircache/dirclient, control, nodelist, metrics.
- `src/app` — application entrypoint, configuration, and `tor` binary wiring.
- `src/ext` — vendored third-party code (e.g. ed25519, curve25519-donna).
- `src/trunnel` — generated wire-format parsers from `.trunnel` definitions.
- `src/test` — C unit tests, integration tests, fuzz targets, and Python helpers.
- `doc/HACKING` — developer documentation (`CodingStandards.md`, `WritingTests.md`, `Module.md`).
- `changes/` — one file per changelog entry (see below); cleared at release time.
- `contrib/` and `scripts/` — operator tools and maintainer scripts.

## Build, Test, and Development Commands

From a release tarball:
```sh
./configure && make && make install      # standard build
```
From a fresh git clone, run `./autogen.sh` first.

Useful targets (run from the build root):
- `make check` — run bundled unit tests and lint checks (`check-spaces`, `check-changes`).
- `make test-full` — `check` plus Stem and Chutney integration tests.
- `make test-full-online` — additionally runs tests requiring network access.
- `./src/test/test <prefix>/..` — run a subset of unit tests; `--no-fork` disables subprocess isolation; `--debug` enables verbose logging.
- `make check-spaces` — enforce whitespace/indentation rules.
- `make distcheck` — verify the source tree packages correctly (required after build-system edits).
- `scripts/coccinelle/apply.sh` — apply Coccinelle refactors used by the project.

## Coding Style & Naming Conventions

Tor follows C99, K&R indentation (two spaces), Unix line endings, no tabs, ≤79 columns, and `if (x)`/`func(y)` spacing. `.editorconfig` is provided. For full rules, see `doc/HACKING/CodingStandards.md`.

- Use Tor wrappers (`tor_malloc`, `tor_free`, `tor_strdup`, `tor_reallocarray`) instead of libc equivalents.
- Use `const` for new APIs; declare functions with `void foo(void)` not `void foo()`.
- Initialize variables where declared, not at scope top.
- Modules (e.g. `relay`, `dirauth`, `dircache`) are toggled at configure time via `--disable-module-{name}`; see `doc/HACKING/Module.md`.
- Prefer `scripts/maint/` linters (CheckSpace, checkIncludes, practracker) to catch issues locally.

## Testing Guidelines

- Unit tests live in `src/test/test_*.c`; the compiled runner is `./src/test/test`, with slow tests in `./src/test/test-slow`.
- Fuzz targets are in `src/test/fuzz/` (run `make check` to execute static fuzzing).
- Integration tests use Stem (`STEM_SOURCE_DIR` + `make test-stem`) and Chutney (`CHUTNEY_PATH` + `make test-network`).
- Coverage: configure with `--enable-coverage`, run tests, then `./scripts/test/coverage <tmpdir>`.
- Every non-trivial change should include unit tests where feasible.

## Commit & Pull Request Guidelines

- Base bug fixes on `maint-*` branches; base features on `main`. Never branch off `release-*` or tags.
- Sent patches as a git branch (best), `git format-patch`, or unified diff.
- Configure with `--enable-fatal-warnings` before building; ensure `make check` passes.
- Add a changelog file under `changes/`, named `ticketNNNNN` or `bugNNNNN`. Format, validated by `make check-changes` (`scripts/maint/lintChanges.py`):
  ```
  o Minor bugfixes (subsystem):
    - Describe the change. Fixes bug 1234; bugfix on 0.3.5.1-alpha.
  ```
- Use present-tense, imperative voice; say "Relays", not "servers" or "nodes".
- Only compatible licenses are accepted (3-clause BSD preferred); see `CONTRIBUTING` and `LICENSE`.
