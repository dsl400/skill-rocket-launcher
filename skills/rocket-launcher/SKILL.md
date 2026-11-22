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
steer.md                     user directions for coordonator
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
The coordonator runs the flow. Each role has a separate context: `<role>`,
`<base>`, current item, step state, and checkpoint interval `k`. Bind `<base>`
to `<workspace>` for worker and `<workspace>/research` for cercetaș. Keep
these contexts separate when switching between subagent events. All shared
lifecycle actions address only the selected context; global cleanup addresses
both. For an article, the hypothesis is the research question and the attempt
is reading and assessing that source.
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

## User steering

The user writes free-form directions in `<workspace>/steer.md`. Create it
empty if missing; preserve user content. The coordonator reads it at startup,
while waiting, and after each completed step, before dispatching further work,
regardless of `k`. Missing or empty content adds no direction.

Apply changed instructions when the affected role's current step finishes;
leave in-flight steps running. An idle role can receive the direction
immediately. Route research and experiment changes through the existing
whiteboards and handoff. If the current attempt no longer fits, close it with
its partial results before selecting another. Record the applied instruction
and its effect in the existing journal so recovery can distinguish applied
instructions from pending ones. Keep the direction in force until the user
changes it; unchanged text does not trigger repeated actions. The coordonator
leaves `steer.md` for the user to edit.

Example `steer.md`:

```text
After the current step, prioritize long inputs. Ask cercetaș to check
whether other teams measured this case before scheduling more experiments.
```

## Handoff checks

| Check | Filler | Direct |
|---|---|---|
| Research supports the decision. | This direction seems exhausted. | Article #4 tested these conditions and found no improvement; reject this direction. |
| The worker receives a testable gap. | Investigate article #4. | Article #4 covers short inputs; measure the same method on our long inputs. |
| Whiteboards carry role assignments. | Everyone follows the shared ideas. | Move the accepted long-input experiment from research/whiteboard.md to whiteboard.md. |
| Saved work informs the next attempt. | Research the method again. | Reuse article #4; test the input length absent from its results. |
| Lifecycle actions address the selected role. | Stop the worker after an article. | Close the cercetaș article in research/; the worker continues. |
| User steering takes effect between steps. | Wait for the next k-step review. | Read steer.md after this step and apply the direction before dispatching another. |
| Owned resources are cleaned up on exit. | Stop when the budget expires. | Stop both roles, save unfinished work, and release the owned lock. |

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
  DO    create missing documents in both workspaces, preserving content; create <workspace>/steer.md empty only if missing
  DO    make target and whiteboard self-contained: include required facts and setup; keep historical handovers and other-project references in archives
  DO    use relative filesystem paths with an explicit base directory in workspace documents and commands
  DO    while execution is active, coordonator monitors both subagents/processes for liveness and progress every 15 minutes, independent of k
  DO    on suspected stall, inspect logs/state, then guide/stop the affected role/process, recording evidence before restart
  DO    reconcile numbered directories and overview entries in both workspaces, marking unfinished/orphaned launches [in curs]
  DO    initialize separate role contexts and shared cycle/step counters; apply the supplied budget to both roles
DECIDE:
  ADD   1 to <cycles>
  STOP  run CLEANUP, save and report continuation checkpoint IF <cycles> >= 1000000000
  DO    read <workspace>/steer.md and reconcile it with applied instructions in journals; apply changes for idle roles and hold changes for active roles until their current step ends
  ASK   do saved measurements meet <workspace>/target.md?
  JUMP  REVIEW IF yes
  ASK   is work globally blocked, budget/scope exhausted, or shutdown requested?
  JUMP  HALT IF yes
  DO    read journals/evidence and review cercetaș findings at its checkpoints; move actionable directions to the worker whiteboard and delete rejected directions from the cercetaș whiteboard
  DO    propose research directions on research/whiteboard.md using saved evidence
  DO    select a role with a ready checkpoint, recoverable item, or available direction; service both roles fairly and leave dispatched steps running in parallel
  ASK   is a role ready to service?
  JUMP  WAIT IF no
  DO    bind <role> and <base> to the selected role context
  ASK   does this role have an [in curs] item?
  JUMP  RECOVER IF yes
  ASK   is <role> cercetaș?
  JUMP  ARTICLE IF yes
  DO    choose hypothesis and attempt from the worker whiteboard, reusing findings and identifying the remaining question
  DO    package the exercise with inputs, ordered steps, configuration, dependencies, expected results or rubric, and reproduction command inside the workspace
  DO    keep evaluation references separate from inputs visible to the tested system
  JUMP  ALLOCATE
ARTICLE:
  DO    have cercetaș search online and select an article for a direction on its whiteboard, using completed experiments and saved research while coordonator supervises both roles
  DO    use the research question as hypothesis and assessment of this source as attempt
  ASK   does this article already have a folder?
  JUMP  REVISIT IF yes
