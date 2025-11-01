# mypycify GitHub Action

**mypycify** is a reusable composite GitHub Action designed to speed up Python wheel builds in CI by leveraging multiple caching strategies. It sets up Python, restores ccache, builds a wheel for the current OS and Python version, saves ccache, and uploads the wheel artifact. By caching both build artifacts and compiler cache, it significantly reduces build times for repeated runs with the same code.

## Why this action exists

- **Faster CI builds:** By caching both Python build artifacts and compiler cache, repeated builds are much faster.
- **Reusable and configurable:** Works for any Python project, with customizable hash key parts to control cache invalidation.
- **Artifact management:** Automatically uploads built wheels for use in downstream jobs or releases.
- **Optional source normalization and commit/PR automation:** Can normalize C files for diffchecking and automatically commit or open a PR with changes.

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
    # ccache: false  # Set to true to enable ccache for C/C++ compilation
    # push-source: false  # Set to true to enable commit/PR automation
    # commit-message: "chore: update build artifacts"
    # normalize-source: false  # Set to true to normalize C files for diffchecking
```

**Note:** The `hash-key` input must be provided as a multiline string (YAML block scalar), with one file glob per line. Do not use a YAML list or a comma-separated string—use the `|` syntax as shown above.

> **Tip:**  
> The `ccache` input controls whether ccache is used for C/C++ compilation (Linux/macOS only).  
> - For small libraries, ccache installation can take 30–120 seconds and may outweigh the benefit, so disabling ccache is recommended for fast builds.  
> - For larger libraries with significant C/C++ compilation, enabling ccache can greatly speed up repeated builds.  
> - The default is `false`.

## Inputs

| Name              | Description                                                                 | Required | Default |
|-------------------|-----------------------------------------------------------------------------|----------|---------|
| python-version    | Python version to use (passed to actions/setup-python)                      | Yes      |         |
| hash-key          | File globs (multiline string, one per line) to include in the hash key for caching. | Yes      |         |
| pip-cache-dependency-path | Dependency files for actions/setup-python pip cache                 | No       | ""      |
| ccache            | Enable ccache for C/C++ compilation (Linux/macOS only). See tip above.      | No       | false   |
| push-source       | Enable auto-commit and push or PR of changes if build output changes.       | No       | false   |
| commit-message    | Commit message to use when committing changes.                              | No       | "chore: compile C files for source control" |
| normalize-source  | Enable normalization of C files for diffchecking.                           | No       | false   |

## Outputs

| Name          | Description                        |
|---------------|------------------------------------|
| artifact-name | The name of the uploaded artifact.  |

## Commit/PR Automation

If `push-source: true` is set and changes are detected after the build (and optional normalization), the action will:
- **Directly commit and push** changes if running on a branch in the main repository (with write permissions).
- **Open a Pull Request** with the changes if running in a fork, on a PR, or if direct commit is not possible.

If `normalize-source: true` is set, normalization of C files for diffchecking will be performed before the commit/PR step.  
**Note:** `normalize-source: true` requires `push-source: true`.

## Example Workflow

```yaml
name: Compile

on:
  pull_request:
    branches: [master]

jobs:
  build-ubuntu:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          ref: ${{ github.head_ref }}

      - name: Drop existing build files
        run: rm -rf build

      - name: Build, normalize, and push/PR C files
        uses: BobTheBuidler/mypycify@master
        with:
          python-version: "3.14"
          hash-key: |
            pyproject.toml
            setup.py
            tox.ini
            faster_web3/**/*.py
            faster_ens/**/*.py
            **/*.c
            **/*.h
          pip-cache-dependency-path: |
            pyproject.toml
            setup.py
            tox.ini
          ccache: true
          push-source: true
          commit-message: "chore: compile C files for source control"
          normalize-source: true
```

**How to get the artifact name after the step completes**

Assign an `id` to the step using this action (e.g., `id: mypycify`). In subsequent steps, you can access the artifact name with:

```yaml
${{ steps.mypycify.outputs.artifact-name }}
```

## License

MIT (or specify your license)
