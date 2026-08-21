# Current Stage

> Single source of truth the core agent (`AGENTS.md`) reads before deciding which persona to apply.
> Whoever is driving updates this file the moment the mob moves to a new stage or switches mode — takes 10 seconds.

```yaml
current_stage: requirements   # requirements | design | task-breakdown | build | test-pass | polish
mode: mob                      # mob | breakout
active_lead: BA                # BA | Dev | Test | All | "BA + Dev (breakout)"
updated_by: <driver name>
updated_at: <HH:MM>
notes: >
  Workshop kickoff — starting requirements stage.
```

## Stage values

| Value | Meaning | Mode allowed |
|---|---|---|
| `requirements` | User stories + acceptance criteria | mob or breakout |
| `design` | Architecture / SAD | mob or breakout (parallel with requirements OK) |
| `task-breakdown` | Dev spec / task list | **full mob only** |
| `build` | Spec-driven build loop | **full mob only** |
| `test-pass` | Test strategy + full pass | breakout to draft strategy; full mob to review |
| `polish` | Demo prep, stable build, token log | **full mob only** |

## Mode values

| Value | Meaning |
|---|---|
| `mob` | Everyone on one compute, one driver |
| `breakout` | Small subgroup on separate machine — allowed for `requirements`, `design`, and early `test-pass` strategy drafting |

**Reconvene rule:** before flipping `mode` back to `mob` after a breakout, the driver confirms the full team has reviewed the breakout output. `current_stage` never advances into `task-breakdown`, `build`, or `polish` while `mode: breakout` is still set.