ALLOCATE:
  DO    allocate n above all existing numbers in <base>
  DO    create <base>/<n>-<description>/jurnal.md (description max 60 chars)
  DO    record hypothesis, attempt, command, config, and code or source references
  DO    append three overview lines in <base>/jurnal.md: #<n>, indented hypothesis, indented [in curs] (256 chars total, no session references)
  JUMP  EXECUTE
REVISIT:
  DO    select the existing article folder and mark its overview entry [in curs], preserving prior findings
  JUMP  EXECUTE
WAIT:
  DO    monitor both roles and steer.md until a checkpoint, steering change, available work, or shutdown condition; an empty worker whiteboard while research progresses means waiting
  JUMP  DECIDE
RECOVER:
  DO    inspect the selected role journal, transcripts, and effects to classify last step: active, completed, unexecuted, or uncertain
  ASK   is the last step active?
  JUMP  OBSERVE IF yes
  ASK   is the last step completed?
  JUMP  STEP_2 IF yes
  ASK   is the last step unexecuted and resuming justified?
  JUMP  EXECUTE IF yes
  DO    record partial effects/interruption, marking uncertain effects as blocker
  JUMP  RECORD
EXECUTE:
  DO    reuse the selected <role> subagent; before replacement confirm its previous subagent and processes stopped
  DO    give <role> its target, whiteboard, overview, hypothesis, and evidence; worker receives accepted research findings and the remaining question
  DO    coordonator sets and records k for <role>, with no further delegation
STEP_1:
  DO    reset selected role steps since last trajectory check to zero
WORK:
  DO    read <workspace>/steer.md and apply pending user directions for the selected role before dispatch, recording their effect in its existing journal
  ASK   does the selected attempt need closing because of user steering?
  JUMP  RECORD IF yes
  ADD   1 to <total_steps>
  STOP  run CLEANUP, save and report continuation checkpoint IF <total_steps> >= 1000000000
  ASK   is work globally blocked, budget/scope exhausted, or shutdown requested?
  JUMP  HALT IF yes
  ADD   1 to <steps>
  DO    journal intended step before dispatch
  DO    dispatch one step to <role>, recording actions and effects in its context
  JUMP  DECIDE
OBSERVE:
  DO    inspect selected role liveness/progress or interrupt if off trajectory, preserving the active step without redispatch
  JUMP  DECIDE
STEP_2:
  DO    selected role reports effects/proposed next action and waits for coordonator direction
  DO    save observed effects and step completion in the selected context before further dispatch
  DO    read <workspace>/steer.md and apply pending user directions for this role, recording their effect
  ASK   does the selected attempt need closing because of user steering?
  JUMP  RECORD IF yes
  ASK   have k steps elapsed for this role, or has its attempt completed/encountered a blocker?
  JUMP  WORK IF no
  DO    coordonator inspects progress, compares evidence to target/hypothesis, and reviews cercetaș findings for whiteboard transfers or rejections
  DO    continue aligned work, redirect deviations or interrupt, adjusting k when useful
  DO    journal observed effects, corrections, and step completion before dispatching
  ASK   is the attempt complete, stopped, or blocked?
  JUMP  RECORD IF yes
  DO    reuse the selected waiting role or confirm its previous processes stopped before replacement
  JUMP  STEP_1
RECORD:
  DO    record effects, measurements, failures, and corrections under result in the selected item journal in <base>
  DO    reference full harness transcripts for main/subagent, noting unavailable ones
  DO    keep journals within 256 lines of 80 chars including collapsed details, linking larger artifacts
  DO    confirm the selected role item and its processes stopped before closing it; leave the other role running
  DO    update <base>/jurnal.md overview: replace [in curs] with result/interruption (max 256 chars)
  DO    update findings for the selected item on <base>/whiteboard.md; coordonator routes new questions to research/whiteboard.md
  JUMP  DECIDE
HALT:
  DO    record blocker, exhaustion, or shutdown reason
  JUMP  REVIEW
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
  CHECK saved measurements support the reported outcome or incomplete status against <workspace>/target.md
  CHECK overview entries follow #<n>, fit 256 chars, and identify one <n>- directory
  CHECK directory descriptions fit 60 chars and journals fit 256 lines of 80 chars
  CHECK result records effects/corrections with evidence/transcript references
  CHECK coordonator supervised at most one cercetaș and one worker, each executing sequentially without further delegation
  CHECK research has one numbered folder per article and follows the same journal rules
  CHECK cercetaș decisions cite article evidence and distinguish research gaps from search limitations
  CHECK worker directions were transferred by coordonator with findings and a testable remaining question
  CHECK rejected directions are removed from the cercetaș whiteboard and saved findings inform further work
  CHECK steer.md content is preserved and changed user directions were applied after the affected current step, before further dispatch, independent of k
  CHECK each lifecycle action used the selected role context and workspace
  CHECK all exits after lock acquisition stopped both roles and released the owned lock after saving records
  CHECK whiteboard reflects current ideas and history is accessible
REPLY:
  SAY   measured outcome, relevant launch numbers, and any concrete blocker
  END
```
