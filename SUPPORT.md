# Support

## Documentation questions

Open a GitHub issue when:

- A page is unclear.
- A command no longer works.
- A feature is missing from the docs.
- A page references stale behavior.
- Search or navigation does not surface the right page.

Use the docs issue template and include the affected URL or file path.

## Product issues

This repository is for documentation. Product bugs should be filed in the product repository or internal tracker, then linked from a docs issue if documentation needs to change.

## Security issues

Do not open public issues for vulnerabilities, credential leaks, or unsafe production guidance. Use [SECURITY.md](SECURITY.md).

## Local preview

For docs-specific local failures:

```bash
uv sync
uv run zensical build --clean
uv run zensical serve --dev-addr 127.0.0.1:8008
```

Include the build output when asking for help.
