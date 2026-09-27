# Chartr shared memory

Shared memory preserves knowledge about an operator's work across coding-agent providers and sessions.

## Language

**Memory project**:
A stable identity for shared project knowledge across machines, folders, and coding-agent providers. Folder names and paths locate the work but do not define its identity.

**Memory boundary**:
A separation between work and personal knowledge. Knowledge crosses that separation only when the operator explicitly shares it.

**Operator-wide preference**:
A preference that applies across projects. Eligible preferences accompany project knowledge by default, subject to the work/personal memory boundary.

**Memory binding**:
The operator-selected project, work/personal boundary, and any repository or worktree context that limits a session's memory access. A change of working directory does not change this binding.

**Worktree-specific knowledge**:
Knowledge whose applicability is limited to one checkout or worktree. Other worktrees receive it only after it has been deliberately generalized.

**Free session**:
A session with no assigned memory project. It receives eligible operator-wide preferences within its selected boundary and keeps other new knowledge local to the session until assigned.

**Source attribution**:
The durable identification of the provider and source work from which knowledge originated. Attribution can survive even when the original evidence is no longer accessible.

**Private knowledge**:
Knowledge requiring explicit private access within either the work or personal memory boundary. Personal knowledge is not necessarily private.

**Memory publication**:
An operator-reviewed snapshot of knowledge shared into a destination scope. It is distinct from its source record and does not implicitly share the source's evidence or later edits.

**Source run**:
An attributable unit of work responsible for a memory write or capture. Its identity remains stable when provisional source attribution is later verified.

**Source installation**:
The registered client installation where source work originated. It distinguishes otherwise identical provider session identifiers and local paths on different clients.

**Provider**:
A coding-agent harness through which the operator works, such as Codex, Claude Code, or Grok Build. It is distinct from the underlying language model.

**Explicit memory access**:
An agent deliberately reads or records shared knowledge during its work.
_Avoid_: Manual memory when it could imply that only the human can record knowledge.

**Automatic memory capture**:
Knowledge is derived from agent work without requiring an explicit memory-write request for each item. Capture does not itself determine whether that knowledge is accepted.

**Existing memory history**:
Knowledge and its history already held in the operator's shared memory store. It is distinct from older conversation files held by individual providers.
_Avoid_: Old-session import when referring to access to already-stored memory.

**Legacy project tag**:
A `project:` scope value on an existing memory card. It identifies the card's historical project grouping but does not by itself establish a stable memory project or access boundary.

**Knowledge proposal**:
A memory claim awaiting operator acceptance or confirmation. It is excluded from ordinary memory access and is available when proposals are explicitly requested.

**Memory successor**:
A new knowledge record replacing an older record whose meaning, scope, or authority changed. The older record remains in history.

**Independent memory evidence**:
Source work that separately observes or establishes a claim, rather than repeating memory previously retrieved by an agent.
