# jgrep semantic gate

Run [jgrep](https://github.com/kyu1204/jgrep) in CI on TypeSafe Jev. Two modes:

- **diff**: fail the job when a rule written in English matches the PR diff.
- **tests**: list the test files a diff can affect, as a step output and a file.

Use `actions/checkout` with `fetch-depth: 0` so the base ref exists.

## Diff gate

```yaml
on: pull_request
jobs:
  gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: kyu1204/jgrep-action@v1
        with:
          mode: diff
          rule: adds an HTTP endpoint that has no auth check
          api-key: ${{ secrets.TYPESAFE_API_KEY }}   # or openrouter-api-key
```

## Test selection

```yaml
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - id: sel
        uses: kyu1204/jgrep-action@v1
        with:
          mode: tests
          api-key: ${{ secrets.TYPESAFE_API_KEY }}
      - if: steps.sel.outputs.tests != ''
        run: npx vitest run $(cat ${{ steps.sel.outputs.tests-file }})   # or: pytest $(cat ...)
```

## Inputs

| input | default | meaning |
|---|---|---|
| `mode` | `diff` | `diff` or `tests` |
| `rule` | | plain-English rule (diff mode) |
| `base` | `origin/<PR base>` | ref to diff against |
| `api-key` | | TypeSafe key |
| `openrouter-api-key` | | OpenRouter key (either key works) |
| `version` | `0.5.0` | npm version of `jevgrep` |
| `threshold` | `0.7` | minimum probability |

## Outputs

| output | meaning |
|---|---|
| `matched` | `true`/`false` (diff mode) |
| `tests` | newline-separated test files (tests mode) |
| `tests-file` | path of a file with the same list |

## Exit codes

jgrep exits `0` = match, `1` = clean, `2` = error. Diff mode fails the step on `0`
(rule matched) and on `2` (could not run), with distinct `::error::` messages, and
passes on `1`. Tests mode passes on `0` and `1`, fails on `2`.

## Privacy

Code chunks from the diff are sent to TypeSafe (or OpenRouter, if you use that key).

More: [jgrep](https://github.com/kyu1204/jgrep) and its benchmark in `bench/`.
