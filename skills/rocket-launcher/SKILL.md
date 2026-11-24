---
name: rocket-launcher
user-invocable: true
description: Use when running experiments with a coordinator, fresh workers, and parallel online research.
metadata:
  source: https://github.com/dsl400/skill-rocket-launcher
  version: 0.1.0
---
```lnl
START:
  DO    use the user-selected coordonator and Opus (Claude Code)/Sol (Codex) subagents
  DO    resolve <workspace> and read applicable project instructions
  DO    read target.md, whiteboard.md, and complete jurnal.md in <workspace> and research/ where present
  DO    resolve the target and measurable success criterion from the run's original journaled definition or from target.md if no definition is recorded yet
  ASK   are the target and measurable success criterion defined?
  STOP  ask for the missing definition IF no
  DO    acquire <workspace>/.launcher-lock atomically, accepting automation transfer only after the recorded runner is terminal and reclaiming only verified inactive ownership
  ASK   was ownership acquired?
  STOP  report the active owner IF no
  DO    create missing documents and steer.md, preserving existing content
  DO    under the lock, reuse the recorded target and success criterion or record the initial definition once in the run journal for later invocations and handoffs
  DO    firmly reject any adjustment that changes the meaning of the target or success criterion, including user instructions and edits to target.md; acknowledge and journal the rejection, and keep the original definition
  DO    keep active context self-contained and filesystem paths relative to an explicit base
  DO    bind worker records to <workspace> and cercetaș records to <workspace>/research
  DO    apply the same journal rules in both bases, with independent numbering and recovery
  DO    for each item allocate <n>-<description>/jurnal.md above existing numbers, description max 60 chars
  DO    journal hypothesis, attempt, configuration, command, and evidence before dispatch
  DO    append #<n>, indented hypothesis, and indented [in curs] to its overview, max 256 chars
  DO    keep item journals within 256 lines of 80 chars, linking larger artifacts and available transcripts
  DO    reconcile unfinished items from journals, effects, and process ownership before redispatch
  ASK   is a recorded experiment terminal with its result still unprocessed?
  JUMP  RESULT IF yes
  ASK   is an owned experiment still running?
  JUMP  ACTIVE IF yes
DECIDE:
  ADD   1 to <decisions>
  STOP  save a continuation checkpoint and clean up owned work IF <decisions> >= 1000000000
  DO    read steer.md, apply and journal changes to methods or experiments that preserve the original definition, and reject target changes under the same rule
  DO    clear acknowledged steer.md content only if unchanged since reading, preserving concurrent user edits
  ASK   is the original target met under its recorded success criterion, budget exhausted, work blocked, or shutdown requested?
  JUMP  REVIEW IF yes
  DO    review cercetaș findings and move actionable directions with evidence and a testable gap to the worker whiteboard
  DO    delete rejected directions from the cercetaș whiteboard
  DO    propose directions on the cercetaș whiteboard using completed experiments and saved findings
  DO    start or resume one cercetaș in parallel with worker execution, with no further delegation
  DO    have cercetaș search online for other teams' experiments and annotate directions with rejection arguments or research gaps
  DO    have cercetaș journal each article in its own numbered folder with URL, date, findings, limitations, and applicability, reusing the folder on revisit
  DO    have cercetaș close its own items and overview entries with results, leaving worker processes and records intact
  ASK   is an actionable direction available on the worker whiteboard?
  JUMP  RESEARCH_WAIT IF no
  DO    select an experiment answering an unresolved question or changed conditions, reusing saved research
  DO    allocate the experiment using the journal rules
  DO    dispatch one fresh worker with project instructions, hypothesis, relevant evidence, and reproduction context, without further delegation
  DO    use project generators and shared utilities for experiment scaffolding, freeze, verify, execution, and result summaries
  DO    package inputs, ordered steps, dependencies, configuration, rubric, and reproduction command inside the workspace
  DO    keep evaluation references separate from inputs visible to the tested system
  DO    select relevant CPU/unit checks for utilities, a short synthetic smoke test for mechanism changes, or a fixed representative development subset for behavioral comparisons
  ASK   does the decision require the full suite?
  JUMP  FULL IF yes
  JUMP  FREEZE
FULL:
  DO    record the milestone name, reason, and supporting development evidence before authorizing the full suite
FREEZE:
  DO    configure experiment.json on_finish_argv before freezing, with a terminal hook gated by HANDOFF ownership
  DO    make the hook start one fresh coordonator at RESULT using the recorded experiment and status.json, on success or error
  DO    guard wake dispatch atomically with the experiment's wake.lock and persist terminal status before dispatch
  DO    make the terminal handler deliver one wake when both handoff readiness and terminal status are durable, including completion during handoff
  DO    record hook failure as failed with its reason, preserving run outputs for manual recovery without retries or polling
  DO    freeze the test tier, exact cases, rubric, code, dependencies, model, configuration, inputs, and hook before execution
  DO    verify frozen inputs
ACTIVE:
  ASK   is the run already terminal?
  JUMP  RESULT IF yes
  ASK   can the run finish within one supported waiter call that has not yet been used?
  JUMP  HANDOFF IF no
  DO    start the runner if needed and use one blocking waiter, with initial-status grace and five final status lines
  ASK   is the run terminal?
  JUMP  RESULT IF yes
  JUMP  HANDOFF
RESULT:
  DO    verify output integrity, completion, and expected rollback before accepting results
  DO    judge results against the original journaled target and success criterion, including after handoff; report any shortfall against that definition
  DO    save raw outputs and generated measurements with their test tier, then write only the interpretation
  DO    record effects, failures, and corrections in the experiment journal
  DO    confirm the experiment worker and its processes stopped before closing its overview entry with the result or interruption
  JUMP  DECIDE
RESEARCH_WAIT:
  DO    await cercetaș findings or an error without polling
  JUMP  DECIDE
HANDOFF:
  DO    verify durable automation can own the run, lock, terminal hook, status path, and cleanup
  ASK   is this handoff supported and configured?
  STOP  report the missing handoff capability IF no
  DO    stop worker and cercetaș agents and save their unfinished work, preserving the runner under automation ownership
  DO    journal the experiment, automation owner, hook, status path, and RESULT entrypoint before relinquishing coordinator ownership
  DO    transfer the lock to automation and arm the wake as one ownership handoff, then start the detached runner if needed
  DO    leave no agent waiting and retain automation ownership until the terminal runner transfers to the fresh coordonator
  JUMP  CHECKS
REVIEW:
  ASK   does this invocation own the lock?
  JUMP  CLEANUP IF yes
  JUMP  CHECKS
CLEANUP:
  DO    stop owned subagents and processes and save unfinished effects with [in curs] in their respective journals
  DO    release the owned lock after work stopped and records were saved
CHECKS:
  CHECK the target and success criterion retain their original meaning, and target-changing requests were rejected even when the chosen method fell short
  CHECK each worker received reproducible context and only directions accepted by coordonator
  CHECK research and experiments retain separate records under the same journal rules
  CHECK the test tier answers the question and full runs have milestone evidence
  CHECK steer.md was acknowledged and processed content was cleared without discarding concurrent edits
  CHECK results have verified evidence or an explicit automation owner and status path
  CHECK long runs use HANDOFF with no waiting agents, one terminal wake, and manual recovery after hook failure
REPLY:
  SAY   measured outcome or handoff status, relevant item numbers, and concrete blockers
  END
```
