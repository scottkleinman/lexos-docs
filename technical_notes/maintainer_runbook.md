# Maintainer Runbook: Versioned Docs Deployment

Lexos documentation is published as a versioned site with `mike` on the `gh-pages` branch of the `lexos` repository.

As part of the split-repo migration, `lexos` dispatches docs publishing events to the `lexos-docs` repository. `lexos-docs` performs docs builds and publishes to the `lexos` `gh-pages` branch.

## Required Repository Settings

1. In `lexos`, GitHub Pages must be configured to **Deploy from a branch**.
2. The selected branch must be `gh-pages`.
3. The selected folder must be `/(root)` (not `/docs`).

## Branch Responsibilities

1. `main` stores the documentation source and triggers development docs updates.
2. `gh-pages` stores only rendered documentation output and mike metadata.
3. `docs-v*` tags (for example `docs-v0.1.0-beta.31`) are the canonical historical docs snapshots.

## Automation Behaviour

There are two workflows involved:

1. `.github/workflows/docs-dispatch.yml` in `lexos` sends events to `lexos-docs`.
2. `.github/workflows/docs-versioned.yml` in `lexos-docs` receives dispatch events and publishes docs.
3. `.github/workflows/docs-versioned.yml` in `lexos` is a legacy fallback workflow and should remain manual-only.

The dispatch workflow in `lexos` handles trigger forwarding.

1. Push to `main` (matching paths): dispatches a `dev` docs update request.
2. Push to a tag matching `v*`: dispatches a release-tag publish request.
3. Release published: dispatches a release publish request.
4. Manual dispatch: forwards a manually-audited publish request.

The legacy workflow in `lexos` (`docs-versioned.yml`) only runs when one of the following is true:

1. The run is started manually (`workflow_dispatch`).
2. Repository variable `LEXOS_ENABLE_LEGACY_DOCS_DEPLOY` is set to `true`.

This prevents duplicate publishes during split-repo operation.

The receiver workflow in `lexos-docs` accepts:

1. `repository_dispatch` event type `lexos_docs_publish` from `scottkleinman/lexos`.
2. `workflow_dispatch` for manual operations (`dry-run`, `backfill`, `alias-repair`, `rollback`).

## Alias Policy

1. `stable` points to latest non-prerelease version.
2. `stable-beta` points to latest prerelease version.
3. `latest` points to the latest published release tag.
4. `dev` points to docs generated from `main`.

## Manual Run Requirements (Strict Audit)

Manual runs require:

1. operation type
2. source ref
3. reason
4. change summary

If required inputs are missing, the run fails.

## Baseline and Rollback Guardrails

Before making workflow or dependency changes, capture a baseline from one dry-run and one real publish run summary:

1. Trigger
2. Operation
3. Source ref
4. Docs ref
5. Version label
6. Push to `gh-pages`

Also capture timing for:

1. Total job duration
2. Install section duration (`Install uv`, `Install Python`, `Sync docs tooling dependencies`)

If a streamlining change breaks publish behavior:

1. Revert `.github/workflows/docs-versioned.yml`, `pyproject.toml`, and `uv.lock` to their previous working revisions.
2. Rerun `workflow_dispatch` with `operation=dry-run`.
3. Resume optimization only after baseline behavior is restored.

## Cutover Validation Sequence

To complete cutover safely, run both validations in this order:

1. `lexos-docs` `docs-versioned.yml` with `workflow_dispatch` and `operation=dry-run`.
2. `lexos` `docs-dispatch.yml` with `workflow_dispatch` to send a real `repository_dispatch` event.

Expected outcome:

1. Dry-run verifies checkout + mkdocstrings path + mike execution without pushing to `gh-pages`.
2. Real dispatch produces a `dev` publish from `lexos-docs` to `lexos` `gh-pages`.

## Required Secrets and Variables for Dispatch

`lexos` repository:

1. By default, `.github/workflows/docs-dispatch.yml` uses `github.token` to send `repository_dispatch` to `scottkleinman/lexos-docs`.
2. If cross-repo dispatch fails with permission errors, switch dispatch auth to a fine-grained PAT secret (for example `LEXOS_DISPATCH_PAT`) and re-run.

`lexos-docs` repository:

1. `LEXOS_PAGES_PAT` secret for pushing mike output to `lexos` `gh-pages` (repository-contents write on `lexos`).
2. `.github/workflows/docs-versioned.yml` to handle `repository_dispatch` event type `lexos_docs_publish`.

## Common Operations

### Backfill a Missing Version

During split-repo operation, run this in `lexos-docs` workflow dispatch with:

1. operation = `backfill`
2. source_ref = release tag or commit
3. docs_ref = optional docs snapshot ref (defaults to `docs-<source_ref>`)
4. version_label = desired docs label
5. reason + change_summary

### Repair an Alias

During split-repo operation, run this in `lexos-docs` workflow dispatch with:

1. operation = `alias-repair`
2. alias_name = `stable`, `stable-beta`, `latest`, or `dev`
3. target_version = existing deployed version label
4. reason + change_summary

### Roll Back Stable Docs

During split-repo operation, run this in `lexos-docs` workflow dispatch with:

1. operation = `rollback`
2. target_version = known good deployed version
3. reason + change_summary

This updates `stable` and the default landing version.

### Emergency Local Fallback Deploy from `lexos`

Only use this when `lexos-docs` is unavailable.

1. Run `.github/workflows/docs-versioned.yml` manually from Actions UI.
2. Provide strict audit inputs (`reason`, `change_summary`, and operation parameters).
3. Disable fallback usage once `lexos-docs` is healthy.

## Troubleshooting Checklist

If the site is not deploying:

1. Verify Pages is `gh-pages` + `/(root)`.
2. Confirm `.github/workflows/docs-dispatch.yml` exists on `main`.
3. Confirm dispatch run succeeded in `lexos` and reached `lexos-docs`.
4. Check `lexos-docs` workflow logs for checkout, mkdocstrings source path, and `mike deploy` failures.
5. Confirm both required tokens are present and not expired.
6. Confirm matching docs refs exist in `lexos-docs` for release/tag backfills (`docs-<source_ref>` pattern).
7. If `lexos` fallback workflow ran unexpectedly, check whether `LEXOS_ENABLE_LEGACY_DOCS_DEPLOY` is set to `true`.
8. If dispatch is rejected immediately, verify the source repository in the receiver payload is `scottkleinman/lexos`.

## Local Maintainer Verification

Before changing deployment logic, verify docs build locally:

```bash
cd lexos
uv sync --group docs-ci --no-install-project
cd src
uv run mkdocs build
```

For API docs resolution during local tests, `mkdocs.yml` now includes default source path fallbacks for both local sibling clones (`../../lexos/src`) and CI checkout layout (`../../source/src`).

If your local checkout layout is different, override it explicitly:

```bash
cd lexos-docs/src
MKDOCSTRINGS_PYTHON_PATH=/absolute/path/to/lexos/src PYTHONPATH=/absolute/path/to/lexos/src uv run mkdocs serve
```

## Troubleshooting Local Import Paths

If `mkdocs serve` or `mkdocs build` reports `ImportError` for `lexos.*`, first verify that Python can import `lexos` from your chosen source path:

```bash
cd lexos-docs/src
MKDOCSTRINGS_PYTHON_PATH=/absolute/path/to/lexos/src PYTHONPATH=/absolute/path/to/lexos/src uv run python -c "import lexos; print(lexos.__file__)"
```

If the command fails, fix the path and try again. If it succeeds, rerun:

```bash
uv run mkdocs serve
```
