---
name: rocket-launcher
user-invocable: true
description: Use when running a sequential experimental workbench with shared target, whiteboard, and launch history.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---
```lnl
START:
  DO    use user-selected principal and one worker: Opus (Claude Code)/Sol (Codex)
  DO    resolve <workspace> as single lab folder
  DO    read <workspace>/target.md, <workspace>/whiteboard.md, and complete <workspace>/jurnal.md where present
  ASK   is target and success criterion defined?
  STOP  ask for missing target definition IF no
  DO    atomically acquire <workspace>/.launcher-lock with owner/session identity, reclaiming only verified inactive ownership
  ASK   was ownership acquired?
  STOP  report owner and yield IF no
  DO    create missing documents, preserving content
  DO    while execution is active, supervisor monitors worker/process liveness and progress every 15 minutes, independent of k
  DO    on suspected stall, inspect logs/state, then guide/stop worker/process, recording evidence before restart
  DO    reconcile numbered directories and overview entries, marking unfinished/orphaned launches [in curs]
DECIDE:
  ADD   1 to <launches>
  STOP  report continuation checkpoint IF <launches> >= 1000000000
  ASK   is there an [in curs] launch in <workspace>/jurnal.md?
  JUMP  RECOVER IF yes
  ASK   do saved measurements meet <workspace>/target.md?
  JUMP  REVIEW IF yes
  ASK   is work blocked or budget/scope exhausted?
  STOP  report blocker/exhaustion IF yes
  DO    read journals/evidence to determine knowns
  DO    choose next hypothesis and attempt
  DO    allocate n above all existing launch numbers
  DO    create <workspace>/<n>-<description>/jurnal.md (description max 60 chars)
  DO    record hypothesis, attempt, command, config, and code references
  DO    append three overview lines: #<n>, indented hypothesis, indented [in curs] (256 chars total, no session references)
  JUMP  EXECUTE
RECOVER:
  DO    inspect journal, transcripts, and effects to classify last step: active, completed, unexecuted, or uncertain
  ASK   is the last step active?
  JUMP  OBSERVE IF yes
  ASK   is the last step completed?
  JUMP  STEP_2 IF yes
  ASK   is the last step unexecuted and resuming justified?
  JUMP  EXECUTE IF yes
  DO    record partial effects/interruption, marking uncertain effects as blocker
  JUMP  RECORD
EXECUTE:
  DO    use one subagent, reusing worker if resuming
  DO    give subagent target, whiteboard, overview, hypothesis, and evidence
  DO    supervisor sets and records k, worker steps between trajectory checks, no further delegation
STEP_1:
  DO    reset steps since last trajectory check to zero
WORK:
  ADD   1 to <total_steps>
  STOP  report continuation checkpoint IF <total_steps> >= 1000000000
  ADD   1 to <steps>
  DO    journal intended step before dispatch
  DO    subagent executes one step, recording actions and effects
  ASK   have k steps elapsed, or has attempt completed/encountered a blocker?
  JUMP  STEP_2 IF yes
  JUMP  WORK
OBSERVE:
  DO    monitor worker/process to checkpoint or interrupt if off trajectory, without redispatching step
STEP_2:
  DO    worker reports effects/proposed next action, waits for supervisor direction
  DO    main agent inspects progress, compares evidence to target/hypothesis
  DO    continue aligned work, redirect deviations or interrupt, adjusting k when useful
  DO    journal observed effects, corrections, and step completion before dispatching
  ASK   is the attempt complete, stopped, or blocked?
  JUMP  RECORD IF yes
  DO    reuse waiting worker or confirm previous worker/processes stopped before replacement
  JUMP  STEP_1
RECORD:
  DO    record effects, measurements, failures, and corrections under result in launch journal
  DO    reference full harness transcripts for main/subagent, noting unavailable ones
  DO    keep journals within 256 lines of 80 chars including collapsed details, linking larger artifacts
  DO    confirm worker and processes stopped before closing launch
  DO    update overview: replace [in curs] with result/interruption (max 256 chars)
  DO    add/adjust/withdraw ideas in <workspace>/whiteboard.md
  JUMP  DECIDE
REVIEW:
  ASK   does this invocation own the workspace lock?
  JUMP  CLEANUP IF yes
  JUMP  CHECKS
CLEANUP:
  DO    stop remaining worker/processes and save unfinished effects/transcript references with [in curs]
  DO    release owned lock only after execution stopped and records saved
CHECKS:
  CHECK saved measurements support outcome against <workspace>/target.md
  CHECK overview entries follow #<n>, fit 256 chars, and identify one <n>- directory
  CHECK directory descriptions fit 60 chars and journals fit 256 lines of 80 chars
  CHECK result records effects/corrections with evidence/transcript references
  CHECK main agent supervised one subagent at a time
  CHECK whiteboard reflects current ideas and history is accessible
REPLY:
  SAY   measured outcome, relevant launch numbers, and any concrete blocker
  END
```