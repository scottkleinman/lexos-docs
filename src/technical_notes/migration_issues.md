# Migration Issues

## `mkdocstrings` cannot watch source files outside the repository

This prevents building from the source code in the `lexos` repository. Here is what the Zensical docs say:

While it is possible to configure search paths that are external to the
docs project (outside of the parent directory containing the
`zensical.toml` file) in the `paths` option of the Python handler, these
external paths will not be watched for change, meaning that modifying
files within them will not trigger a rebuild and reload. The reason is that
the file agent will refuse to watch files that are outside the project folder
for security reasons. We are working on lifting this limitation. You can
follow progress on these backlog items:

- [Proposal: Configuration](https://github.com/zensical/backlog/issues/47)
- [Allow use of `..` in `docs_dir` and `site_dir`](https://github.com/zensical/backlog/issues/56)
- [Symbolic links pointing outside of `docs_dir`](https://github.com/zensical/backlog/issues/55)

## `git-revision-date-localized` is not yet compatible with Zensical

A workaround is possible, so this is a minor issue.
