---
name: rocket-launcher
description: Use when running a sequential experimental workbench with a shared target, editable whiteboard, and a launch history across agent sessions.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---

# Rocket launcher

The main agent supervises one executing subagent. Attempts run sequentially.
Both start with the target, whiteboard and complete launch overview, then
open launch details as needed.
Repeat the flow for successive launches until the target is met, work is
blocked, or the agreed budget or scope is exhausted.

## Trajectory correction

Use a two-step cycle within each attempt:

1. The subagent executes a bounded step and reports the action, observed
   effects and intended next step to the main agent.
2. The main agent compares that evidence and next step with `target.md`
   and the launch hypothesis, then continues, redirects or stops execution.

The main agent defines checkpoints with the subagent before execution and
inspects available progress between them. The subagent waits for direction
at each checkpoint. Redirect with the specific deviation and next action;
interrupt promptly when continuing would deepen the deviation. Confirm the
worker and its launched processes have stopped before replacing it.

The main agent owns trajectory decisions and journal finalization. The
subagent executes the assigned attempt and supplies evidence. Keep one
executing subagent, with no further delegation. Record corrections and
stops with their effects under `result`; reference the available main and
subagent harness transcripts in the launch journal.

## Workspace

Paths below are relative to the user's project. Preserve existing content.

```text
workspace/
  target.md
  whiteboard.md
workbench/
  jurnal.md
  <n>-<description>/
    jurnal.md
```

`target.md` holds the exercise and measurable success criterion.
`whiteboard.md` is editable working state: add, adjust or withdraw ideas as
work progresses. Launch journals and harness transcripts preserve history.

The directory description has at most 60 characters. Each unique `#n` in the
overview identifies the directory beginning with `<n>-`.

## Launch overview

`workbench/jurnal.md` uses this shape, with at most 256 characters per entry,
including its identifier and whitespace. Keep session references in the
launch journal.

```text
#1
  Selected-message memory survives eviction from KV.
  Exact recall: 18/20; failures recorded in result.
#2
  A larger cartridge preserves the remaining associations.
  [in curs]
```

## Launch journal

Each launch's `jurnal.md` holds at most 256 lines of at most 80 characters.
Record the attempted change and observed effects; put outcome details under
`result`. A harness session-file reference belongs here. Preserve the
harness-generated full transcript as the third level of evidence.

```md
# Hypothesis
A larger cartridge preserves the remaining associations.

# Attempt
Configuration, command and references to code or artifacts used.

# result
Measurements, failures, unexpected effects and remaining uncertainty.

# Session
Reference to the harness-generated session file.
```

Optional collapsed details count toward the same limits. Reference larger
artifacts from the journal.

## Checks

| Check | Filler | Direct |
|---|---|---|
| Target | Make the system better. | Exact recall >= 95% after KV eviction. |
| Overview | We performed an interesting experiment. | 18/20 exact; two codes lost. |
| Evidence | It seems successful. | result records 18/20 and the output file. |
| Whiteboard | This idea must stay because we tried it. | Adjust or withdraw the idea using #1. |
| Trajectory | Keep working and report at the end. | Report at the checkpoint; principal redirects the next step. |
| Continuity | Start another run after an interruption. | Inspect the effects of the [in curs] run. |

## Flow

```lnl
START:
  DO    read workspace/target.md, workspace/whiteboard.md and workbench/jurnal.md where present
  ASK   are the exercise and measurable success criterion defined?
  STOP  ask for the missing target definition IF no
  DO    create missing workspace and workbench documents using the agreed target
  ASK   is another main agent supervising this workbench?
  STOP  report the active owner and leave execution to that agent IF yes
  ASK   does the overview contain an [in curs] launch?
  JUMP  RECOVER IF yes
DECIDE:
  ASK   do saved measurements meet the target?
  JUMP  REVIEW IF yes
  ASK   is further work blocked or beyond the user's agreed budget or scope?
  STOP  report the concrete blocker or exhausted budget IF yes
  DO    read relevant launch journals and referenced evidence to determine what is already known
  DO    choose the next hypothesis and attempt using the target, evidence and whiteboard
  DO    allocate the next unused launch number above all existing entries and directories
  DO    create workbench/<n>-<description>/jurnal.md with the hypothesis and intended attempt
  DO    append the overview entry with [in curs] before execution
  DO    assign the attempt to one subagent with the shared overview, hypothesis and bounded checkpoints
  DO    supervise execution through the two-step trajectory correction cycle until completion, stop or blocker
  JUMP  RECORD
RECOVER:
  DO    inspect the interrupted launch journal, harness transcript and surviving effects
  DO    establish whether the previous worker or its processes are still active before resuming
  DO    resume that attempt under the two-step supervision cycle if justified or record its partial result and interruption
RECORD:
  DO    write observed effects, trajectory corrections, stops and evidence references under result in the launch journal
  DO    reference available main and subagent harness transcripts, recording any unavailable transcript
  DO    replace [in curs] in the overview with the measured result or interruption
  DO    adjust workspace/whiteboard.md as warranted, including withdrawing unsupported ideas
  JUMP  REVIEW
REVIEW:
  CHECK trajectory corrections and stops are recorded with their observed effects
  CHECK the reported outcome is supported by saved observations against workspace/target.md
  CHECK each overview entry fits 256 characters and resolves to one launch directory
  CHECK each launch journal fits 256 lines of 80 characters and references available evidence
  CHECK the whiteboard reflects current working ideas and launch history remains accessible
REPLY:
  SAY   the measured outcome, relevant launch numbers and any concrete blocker
  END
```
