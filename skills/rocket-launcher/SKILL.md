---
name: rocket-launcher
description: Use when running a sequential experimental workbench with shared target, whiteboard, and launch history.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---

```lnl
START:
  DO    use Fable or Astra as principal, with Opus or Sol as subagent according to the harness
  DO    resolve <workspace> as the single laboratory folder
  DO    read <workspace>/target.md, <workspace>/whiteboard.md and complete <workspace>/jurnal.md where present
  ASK   is the target and success criterion defined?
  STOP  ask for missing target definition IF no
  DO    create missing documents, preserving existing content
  ASK   is another agent supervising this workbench?
  STOP  report owner and yield IF yes
  ASK   is there an [in curs] launch in <workspace>/jurnal.md?
  JUMP  RECOVER IF yes
DECIDE:
  ASK   do saved measurements meet <workspace>/target.md?
  JUMP  REVIEW IF yes
  ASK   is work blocked or budget/scope exhausted?
  STOP  report blocker/exhaustion IF yes
  DO    read journals and evidence to determine knowns
  DO    choose next hypothesis and attempt
  DO    allocate n above all existing launch numbers
  DO    create <workspace>/<n>-<description>/jurnal.md (description max 60 chars)
  DO    record hypothesis, attempt, command, config and code references
  DO    append three overview lines: #<n>, indented hypothesis, indented [in curs] (256 chars total, no session references)
  JUMP  EXECUTE
RECOVER:
  DO    inspect interrupted journal, transcripts, and effects
  DO    check whether previous worker or its processes remain active
  ASK   is resuming the interrupted attempt justified?
  JUMP  EXECUTE IF yes
  DO    stop surviving execution and record partial effects
  JUMP  RECORD
EXECUTE:
  DO    use one subagent, reusing worker if resuming
  DO    give subagent target, whiteboard, overview, hypothesis, and evidence
  DO    assign bounded steps and checkpoints, without further delegation
STEP_1:
  DO    subagent executes bounded step, reports actions, effects, and next step
  DO    subagent waits for main agent direction at checkpoint
STEP_2:
  DO    main agent inspects progress, compares evidence to target/hypothesis
  DO    continue aligned work, redirect deviations, or interrupt
  DO    confirm worker and its processes stopped before replacement
  DO    retain corrections and stops in launch journal
  DO    repeat bounded execution and trajectory check until attempt completes, stops, or blocks
RECORD:
  DO    record effects, measurements, failures and corrections under result in launch journal
  DO    reference full harness transcripts for main/subagent, noting unavailable ones
  DO    keep journals within 256 lines of 80 chars including collapsed details, linking larger artifacts
  DO    update overview: replace [in curs] with result/interruption (max 256 chars)
  DO    add/adjust/withdraw ideas in <workspace>/whiteboard.md
  DO    repeat sequential launches until target met, blocked, or budget exhausted
REVIEW:
  CHECK saved measurements support the reported outcome against <workspace>/target.md
  CHECK overview entries follow #<n>, fit 256 chars, and identify one <n>- directory
  CHECK directory descriptions fit 60 chars and journals fit 256 lines of 80 chars
  CHECK result records effects/corrections with evidence/transcript references
  CHECK main agent supervised one subagent at a time
  CHECK whiteboard reflects current ideas and history is accessible
REPLY:
  SAY   measured outcome, relevant launch numbers, and any concrete blocker
  END
```
