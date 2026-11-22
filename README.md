# Rocket launcher

A sequential experimental workbench. The main agent reads accumulated
knowledge and chooses an attempt. One subagent executes it in bounded steps;
the main agent checks the trajectory and continues, redirects or stops it.

The skill lives in `skills/rocket-launcher/SKILL.md`; its flow uses
[lnl](https://github.com/dsl400/lnl).

## Check rationale

- **Self-contained exercises:** local inputs, steps, dependencies and evaluation
  criteria let a fresh worker reproduce an attempt from workspace artifacts.
- **Active context:** target and whiteboard carry the facts needed for execution;
  historical handovers and other-project references belong in archives.
- **Relative paths:** explicit base directories keep commands portable.
- **Evaluation isolation:** withheld references prevent answers from leaking into
  the system being tested.
- **Target:** a measurable criterion determines when the loop has succeeded.
- **Overview:** 256 characters per launch keeps the full history affordable
  to read at the start of each session.
- **Evidence:** the launch journal holds result details within 256 lines of
  80 characters; artifact and transcript references provide deeper evidence.
- **Whiteboard:** editable ideas let the next attempt use current reasoning.
- **Continuity:** an entry marked before execution makes interrupted work
  discoverable by the next agent.
- **Identity:** the launch number links overview and directory; a description
  of at most 60 characters keeps directory names readable.
- **Execution:** one executing subagent preserves a sequential chain of
  attempts while the main agent supervises.
- **Trajectory:** checkpoints expose deviations before the next step;
  recorded corrections explain why an attempt changed direction.

## Install

```sh
gh release download --repo dsl400/skill-rocket-launcher --pattern rocket-launcher.tar.gz -O - | tar -xz -C ~/.claude/skills
```

Also available through [dsl400/skills](https://github.com/dsl400/skills).
