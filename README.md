# Rocket launcher

An experimental workbench with a **coordonator**, a **cercetaș**, and a
**worker**. The cercetaș researches other teams' experiments while the worker
executes accepted directions. The coordonator proposes directions on the
cercetaș's whiteboard, transfers actionable ones to the worker's whiteboard,
and deletes rejected ones from the cercetaș's whiteboard.

Research lives in `research/`, with the same target, whiteboard, and journal
structure as experiments. Each article has a numbered folder with its details.

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
- **Execution:** one cercetaș and one worker let research feed experiments
  while each role preserves a sequential history under one coordonator.
- **Role context:** separate item state and workspace bindings keep article
  completion and recovery from stopping experiments or changing their records.
- **Shutdown:** checking the budget before dispatch and cleaning up owned
  resources on every exit prevents orphaned subagents and locks.
- **Articles:** one folder per article makes source details reusable under
  the existing journal rules.
- **Research evidence:** source applicability and search limitations keep
  rejection arguments and proposed gaps grounded in what was read.
- **Handoff:** separate whiteboards let the coordonator select work and pass
  the remaining question with its supporting findings to the worker.
- **Reuse:** consulting saved findings prevents repeating settled research
  and experiments without a new question or changed conditions.
- **Trajectory:** checkpoints expose deviations before the next step;
  recorded corrections explain why an attempt changed direction.

## Install

```sh
gh release download --repo dsl400/skill-rocket-launcher --pattern rocket-launcher.tar.gz -O - | tar -xz -C ~/.claude/skills
```

Also available through [dsl400/skills](https://github.com/dsl400/skills).
