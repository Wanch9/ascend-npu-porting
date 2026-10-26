# GLM coding-agent compatibility

Use this reference when GLM is the model executing the port. The NPU workflow
and evidence gates do not change; this file only makes the agent-host contract
explicit.

For a requested independent-execution experiment, also read
[agent-validation.md](agent-validation.md). Its fresh replay and intervention
record distinguish an independently completed port from an assisted development
run. Host compatibility has to be tested on the intended deployment host.

When the user authorizes reuse of earlier MiniMax-H3 experience, also read
[MiniMax-H3 lessons](models/minimax-h3.md). Record that input in the experiment
contract and map each reused lesson to current source and tests. This permission
does not turn an earlier model's results into current acceptance evidence.

## Distinguish the model from the host

GLM can run behind Claude Code, OpenCode, OpenClaw, Cline, Roo Code, Kilo Code,
Cursor, and other coding tools. Z.AI documents these as separate integrations,
and its official [GLM skills repository](https://github.com/zai-org/GLM-skills)
uses a directory containing `SKILL.md`. Therefore never infer file, shell, Git,
SSH, skill-discovery, or persistence capabilities from the word “GLM.” Inspect
the host first.

Official starting points:

- [Z.AI Coding Plan quick start](https://docs.z.ai/devpack/quick-start)
- [Z.AI coding-tool integrations](https://docs.z.ai/devpack/tool/others)
- [Official GLM skills repository](https://github.com/zai-org/GLM-skills)

## Host capability gate

Before delegating a port, prove that the host can:

1. read the complete toolkit directory, not just one pasted prompt;
2. recursively inspect the project and preserve exact line-level edits;
3. execute shell commands with an explicit working directory and return the
   exit code plus untruncated stdout/stderr;
4. inspect Git revisions and dirty state without resetting user changes;
5. keep a file-backed run ledger across context compaction or restarts;
6. reach the Ascend target for direct mode, or create and validate artifacts
   for handoff mode.

If 1–4 are absent, GLM can review text but cannot perform a reliable code port.
If only 6 is absent, use the offline handoff protocol and stop at **prepared**.
API-only GLM is not a coding agent until an orchestrator supplies these tools.

Skill auto-discovery is host-specific. Hosts compatible with directory skills
may discover `SKILL.md` after the whole folder is installed, but still verify
that discovery. For all other hosts, give GLM the toolkit's absolute path and
explicitly instruct it to read `SKILL.md`, this file, and every routed reference
in full. Do not translate the workflow into host-only tool names or assume MCP,
plugins, or Codex metadata.

## Portable shell and diagnostic output

Check the controller's actual shell. A Linux target does not make the local
shell Bash. Put multi-line orchestration in a script and execute it explicitly
with Bash after `bash -n SCRIPT`. For remote execution, use the bundled
`scripts/ssh_script.py` following [ssh-execution.md](ssh-execution.md); it checks
syntax and sends the script through stdin with argument-array SSH invocation.
When Python generates a script, create its source file through the host's
file-editing capability (for example, `apply_patch` when available), then run
that file instead of nesting another heredoc inside a shell string.
Use argument arrays for dynamic values and nested SSH. Avoid reserved
variable names such as zsh's `status`; use `gate_exit` or `probe_exit`. Bash
`PIPESTATUS` is not portable to zsh. Capture command output to a unique log and
save its exit code before any subsequent command changes it. Keep failed
attempts rather than overwriting their logs on retry.

An explicit Bash interpreter does not make controller utilities GNU versions.
For portable file inventories, use Python `pathlib` or probe supported options;
macOS `find` does not support GNU `-printf`. Under `pipefail`, an early-exiting
consumer such as `grep -q` or `head` can give the producer SIGPIPE and turn a
successful match into a failed pipeline. Capture the producer output and exit
status first, then inspect the completed file for the expected gate.

Read one document or a bounded line range per tool call, sized to the host's
output limit. If a result is truncated, read the missing ranges; do not keep
concatenating all references into another truncated result.
For a known artifact, read its named manifest or evidence record first. Avoid
recursive content searches through raw event logs, downloads, and archives;
one matching JSONL record can return an entire earlier tool output. Locate
candidate filenames, then select the required JSON fields or bounded log range.

Diagnostics must not collect credentials incidentally. For broad process
inventory use fields such as `pid,ppid,user,stat,etime,comm`; omit `args` and
`command`. Jupyter and other services can carry authentication tokens in their
command line. Do not dump `env`, `/proc/*/environ`, credential files, or full
command lines. Query only named non-secret topology variables. If a specific
owned process needs more detail, filter sensitive fields before storing or
returning its output, not after sending the log to the model. Public evidence
must be reviewed and redacted independently of its validity/hash checks.

Probe the type of metadata paths before using them: a rank-table environment
variable may point to a directory rather than a JSON file. Inspect the parsed
JSON root and the producing code before assuming a mapping, list, or array;
validate the actual schema instead of guessing wrapper keys. Check data API
return types before array arithmetic, and preserve the failed validator's log
when reusing an already-produced artifact. Treat missing optional discovery
inputs as such; mandatory identity/runtime failures must not be hidden by a
final echo or an unrelated command that exits zero.
Parse manifests according to their schema: distinguish entries from documented
comments and headers, reject malformed entries, and compare entry counts and
file identities. Physical line count is not an artifact count.

## Keep long work resumable

GLM must store progress in the run directory rather than relying on chat
memory. Maintain a small ledger containing:

- goal and strongest proven readiness level;
- source graph, immutable revisions, and dirty states;
- runtime/topology and local artifact availability;
- executed commands, exit codes, positive gates, and artifact hashes;
- changed files and why each change exists;
- current failure, next smallest gate, and rollback.

Update the ledger after each completed logical gate and before a long-running
operation. For an active operation, record the owned process/session identity,
destination, log, and completion check before waiting. Keep full logs on disk;
return the exit status, gate, and relevant error excerpt to the model. Batch
independent read-only probes, and restrict source scans to the executed path.
These reduce repeated context without weakening the actual tests.

After any context reset, re-read `SKILL.md`, the routed references, and this
ledger before acting. Re-probe mutable target state such as free devices,
processes, ports, IP addresses, and local paths. Do not repeat a mutation merely
because the preceding chat is missing.

If the ledger predates the final tool calls, reconcile it with the raw events
and expected artifacts first. A reviewer-written continuation note must identify
its author and evidence; it is not a GLM-authored ledger or a passed gate. Reuse
unchanged, previously verified original assets with their verification records;
check identity/existence and rehash if they changed or integrity is uncertain.
Do not download or hash the same large files again solely because a session
ended. A prepared archive is not evidence that the target received it.

Record how source was deployed. An extracted source archive may intentionally
lack `.git`; do not run remote Git assertions unless metadata was included.
Link the controller revision and patch to a relative-path/content-hash manifest,
then verify the deployed project files against that manifest. Declare exclusions
for generated bytecode or packaging metadata explicitly; do not exclude project
files to make a mismatch disappear. The manifest generator/verifier in this
toolkit accepts a directory root and declared exclusion patterns.

When a provider rejects a request for quota, preserve the raw error and stop
the failed agent session. Distinguish account balance, resource packages,
endpoint entitlement, and transient rate limits from current evidence. An old
balance page or the absence of a subscription alone does not diagnose the
failure. Do not loop paid probes after a current zero balance is confirmed.
Continue from the saved state once access is restored; do not switch the
adapting model or buy credit without user authorization.

## Composite-source rule

Before the risk scan, trace the user's requested command to its actual sources.
Training projects commonly combine a library checkout, a launcher from another
branch, a patch or overlay, and generated artifacts. Record these as a graph:

```text
launcher@revision -> imports/patches -> library@revision
                 -> reads          -> local artifact contract
                 -> writes         -> unique run directory
```

Freeze every source node independently and preserve the composition order.
Never overwrite one branch with another to make a synthetic checkout. Scan only
the nodes and paths reachable from the requested entrypoint; repository-wide
CUDA matches are an inventory, not a to-do list.

Rebuild this graph from the pinned source revision on every new experiment.
A historical example may have split launchers across branches that a later
revision consolidates. Check current paths and imports before adding another
checkout. Re-read checkpoint, cache, and audio/token contracts when the source
changes; a previously valid cache can encode a different execution contract.

## ActionWM example

The [ActionWM case](models/actionwm.md) records the historical composite-source
audit, PAC contracts, memory constraints, and the bounded development result.
Read it when mapping a similar library/launcher split; those revisions are not
a default checkout recipe.

## Prompt template for GLM

```text
You are operating through [HOST]. First prove which file, shell, Git, context-
persistence, and target-execution capabilities this host actually exposes.
Read [TOOLKIT]/SKILL.md and [TOOLKIT]/references/glm-agent.md completely, then
follow all references routed for [OUTCOME]. Run the toolkit self-check. Map the
real command across every repository, branch, patch, and generated artifact;
freeze each revision and dirty state before editing. Keep a file-backed ledger.
Preserve CPU/CUDA behavior and the installed torch/torch_npu/CANN tuple. Treat
the static scan as a scope-limited inventory. Patch and validate one failure
category at a time through the requested gates. Never infer success from an
import, another model's NPU example, or a live PID. If target execution is
unavailable, build the manifest-verified two-round handoff and label it
prepared. If a checkpoint/input is absent locally, record its exact interface
and path as a blocker; do not start an unrequested transfer.
```

The prompt is a bootstrap, not a substitute for the toolkit files. If the host
cannot read those files completely, do not delegate an autonomous port to it.
