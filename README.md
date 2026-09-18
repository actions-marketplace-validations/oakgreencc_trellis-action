# trellis-action

Lightweight composite GitHub Action that runs [trellis](https://github.com/jayminwest/trellis)
against a TypeScript workspace and exposes the 0–100 **sloppiness index** (lower is better)
as a step output. Also posts the index to the job summary.

## Usage

```yaml
- uses: actions/checkout@v6
- id: trellis
  uses: oakgreencc/trellis-action@v1
  with:
    path: .                # workspace to audit
- run: echo "sloppiness index: ${{ steps.trellis.outputs.index }} (grade ${{ steps.trellis.outputs.grade }})"
```

Gate on a threshold:

```yaml
- run: |
    test "$(printf '%.0f' "${{ steps.trellis.outputs.index }}")" -le 40 \
      || { echo "::error::sloppiness index too high"; exit 1; }
```

Or declare a `trellis.yaml` policy in the audited repo — a tripped policy (trellis exit `2`)
fails the step unless `fail-on-policy: "false"`.

## Inputs

| name | default | description |
|---|---|---|
| `path` | `.` | Workspace to audit |
| `trellis-ref` | `main` | Git ref of `jayminwest/trellis` to run (pin a SHA for reproducible scores) |
| `report-path` | `trellis-report.json` | Where the JSON report is written |
| `baseline` | | Previous `report.json` to compare against |
| `config` | | Explicit `trellis.yaml` |
| `fail-on-policy` | `true` | Fail the step on trellis exit `2` |
| `extra-args` | | Extra args passed verbatim to `trellis audit` |
| `grade-thresholds` | `10,20,35,50` | Inclusive upper bounds for grades A,B,C,D; above the last → F |

## Outputs

| name | description |
|---|---|
| `index` | 0–100 sloppiness index (lower is better) |
| `grade` | Letter grade `A`–`F` from the index (see `grade-thresholds`) |
| `partial` | `true` if scored from an incomplete analysis |
| `completeness` | `complete` / `incomplete` |
| `findings` | Number of findings |
| `exit-code` | `0` pass · `2` policy failure · `1` operational error |
| `report-path` | Path to the JSON report (upload it with `actions/upload-artifact`) |
| `analyzer-version` | trellis version that produced the report |

## Grading

The index is trellis's; the letter grade is this action's convention over it (lower index = better):

| grade | index |
|---|---|
| A | ≤ 10 |
| B | ≤ 20 |
| C | ≤ 35 |
| D | ≤ 50 |
| F | > 50 |

Override with `grade-thresholds: "5,15,30,45"`.

## Notes

- trellis isn't on npm yet, so the action clones it at `trellis-ref` and runs it with Bun
  (`oven-sh/setup-bun@v2`). When it's published, swap the install step for `bun install -g @os-eco/trellis-cli@<ver>`.
- An operational failure (exit `1`, or no report written) always fails the step.

## License

MIT — see [LICENSE](LICENSE). Trellis itself is MIT (© Jaymin West).

## Releases

Every push to `main` is versioned from [conventional commits](https://www.conventionalcommits.org/)
(`feat:` → minor, `fix:` → patch, `BREAKING CHANGE`/`!` → major). A `vX.Y.Z` tag and GitHub
release are cut automatically and the floating major tag (`v1`, …) is moved. Commits with no
releasable type (`chore:`, `docs:`, …) don't cut a release.
