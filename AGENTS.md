# Agent Requirements

All agents must follow these rules:

1) Fully test your changes before submitting a PR (run the full suite or all relevant tests).
2) PR titles must be descriptive and follow Conventional Commits-style prefixes:
   - Common: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`, `perf:`
   - Support titles: `fix(docs):`, `fix(benchmarks):`, `fix(cicd):`
3) Commit messages must follow the same Conventional Commits-style prefixes and include a short functional description plus a user-facing value proposition.
4) PR descriptions must include Summary, Rationale, and Details sections.
5) Maximize the use of caching in GitHub workflow files to minimize run duration.
6) Use one of `paths` or `paths-ignore` in every workflow file to make sure workflows only run when required.
7) All mypy configuration (flags, overrides, per-module ignores, and file targets) should go in pyproject.toml. Do not split config across CLI args, mypy.ini, setup.cfg, or workflow steps.

Reference: https://www.conventionalcommits.org/en/v1.0.0/
