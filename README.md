# GitHub Actions: Artifacts & Outputs Example

A dummy example project demonstrating how to use **artifacts** and **job outputs** in GitHub Actions workflows.

## What This Project Shows

This repo is a learning playground for two core GitHub Actions features:

1. **Artifacts** — Files produced by a job that can be shared across jobs or downloaded after a workflow run completes.
2. **Outputs** — String values passed between steps and jobs within the same workflow run.

## Project Structure

```
gh-data/
├── .github/
│   └── workflows/
│       ├── artifacts-demo.yml      # Upload & download artifacts
│       ├── outputs-demo.yml        # Pass outputs between jobs
│       └── combined-demo.yml       # Use both together
├── scripts/
│   ├── build.sh                    # Generates a fake build artifact
│   └── generate-data.sh            # Produces sample JSON data
└── README.md
```

## Example 1: Artifacts

Artifacts are great for sharing build outputs, test reports, logs, or any files between jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate build files
        run: |
          mkdir -p dist
          echo "Hello from build" > dist/output.txt
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  consume:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: cat output.txt
```

## Example 2: Job Outputs

Outputs are best for passing small string values (versions, IDs, flags) between jobs.

```yaml
jobs:
  generate:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.set-version.outputs.version }}
      build-id: ${{ steps.set-version.outputs.build-id }}
    steps:
      - id: set-version
        run: |
          echo "version=1.2.3" >> $GITHUB_OUTPUT
          echo "build-id=$(date +%s)" >> $GITHUB_OUTPUT

  consume:
    runs-on: ubuntu-latest
    needs: generate
    steps:
      - run: |
          echo "Version: ${{ needs.generate.outputs.version }}"
          echo "Build ID: ${{ needs.generate.outputs.build-id }}"
```

## Example 3: Step Outputs (within one job)

```yaml
steps:
  - id: compute
    run: echo "result=42" >> $GITHUB_OUTPUT
  - run: echo "The answer is ${{ steps.compute.outputs.result }}"
```

## Artifacts vs Outputs: When to Use Which

| Use Case                     | Artifacts                | Outputs                  |
| ---------------------------- | ------------------------ | ------------------------ |
| File or directory            | Yes                      | No                       |
| Small string value           | Overkill                 | Yes                      |
| Persists after workflow ends | Yes (up to 90 days)      | No                       |
| Available across jobs        | Yes (via download)       | Yes (via `needs`)        |
| Size limit                   | 10 GB (default retention)| ~1 MB per output map     |

## Running the Examples

1. Fork or clone this repo.
2. Push to your own GitHub repo.
3. Navigate to the **Actions** tab and trigger any of the demo workflows manually (`workflow_dispatch`).
4. Inspect the workflow run — artifacts appear in the run summary; outputs are visible in the job logs.

## Key Gotchas

- `actions/upload-artifact@v4` and `download-artifact@v4` are **not backwards compatible** with v3 — pick one version and stick with it.
- Outputs are always strings — encode booleans, numbers, or JSON yourself.
- Artifact names must be unique within a workflow run (in v4).
- Matrix jobs need unique artifact names per matrix combination, e.g. `name: build-${{ matrix.os }}`.

## References

- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [Defining outputs for jobs](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/download-artifact](https://github.com/actions/download-artifact)

## License

MIT — this is a demo project, use it however you'd like.
