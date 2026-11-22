---
name: rocket-launcher
description: Use when running a sequential experimental workbench with a shared target, editable whiteboard, and a launch history across agent sessions.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---

# Rocket launcher

One agent runs at a time. Each agent starts with the target, whiteboard and
complete launch overview, then opens launch details as needed.
Repeat the flow for successive launches until the target is met, work is
blocked, or the agreed budget or scope is exhausted.

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
| Continuity | Start another run after an interruption. | Inspect the effects of the [in curs] run. |

## Flow

```lnl
START:
  DO    read workspace/target.md, workspace/whiteboard.md and workbench/jurnal.md where present
  ASK   are the exercise and measurable success criterion defined?
  STOP  ask for the missing target definition IF no
  DO    create missing workspace and workbench documents using the agreed target
  ASK   is another agent running this workbench?
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
  DO    execute the attempt as the sole active agent
  JUMP  RECORD
RECOVER:
  DO    inspect the interrupted launch journal, harness transcript and surviving effects
  DO    resume that attempt if justified or record its partial result and interruption
RECORD:
  DO    write observed effects and evidence references under result in the launch journal
  DO    reference the full harness session transcript when its location is available, otherwise record that it is unavailable
  DO    replace [in curs] in the overview with the measured result or interruption
  DO    adjust workspace/whiteboard.md as warranted, including withdrawing unsupported ideas
  JUMP  REVIEW
REVIEW:
  CHECK the reported outcome is supported by saved observations against workspace/target.md
  CHECK each overview entry fits 256 characters and resolves to one launch directory
  CHECK each launch journal fits 256 lines of 80 characters and references available evidence
  CHECK the whiteboard reflects current working ideas and launch history remains accessible
REPLY:
  SAY   the measured outcome, relevant launch numbers and any concrete blocker
  END
```
