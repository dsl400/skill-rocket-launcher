---
name: rocket-launcher
user-invocable: true
description: Use when running experiments with a coordinator, parallel online researcher, and worker.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---

# rocket-launcher

## Roles and workspaces

The main agent is **coordonator**. Its subagents are **cercetaș** (online
research) and **worker** (experiments). Each subagent works sequentially;
research and experiments run in parallel, under the same coordonator.
Subagents execute their assigned role directly, without further delegation.

Paths below are relative to `<workspace>`:

```text
target.md
whiteboard.md                 worker directions
jurnal.md
1-<experiment>/jurnal.md
research/
  target.md                  research questions serving the main target
  whiteboard.md              directions proposed by coordonator
  jurnal.md                  overview of articles read
  1-<article>/jurnal.md       article details and findings
```

Apply the journal, numbering, recovery, evidence, and size rules in the flow
to both workspaces. Research uses an article as its unit of work; experiments
use an attempt. Number each workspace independently. The coordonator owns
the lab lock and supervises both roles, including during recovery and cleanup.
For an article, the hypothesis is the research question and the attempt is
reading and assessing that source. The cercetaș applies RECOVER and the
STEP_1 through RECORD lifecycle to its own work, returning to its whiteboard
for the next article. The coordonator handles dispatch and trajectory checks.
Coordinate whiteboard edits at cercetaș checkpoints.

The cercetaș reads completed experiments and existing research, searches
online for other teams' work on the proposed directions, and reads the
relevant articles. Each article journal identifies its URL, consultation
date, findings, limitations, and applicability to the proposed experiments.
Reuse an existing article folder when revisiting that article.

The cercetaș annotates directions on its whiteboard with supported rejection
arguments or research gaps worth testing, linking the article evidence.
Assess whether other teams' conditions apply to ours; describe an unsuccessful
search as a search limitation. Feed new suggestions into this same whiteboard.

Only the coordonator transfers actionable directions from the cercetaș's
whiteboard to the worker's whiteboard, carrying the relevant findings,
evidence links, and remaining question. It deletes rejected directions from
the cercetaș's whiteboard. The worker executes directions from its own
whiteboard. The coordonator consults saved findings when choosing work;
repeating research or an experiment requires a stated unresolved question
or changed conditions.

## Handoff checks

| Check | Filler | Direct |
|---|---|---|
| Research supports the decision. | This direction seems exhausted. | Article #4 tested these conditions and found no improvement; reject this direction. |
| The worker receives a testable gap. | Investigate article #4. | Article #4 covers short inputs; measure the same method on our long inputs. |
| Whiteboards carry role assignments. | Everyone follows the shared ideas. | Move the accepted long-input experiment from research/whiteboard.md to whiteboard.md. |
| Saved work informs the next attempt. | Research the method again. | Reuse article #4; test the input length absent from its results. |

## Flow

```lnl
START:
  DO    use user-selected coordonator, cercetaș, and worker: Opus (Claude Code)/Sol (Codex)
  DO    resolve <workspace> as single lab folder
  DO    read target.md, whiteboard.md, and complete jurnal.md in <workspace> and <workspace>/research where present
  ASK   is target and success criterion defined?
  STOP  ask for missing target definition IF no
  DO    atomically acquire <workspace>/.launcher-lock with owner/session identity, reclaiming only verified inactive ownership
  ASK   was ownership acquired?
  STOP  report owner and yield IF no
  DO    create missing documents in both workspaces, preserving content
  DO    make target and whiteboard self-contained: include required facts and setup; keep historical handovers and other-project references in archives
  DO    use relative filesystem paths with an explicit base directory in workspace documents and commands
  DO    while execution is active, coordonator monitors both subagents/processes for liveness and progress every 15 minutes, independent of k
  DO    on suspected stall, inspect logs/state, then guide/stop worker/process, recording evidence before restart
  DO    reconcile numbered directories and overview entries in both workspaces, marking unfinished/orphaned launches [in curs]
  DO    set and record cercetaș checkpoint interval k; resume or start one cercetaș on research/whiteboard.md, applying the same work lifecycle to each article while worker experiments proceed
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
  DO    propose research directions on <workspace>/research/whiteboard.md using saved evidence
  DO    review cercetaș findings; move actionable directions to <workspace>/whiteboard.md and delete rejected directions from research/whiteboard.md
  ASK   is a direction available on the worker whiteboard?
  JUMP  RESEARCH_WAIT IF no
  DO    choose next hypothesis and attempt from the worker whiteboard, reusing research findings and identifying the remaining question
  DO    allocate n above all existing launch numbers
  DO    create <workspace>/<n>-<description>/jurnal.md (description max 60 chars)
  DO    record hypothesis, attempt, command, config, and code references
  DO    package each exercise with inputs, ordered steps, configuration, dependencies, expected results or rubric, and reproduction command inside the workspace
  DO    keep evaluation references separate from inputs visible to the tested system
  DO    append three overview lines: #<n>, indented hypothesis, indented [in curs] (256 chars total, no session references)
  JUMP  EXECUTE
RESEARCH_WAIT:
  DO    supervise cercetaș to its next checkpoint and inspect findings or blockers
  JUMP  DECIDE
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
  DO    use one worker alongside the cercetaș, reusing worker if resuming
  DO    give worker target, its whiteboard, overview, hypothesis, and accepted research findings with evidence
  DO    coordonator sets and records k separately for each subagent: steps between trajectory checks, no further delegation
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
  DO    coordonator inspects progress, compares evidence to target/hypothesis, and reviews cercetaș findings for whiteboard transfers or rejections
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
  DO    update attempted directions in <workspace>/whiteboard.md and propose new questions on research/whiteboard.md
  JUMP  DECIDE
REVIEW:
  ASK   does this invocation own the workspace lock?
  JUMP  CLEANUP IF yes
  JUMP  CHECKS
CLEANUP:
  DO    stop both subagents and their processes and save unfinished effects/transcript references with [in curs] in their respective workspaces
  DO    release owned lock only after execution stopped and records saved
CHECKS:
  CHECK a fresh worker can run and evaluate the exercise from workspace artifacts without chat history, archived handovers, or another project
  CHECK target and whiteboard contain no pointers to archived handovers or other projects; filesystem paths are relative with explicit bases
  CHECK saved measurements support outcome against <workspace>/target.md
  CHECK overview entries follow #<n>, fit 256 chars, and identify one <n>- directory
  CHECK directory descriptions fit 60 chars and journals fit 256 lines of 80 chars
  CHECK result records effects/corrections with evidence/transcript references
  CHECK coordonator supervised at most one cercetaș and one worker, each executing sequentially without further delegation
  CHECK research has one numbered folder per article and follows the same journal rules
  CHECK cercetaș decisions cite article evidence and distinguish research gaps from search limitations
  CHECK worker directions were transferred by coordonator with findings and a testable remaining question
  CHECK rejected directions are removed from the cercetaș whiteboard and saved findings inform further work
  CHECK whiteboard reflects current ideas and history is accessible
REPLY:
  SAY   measured outcome, relevant launch numbers, and any concrete blocker
  END
```
