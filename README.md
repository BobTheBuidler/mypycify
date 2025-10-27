# mypycify GitHub Action

**mypycify** is a reusable composite GitHub Action designed to speed up Python wheel builds in CI by leveraging multiple caching strategies. It sets up Python, restores ccache, builds a wheel for the current OS and Python version, saves ccache, and uploads the wheel artifact. By caching both build artifacts and compiler cache, it significantly reduces build times for repeated runs with the same code.

## Why this action exists

- **Faster CI builds:** By caching both Python build artifacts and compiler cache, repeated builds are much faster.
- **Reusable and configurable:** Works for any Python project, with customizable hash key parts to control cache invalidation.
- **Artifact management:** Automatically uploads built wheels for use in downstream jobs or releases.

## Caching strategies

- **Wheel artifact caching:** The built wheel (`dist/*.whl`) is cached using a unique key based on the OS, Python version, and a hash of user-specified files. If the cache is hit, the build is skipped and the cached wheel is used.
- **ccache compiler cache:** The C/C++ compiler cache is restored and saved using a key that includes the OS, Python version, and a hash of relevant source files. This speeds up builds that involve native extensions.
- **Dependency caching:** Optionally, pip dependencies can be cached by specifying `pip-cache-dependency-path`.

## Usage

Add this step to your workflow:

```yaml
- uses: BobTheBuidler/mypycify@v0.0.1
  with:
    python-version: '3.11'
    hash-key: |
      pyproject.toml
      my_lib/**/*.py
      **/*.c
      **/*.h
    # Optional:
    # pip-cache-dependency-path: requirements.txt
```

**Note:** The `hash-key` input must be provided as a multiline string (YAML block scalar), with one file glob per line. Do not use a YAML list or a comma-separated string—use the `|` syntax as shown above.

## Inputs

| Name                      | Description                                                                 | Required | Default |
|---------------------------|-----------------------------------------------------------------------------|----------|---------|
| python-version            | Python version to use (passed to actions/setup-python)                      | Yes      |         |
| hash-key                  | File globs (multiline string, one per line) to include in the hash key for caching. | Yes      |         |
| pip-cache-dependency-path | Dependency files for actions/setup-python pip cache                         | No       | ""      |

## Outputs

| Name          | Description                        |
|---------------|------------------------------------|
| artifact-name | The name of the uploaded artifact. |

## Example Workflow

```yaml
name: Build Wheel

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: BobTheBuidler/mypycify@v0.0.1
        with:
          python-version: '3.11'
          hash-key: |
            pyproject.toml
            setup.py
            faster_web3/**/*.py
            faster_ens/**/*.py
            **/*.c
            **/*.h
          # pip-cache-dependency-path: requirements.txt
```

## License

MIT (or specify your license)
