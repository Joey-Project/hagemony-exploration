# Codex-Gated Repository Template

This template starts a repository with the Codex review gate workflow already on
the default branch. It also includes a modular CI generator for adding
project-specific formatter, linter, test, and benchmark entrypoints after a
repository is created from the template.

## Included

- `.github/workflows/codex-review-gate.yml`
- `.github/workflows/codex-review-gate-controller.yml`
- `.github/CODEOWNERS`
- `.gitignore`
- `scripts/setup-ci.mjs`
- this README

The read-only verifier runs on pull request changes and produces the native
`codex/github-review-gate` check for the current head. The controller handles
Codex bot comment wake-ups and protected, single-PR manual `begin-review` or
`reconcile` runs. Both workflows use the owner-approved floating `v2` major of
`JoeyTeng/codex-review-gate-action`; compatible updates are intentional.

## Generate Project CI

Run the setup script from a new repository created from this template:

```bash
node scripts/setup-ci.mjs
```

With no arguments, the script opens an interactive selector. For repeatable setup,
pass modules explicitly:

```bash
node scripts/setup-ci.mjs --tool js-ts --tool python --tool docker --tool markdown
node scripts/setup-ci.mjs --all --benchmark --dry-run
```

The script writes `.github/workflows/ci.yml` plus the selected tool configs. It is
idempotent when generated files have not changed. If a target file already exists
with different content, the script refuses to overwrite it unless `--force` is
provided. Use `--dry-run` to inspect planned writes first.

Supported modules:

- `js-ts`: pnpm, ESLint, Prettier, Vite, and Vitest.
- `python`: uv, Ruff, Pyright, and pytest.
- `swift`: swift-format, SwiftLint, and `swift test`.
- `go`: `gofmt`, `go vet`, and `go test`.
- `rust`: `cargo fmt`, `cargo clippy`, and `cargo test`.
- `github-actions`: actionlint.
- `bash`: shfmt, shellcheck, and `bash -n`.
- `markdown`: Prettier and markdownlint-cli2.
- `docker`: hadolint and `docker buildx build --check`.

`--benchmark` creates `scripts/benchmark.sh` only. It does not create or enable a
benchmark workflow, and benchmark commands are not part of the default PR gate.
The script includes benchmark entries for JavaScript/TypeScript, Python, Go, and
Rust when those modules are selected.

## After Creating a Repository

1. Add the project source, tests, and license.
2. Run `node scripts/setup-ci.mjs` and commit the generated CI/tooling files.
3. Install or lock generated dependencies where applicable, such as `pnpm install`
   for JavaScript/TypeScript or Markdown modules.
4. Confirm both review-gate workflows and `.github/CODEOWNERS` are on the default
   branch before requiring the new check.
5. Follow the [v2 installation guide](https://github.com/Joey-Tools/codex-review-gate/blob/master/docs/install/human.md)
   from `Joey-Tools/codex-review-gate` to stage the ruleset, exercise a separate
   exact-head canary, and then require `codex/github-review-gate` from GitHub
   Actions. Keep strict up-to-date, Code Owner review, stale-review dismissal,
   and resolved-conversation protections. Do not require the check while these
   workflows exist only on a feature branch.

## Optional Repository Variables

- `CODEX_REVIEW_GATE_USE_UBUNTU_LATEST=true`: use `ubuntu-latest` instead of the
  default `ubuntu-slim` runner when needed.
- `CODEX_REVIEW_GATE_LIMITS_PROFILE=expanded`: raise the supported scan limits
  for an unusually large pull request when the gate summary directs it.

The v2 workflows do not run a scheduled retry. When a provider result
arrives only as a review or reaction, or an edited carrier needs reevaluation,
run the default-branch controller's `reconcile` dispatch for that one pull
request and its exact current head.

## Template Maintenance

Run the generator tests with Node's built-in test runner:

```bash
node --test test/setup-ci.node-test.mjs
```
