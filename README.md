# Rocket launcher

The coordonator selects experiments, the cercetaș researches in parallel,
and a fresh worker executes each experiment. The skill lives in
`skills/rocket-launcher/SKILL.md` and uses [lnl](https://github.com/dsl400/lnl).

## Check rationale

- **Worker context:** reproducible inputs and accepted directions keep each
  fresh worker focused on the remaining question.
- **Research:** separate records preserve article evidence and experiment
  results while sharing the established journal format.
- **Test scope:** unit, smoke, and development checks answer routine questions;
  full runs require milestone evidence to justify their cost.
- **Steering:** changing trajectory between experiments preserves the active
  frozen run. Clearing applied content acknowledges the user's direction.
- **Results and ownership:** verified measurements support decisions; recorded
  ownership keeps recovery and cleanup accountable across automation handoff.
- **Execution:** deterministic utilities and event-based waiting avoid repeated
  agent work for scaffolding, verification, summaries, and idle monitoring.
- **Limits:** 60-character descriptions keep paths readable; 256-character
  overviews keep history affordable; 256-line journals of 80 characters link
  larger evidence. The billion-iteration guards provide continuation points.

## Install

```sh
gh release download --repo dsl400/skill-rocket-launcher --pattern rocket-launcher.tar.gz -O - | tar -xz -C ~/.claude/skills
```

Also available through [dsl400/skills](https://github.com/dsl400/skills).
