# devin-devkit-action

GitHub Action that checks Devin skills and rules in a checkout with
[`devin-skill-catalog`](https://pypi.org/project/devin-skill-catalog/) —
the inventory/lint package of the
[devin-devkit](https://github.com/Icaro0310/devin-devkit) product in the
devin-* ecosystem.

It runs `devin-skill-catalog lint` (or an offline `gate g1`/`g2`) on the
workspace and fails the job on `FAIL` findings. Read-only by design:
the action only inspects. It never runs `install`, `update`,
`quarantine`, `promote` or any other mutation — those surfaces are
deliberately not exposed here.

## Usage

```yaml
- uses: actions/checkout@v4
- uses: Icaro0310/devin-devkit-action@v0.1.0
```

With options:

```yaml
- uses: Icaro0310/devin-devkit-action@v0.1.0
  with:
    paths: ".devin shared/.devin"    # default: "."
    gate: g1                          # default: lint; or g2 with packs-dir
    fail-on-warn: "true"             # default: only FAIL fails
    version: "devin-skill-catalog==0.1.0"
```

## The two distinct failures

| Situation | Exit | Meaning |
|---|---|---|
| tool could not run / unreadable output | 2 | tooling failure — not a lint verdict |
| FAIL findings (or WARN with `fail-on-warn`) | 1 | real findings, named per item and line |

## Inputs / outputs

- `paths` — space-separated workspace roots, `.devin` dirs, skill dirs
  or files (default `.`).
- `gate` — `""` runs `lint` (default); `g1` runs the offline structural
  gate; `g2` also verifies rubric packs load (needs `packs-dir`).
- `packs-dir` — devin-evals rubric pack dir for `gate: g2`.
- `include-user` — also scan `~/.devin` and `~/.config/devin` (default
  `false`; CI checks the checkout only).
- `fail-on-warn` — WARN findings also fail the job (default `false`).
- `version` — pip specifier (default `devin-skill-catalog>=0.1,<0.2`).
- outputs: `verdict` (`PASS`/`WARN`/`FAIL`), `findings`, `failures`.

## Platform support

Runs on any GitHub-hosted runner — lint and the offline `g1`/`g2` gates
need no Devin install; Python 3.12 comes from `actions/setup-python`.

## Notes

- Read-only by design — lint/gate inspect items and emit findings;
  without `--apply` nothing is recorded anywhere, and this action never
  passes `--apply`.
- The satellite repo is only the Action distribution vehicle; all
  catalog logic lives in
  [`devin-skill-catalog`](https://github.com/Icaro0310/devin-devkit/tree/main/packages/skill-catalog).
