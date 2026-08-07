# How API Docs are Generated

## Building Locally

To build the Lexos Documentation website locally, you must clone both the `lexos` and `lexos-docs` repos *and* they must be located in the same parent directory.

- `mkdocstrings` is configured in `mkdocs.yml`
- It uses the Python handler with these source paths:
  - `../../lexos/src`
  - `../../source/src`

That means:

- when you run `uv run mkdocs serve` from `lexos-docs/src`, `mkdocstrings` looks for the `lexos` package in a sibling scott path
- the first path is a local sibling clone of the `lexos` repo: `../../lexos/src`
- the second path is a fallback path used by CI when the source is checked out into `../../source/src`

So locally, if you want API docs to build, you need a local copy of the `lexos` source available at `../../lexos/src` relative to `lexos-docs/src`.

Although you can re-configure these paths in `mkdocs.yml` on your local system, these changes should not be committed to the repository. If you *really* need to store your local repositories in different locations, you can set `MKDOCSTRINGS_PYTHON_PATH` / `PYTHONPATH` to point to the source directory when running `mkdocs serve`:

```bash
MKDOCSTRINGS_PYTHON_PATH=/absolute/path/to/lexos/src PYTHONPATH=/absolute/path/to/lexos/src uv run mkdocs serve
```

Or you can export the paths first:

```bash
export MKDOCSTRINGS_PYTHON_PATH=/absolute/path/to/lexos/src
export PYTHONPATH=/absolute/path/to/lexos/src
uv run mkdocs serve
```

## What the CI/Workflow Does

- in GitHub Actions, the workflow checks out the `lexos` repo under `source/`
- then it sets `MKDOCSTRINGS_PYTHON_PATH` / `PYTHONPATH` to that checked-out source
- so CI also uses a local checkout, not a remote download during the actual MkDocs render
