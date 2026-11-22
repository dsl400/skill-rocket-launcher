---
name: rocket-launcher
description: Use when running a sequential experimental workbench with a shared target, editable whiteboard, and a launch history across agent sessions.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---

```lnl
START:
  DO    resolve <workspace> as the single laboratory folder in the user's project
  DO    read <workspace>/target.md, <workspace>/whiteboard.md and the complete <workspace>/jurnal.md where present
  ASK   are the exercise and measurable success criterion defined?
  STOP  ask for the missing target definition IF no
  DO    preserve existing documents and create missing ones with the agreed target in <workspace>/target.md
  ASK   is another main agent supervising this workbench?
  STOP  report the active owner and leave execution to that agent IF yes
  ASK   does <workspace>/jurnal.md contain an [in curs] launch?
  JUMP  RECOVER IF yes
DECIDE:
  ASK   do saved measurements meet <workspace>/target.md?
  JUMP  REVIEW IF yes
  ASK   is further work blocked or beyond the user's agreed budget or scope?
  STOP  report the concrete blocker or exhausted budget IF yes
  DO    read relevant launch journals and referenced evidence to determine what is already known
  DO    choose the next hypothesis and attempt using the target, evidence and whiteboard
  DO    allocate the next unused integer n above all existing overview entries and launch directories
  DO    create <workspace>/<n>-<description>/jurnal.md with a description of at most 60 characters
  DO    record the hypothesis, intended attempt, configuration, command and code references in the launch journal
  DO    append an overview entry as three lines: #<n>, two spaces followed by the hypothesis, two spaces followed by [in curs]
  DO    keep each overview entry within 256 characters including identifier and whitespace, with session references confined to the launch journal
  JUMP  EXECUTE
RECOVER:
  DO    inspect the interrupted launch journal, available harness transcripts and surviving effects
  DO    establish whether the previous worker or its processes remain active
  ASK   is resuming the interrupted attempt justified?
  JUMP  EXECUTE IF yes
  DO    stop any surviving execution and record partial effects and interruption
  JUMP  RECORD
EXECUTE:
  DO    use one executing subagent, reusing the existing worker when resuming
  DO    give the subagent the target, editable whiteboard, complete overview, launch hypothesis and relevant evidence
  DO    assign bounded steps and checkpoints, with all execution confined to that subagent and no further delegation
STEP_1:
  DO    have the subagent execute the assigned bounded step and report actions, observed effects and the intended next step
  DO    have the subagent wait for the main agent's direction at the checkpoint
STEP_2:
  DO    as the main agent, inspect available progress between checkpoints and compare reported evidence and next actions with the target and launch hypothesis
  DO    continue aligned work, redirect deviations with a specific next action, or interrupt promptly when continuing would deepen the deviation
  DO    confirm the worker and its launched processes have stopped before replacing it
  DO    retain corrections, stops and their observed effects for the launch journal
  DO    repeat the bounded execution and trajectory check until the attempt completes, is stopped or encounters a blocker
RECORD:
  DO    as the main agent, finalize observed effects, measurements, failures, uncertainty and trajectory corrections under result in the launch journal
  DO    reference available main and subagent harness-generated full transcripts, recording any unavailable transcript
  DO    keep each launch journal within 256 lines of at most 80 characters, including collapsed details, and reference larger artifacts separately
  DO    replace [in curs] in the overview with the measured result or interruption while preserving the 256-character entry limit
  DO    add, adjust or withdraw ideas in <workspace>/whiteboard.md while preserving history in launch journals and harness transcripts
  DO    repeat the flow for successive sequential launches until the target is met, work is blocked, or the agreed budget or scope is exhausted
REVIEW:
  CHECK the reported outcome is supported by saved observations against <workspace>/target.md
  CHECK each overview entry follows #<n> with indented hypothesis and result, fits 256 characters and identifies exactly one <n>- directory
  CHECK each directory description fits 60 characters and its journal fits 256 lines of 80 characters
  CHECK result records effects and corrections with available evidence and harness transcript references
  CHECK the main agent supervised one executing subagent at a time
  CHECK the whiteboard reflects current working ideas and launch history remains accessible
REPLY:
  SAY   the measured outcome, relevant launch numbers and any concrete blocker
  END
```
