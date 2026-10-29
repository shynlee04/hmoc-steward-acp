# Glossary

Canonical terms for hivemind-steward-oc-devin. Resolve a term here before
coining a synonym. One context, grouped by cluster; split into a map with
node glossaries only when one term must mean two different things that
cannot be renamed, or the code splits into separately owned packages.

## Harness and wire

Shared by every Harness and both lifecycles.

**ACP agent** — software that accepts Agent Client Protocol requests and
drives agent turns, including its own tool execution and session history.
Distinct from a model provider, which supplies model calls.
_Avoid_: "ACP model" (confuses an agent with the model it uses).

**Harness** — one ACP agent product this plugin can drive (Devin,
Antigravity), identified by its native provider id `<id>-native`. A Native
child belongs to exactly one Harness for its whole life.
_Avoid_: "platform" or "backend" in code and specs.

**Harness Profile** — the per-Harness data and adapters the shared code
reads: identity, launch, Wire dialect, delegated mode, thinking facet
presence, observation identity and copy. Behavior shared by every Harness
is not part of a profile.
_Avoid_: "agent profile" (collides with Role agent).

**Harness Registry** — the single resolution seam that maps the registered
Harness Profiles plus the Delegate config document, `ctx.options` and
detected launch facts into a per-call Registry snapshot. A pure
projection, not a live service: it owns no session or engine state and
holds no Harness id literals of its own.
_Avoid_: "registry service" (nothing registers at runtime; profiles are
static code), "router" alone (`route` is one pure function over the
snapshot, not the store behind it).

**Registry snapshot** — the frozen per-call projection the Harness
Registry produces: one Harness view per registered profile, frozen at
call start and read by the router, the admission ledger, setup status and
the doctor check.
_Avoid_: "config" alone (the snapshot also carries launch and sign-in
facts, each field stamped with its winning source).

**Harness view** — one profile's resolved row inside a Registry snapshot:
enabled, rank, launch outcome, ranked model slots, thinking, per-mode
limits and sign-in state — every field stamped with its winning source
(`options`, `doc`, `doc-legacy` or `default`).
_Avoid_: "resolved profile" (the view is a projection, not the profile
itself).

**Wire dialect** — the adapter that turns one Harness's vendor `_meta`
keys, extension notifications and error data into typed observations and a
Failure class. Standard ACP fields are not dialect.

**Failure class** — the closed category an ACP error carries across the
wire boundary: `auth`, `quota`, `refusal`, `invalid`, `retryable` or
`unknown`. Decisions read the class, never a vendor error string. The
tool-result carrier reuses the term off this enum: DSH-17's
`ForeignToolFacts.failureClass` holds the single literal `capacity` until
the held DSH-16 lane merges and reconciles one vocabulary.

**Engine** — one Harness's ACP server subprocess (for Devin, `devin acp`)
plus its JSON-RPC client, as seen through the engine interface in
`src/engine-port.ts`. A lane's engine outlives the individual session bindings
made on it.
_Avoid_: "connection" (that is the wire, not the owned thing), "client",
"backend".

**Trajectory** — the recorded context and ordered events of an agent
conversation. A harness may call the conversation a session or use another
name.
_Avoid_: "transcript" when context and non-message events are also meant.

**Native trajectory** — a Trajectory recorded in a Harness's own session or
conversation storage and presented through its own native interface (such as
Devin's web/desktop app or CLI). Distinct from an OpenCode **Native root
session**, a **Native child**, a plugin's render history, or an exported
artifact.
_Avoid_: "main session" or "root session" when the intended meaning is the
foreign Harness's own storage and interface rather than a parentless OpenCode
session (**Native root session**) or a parent-child relationship (**Native
child**).

**Active writer** — the client currently advancing a Trajectory through agent
turns. Recording or catching up a native history does not itself constitute
an independent conversation writer.
_Avoid_: "database writer" (describes storage operations, not turn ownership).

**Foreign session defaults** — the model, thinking (when the Harness has a Thinking facet), and
mode values a foreign session advertises as its own in the `session/new` /
`session/load` catalog (`configOptions`, `modes`, `config_option_update`).
_Avoid_: "native default model" (the *value* is foreign-reported, not
hardcoded), "agent config" (this is the wire catalog, not a file).

**Config Option** — one configurable facet of a foreign session as its Harness
reports it via `config_option_update` or the new/load response (`model`,
`mode`, and `thought_level` where present): an id, a name, a choice list, and
a current value. The hosted pickers render only these rows.
_Avoid_: "setting", "preference" (too generic), "picker" (that is the Overlay
rendering the choices).

**Continuation** — reopening a foreign session that exists in its Harness's store
regardless of which client created it, via `session/load`.
_Avoid_: "resume" alone when the distinction between lane-remount and
cross-client reload matters; say "lane remount" for the TUI navigation case, and Child continuation for
another turn on a Native child.

## Native delegation

An OpenCode parent hands work to a Harness through genuine host children.

**Delegation** — handing work from an OpenCode parent to a Harness. Native
delegation uses a delegate tool (today the Pinned alias `delegate-devin`) and
genuine host children; `devin_session` remains its existing non-delegation
Tool lane with a separate ACP lifecycle (`devin_run` and the duplicate
`devin_doctor` retired 2026-10-22; `hivemind-doctor` is the sole doctor).
Distinct from the hosted route
and ordinary builtin `subagent` selection.
_Avoid_: "handoff" (reserved for document handoffs between agent sessions).

**Native child** — an OpenCode child session carrying a `parentID` and
presented through the host's own viewer and job controls (whether created by
builtin `subagent`, a delegate tool, or a host command subtask). Distinct from
a parentless **Native root session** and from the plugin's hosted **Lane** and
**Tool lane**, which are separate lifecycles; child versus root placement is a
host session identity and topology distinction, not a permission grant.
_Avoid_: "subagent session" (collides with in-stream subagent entries),
"child lane", "root session".

**Native root session (parentless OpenCode session)** — a genuine parentless
OpenCode session (`parentID` absent, covering both top-level main sessions and
host-created forks) that selects an observed `<id>-native/<model>[#variant]`
model reference and runs in the host's primary session surface rather than as a
**Native child** or **Hosted session**. Root versus child placement is a host
session identity and topology distinction only, not a permission or mode grant,
and is distinct from the foreign Harness's **Native trajectory**; its persisted
association is a **Root binding** at `root-bindings/<sessionID>`, distinct from
any child-only **Child binding**.
_Avoid_: "native trajectory" or "foreign main session" (those refer to the
Harness's own session storage and UI), "hidden child", "hosted session".

**Child continuation** — another turn on an existing Native child, made by
calling a delegate tool with the child's native `sessionID`. It stays on the
child's Harness and Foreign trajectory. Distinct from Continuation, which
reopens a foreign session via `session/load`.
_Avoid_: "resume" alone, "continuation" unqualified when both are in scope.

**Native delegation driver** — the candidate module satisfying the host's
provider/model contract for native children while driving whole ACP agent
turns; owns the child-to-foreign binding, control mediation, and
foreign-event translation. Never a stateless model-completion shim.
_Avoid_: "ACP model" (confuses an agent with a model), "adapter" alone.

**Provider foundation** — the planned single ACP execution owner behind one public registration entry point, serving parentless main sessions and builtin subagent children through the same path after host identity and topology validation. Owns launch, association, delivery, recovery, confirmation and translation. Never a second delegation controller, copied policy evaluator, or per-Harness orchestration copy.
_Avoid_: "delegation driver" for the target shape (that name stays with the current child-only module until migration lands).

**Compatibility facade** — a public delegate alias retained after migration as selection and forwarding only, without its own session lifecycle or policy evaluation.
_Avoid_: "wrapper" (implies the old claim-coupled ownership).

**Admission unit** — the host primary model request admitted per request, replacing the delegate call as the unit of admission. Carries resolved identity, selection origin, consumed stamp and frozen policy context.
_Avoid_: "claim" (that name stays with the legacy wrapper mechanism until migration lands).

**Role agent** — the caller-selected OpenCode agent role for a native session
(**Native child** or **Native root session**), including its host identity,
instructions, eligibility, and permission rules. Distinct from the
**Execution-driver identity**, from the foreign **ACP agent**, and from the
**Hosted Lane** lifecycle, which never inherits a native **Role agent**.
_Avoid_: "Devin agent" or "Antigravity agent" when referring to the host role,
"hosted role" (hosted sessions have no **Role agent**).

**Execution-driver identity** — the private `<id>-native/nd-delegate` sentinel
reference (such as `devin-native/nd-delegate`) selecting a Harness's **Native
delegation driver** as internal execution-route plumbing when a new delegation
omits an explicit model. Distinct from observed vendor
`<id>-native/<model>[#variant]` model references, **Explicit model
selection**, **Creation policy**, and **Effective foreign settings**; it
identifies the transport route only, never an advertised vendor model, caller
selection origin, or foreign serving setting.
_Avoid_: "foreign model" or "default vendor model" for the driver sentinel,
"public catalog model".

**Explicit model selection** — an observed `<id>-native/<model>[#variant]`
model reference and optional `#variant` effort suffix supplied by the caller,
distinct from an omitted selection, the private `nd-delegate`
**Execution-driver identity** sentinel, a **Model slot chain** default, or a
role/parent fallback. It expresses a caller request, not **Creation policy**
admission or **Effective foreign settings**, and a resolved model ID alone
does not disclose whether the choice came from an explicit caller argument, a
role, or a command.
_Avoid_: "effective model" for an unfulfilled caller selection, "driver
sentinel", "selection origin" (a resolved model ID does not prove how it was
chosen).

**Native tool contract** — the host tool's input/output schemas, annotations,
options, validation and execution behavior, including controls and its complete
result envelope. Distinct from the tool's Outer tool card.
_Avoid_: "native card parity" when describing schema or execution behavior.

**Outer tool card** — the host's visual presentation of a tool invocation in
the parent's history. Distinct from the Native child's viewer and the Native
tool contract behind the invocation.
_Avoid_: "native child viewer" for the parent's invocation card.

**Pinned alias** — a delegate tool fixed to one Harness that shares the
general delegate tool's Native tool contract (`delegate-devin`).
_Avoid_: "wrapper" (it adds no behavior of its own).

**Harness rank** — the user's ordered list of Harnesses that picks the
Harness for a delegation that names no model.
_Avoid_: "priority", "fallback list" (a supplied model is never fallen over).

**Preflight** — the checks run over the Harness rank before any Native
child exists; the only point where a delegation may move to another Harness.
_Avoid_: "retry", "fallback" (nothing is created or resent).

**Recorded sign-in state** — the plugin's last known signed-in or
signed-out outcome for a Harness whose sign-in the plugin drives. Evidence
for Preflight, not proof of a live credential.
_Avoid_: "auth status" (implies a live check).

**Auth generation (`authEpoch`)** — an opaque, monotonically increasing
non-negative integer recorded alongside a Harness's Recorded sign-in state
whenever authentication changes (user-consented sign-in, local sign-out, or an
observed `auth` Failure class). It carries no account email, credential
fingerprint, or vendor token, and marks engines or cached catalog hints from
an older generation as stale.
_Avoid_: "account id", "auth token", "session epoch".

**ACP binary (`acpBinary`) vs. CLI companion (`cliCompanion`)** — the two
independent executable artifacts a Harness may distribute: `acpBinary` is the
stdio ACP server binary the Engine spawns, whereas `cliCompanion` is the
user-facing CLI binary invoked in a terminal. Each carries its own path and
version provenance; a CLI companion is never substituted for an ACP binary.
_Avoid_: "agent binary" alone when both artifacts exist.

**Readiness tier** — the classification of how far a Harness has been checked:
`static-checked` (binary and local state inspected with zero spawns),
`initialize-checked` (bounded ACP `initialize` handshake completed with zero
session creation), `session-checked` (explicit user-consented `session/new`
catalog check completed and closed), `blocked`, or `unknown`. Only
`session-checked` observes a live `session/new` catalog, and none of the tiers
proves live prompt quota or external MCP tool health.
_Avoid_: "ready" unqualified, "authenticated" for `initialize-checked`.

**Catalog evidence (`CatalogEvidence`)** — a non-authoritative cached snapshot
of the model, mode, and thinking choices observed from a real `session/new` or
`session/load` catalog, stamped with `(harness, acpBinary, authEpoch, cwd,
observedAt)`. Valid only as a setup or doctor hint while its scope stamp
matches the current Harness, binary, Auth generation, and workspace; runtime
creation still validates against the live wire catalog.
_Avoid_: "cached config", "authoritative catalog".

**Delegated mode** — the foreign session mode a Harness Profile names for
delegated children, in which the Harness runs authorized tools without
asking (`bypass` for Devin, `yolo` for Antigravity). Distinct from the mode
a session starts in (a Foreign session default).
_Avoid_: "bypass" for the general concept.

**Thinking facet** — a Harness's separate reasoning-effort Config Option
(Devin's `thought_level`). A Harness without one may encode effort in its
model choices; that is not a Thinking facet. Distinct from the native
`#variant` model-reference suffix (which names a Thinking-facet value
where one exists) and from the retired `thinking` public config facet.
_Avoid_: "variant" (a native model-reference suffix, not a foreign option).

**Foreign trajectory** — the Harness-side Trajectory a native OpenCode session
(**Native child** or **Native root session**) binds to, advanced by exactly one
Active writer at a time.
_Avoid_: "remote session" (vague), "Devin-side copy" (no copies exist, and
the Harness may not be Devin).

**Child binding** — the single KV document associating a **Native child** with
a **Foreign trajectory** (ADR-008): one JSON record at
`bindings/<childSessionID>` in the plugin's namespace, carrying schema
`version`, turn/phase markers, and `foreignAck` fields so recovery reconciles
landed versus acknowledged without compare-and-swap. Scoped strictly to a
**Native child**; distinct from **Binding** (the hosted Lane↔`sessionId`
association, which shares no native binding state) and from a **Root binding**
at `root-bindings/<sessionID>`.
_Avoid_: "binding record" alone (ambiguous with hosted **Binding**),
"child-to-foreign map" (describes the concept, not the record), "root binding"
or "session binding" (a **Child binding** is child-only).

**Root binding** — the single KV document associating a parentless **Native root
session** (main session or host fork) with its **Foreign trajectory** (ADR-008
root binding disposition, decision 114): one JSON record of the same `version: 1`
shape persisted at `root-bindings/<sessionID>` in the plugin's namespace, keeping
`bindings/<childSessionID>` strictly child-only. On first admission of a root
session with no `root-bindings/<sessionID>` record, a legacy pre-`42c2096` row
at `bindings/<sessionID>` that decodes cleanly and matches that exact session
and Harness is adopted once by verbatim copy to `root-bindings/<sessionID>`,
leaving the legacy key untouched; corrupt or mismatched legacy rows refuse
visibly. A forked session has a new `sessionID` and gets a fresh **Root
binding**, never inheriting its source session's foreign trajectory,
`consumedInputs`, `roleDelivery`, or `foreignAck`.
_Avoid_: "child binding" (child-only under `bindings/`), "hosted binding".

**Transcript history seeding** — the one-time delivery of prior native session
transcript history (ordered prior user, assistant, tool-call/result summary, and
compaction-summary context rows) into a newly created foreign ACP session
(`session/new`) when a native session already carries prior history before the
current turn's new user input — specifically on a host fork, a cross-Harness or
cross-model switch on an existing native session, or an idle **Foreign-subagent
child** resumed on its Harness (decision 114 items 3–4). All seeded message IDs
are recorded in `consumedInputs` on the new binding so prior turns are never
re-executed or re-sent on subsequent turns. The seeded history is the verified
ordered host context read through the public `ctx.session.context` seam —
genuine host compaction summaries plus user, assistant and tool rows; a
metadata-only **Binding checkpoint** is never substituted for conversation.
Applicable context is preserved without silent truncation or automatic
summarization, and the host never compacts automatically before a model switch
(Decision 115). When the context is unavailable or unverified, carries corrupt
or unknown rows, holds an incomplete or unrecoverable checkpoint, or exceeds a
documented and tested size guard (including the target model's evidenced,
operator-configured or runtime-learned context limit), the create/seed/replace
pauses visibly instead.
_Avoid_: "replay" (which means foreign `session/load` stream replay), "blind
resend" (seeding supplies prior context to a fresh `session/new` and stamps all
prior message IDs as consumed).

**Accepted turn** — a delivered turn whose replayed user echo matches the
Child binding's `sentText`: acceptance proven by evidence, outcome
unproven. It is a distinct phase from **acknowledged** (a real foreign ack
or settle evidence) — an echo alone never writes `foreignAck`.
_Avoid_: "acknowledged" for an echo-only reconcile, "resumed" (nothing is
resent or continued on the strength of an echo).

**Consumed input** — a native context row that a driver request actually
consumed, named by its genuine durable host message id (`msg_*`). Distinct
from a pending or promoted inbox input, which marks a submission state and
may never be consumed.
_Avoid_: "delivered input" (delivery is the foreign-side record),
"message id" alone (a pending id is not a consumed id).

**Role instruction block** — the selected Role agent's effective system
text sent to the foreign side exactly once per role or effective-text
revision, as one labelled block: the request's epoch baseline plus, when
verified durable updates exist, a labelled chronological-updates section
carrying those update texts. Native-authored guidance on the Foreign
trajectory; not a Foreign session default or an Effective foreign setting.
_Avoid_: "system prompt" alone (collides with the Composer's user prompt),
"foreign config".

**Steering** — ongoing-intent input directed at an already-running turn on a
bound Foreign trajectory. It names a wire capability, not an act.
_Avoid_: "amend" (that is the Amendment path), "interrupt" (a control
mediation act, not input delivery).

**Amendment** — the interim redirect of a running turn: a labelled
cancellation, genuine turn-correlated settlement, then resubmission on the
same Foreign trajectory, presented to the user as an amendment. Never
presented as steering.
_Avoid_: "steering" (no such channel is advertised), "resend" (resubmission
follows real settlement, not a silent retry).

**Creation policy** — an explicit model (and, via the canonical
`#variant` grammar on the admitted model's own catalog, effort) selection
admitted inside the **Bound-but-unprompted child** window at foreign-session
creation time. Distinct from **Explicit model selection** (the caller's
request), **Execution-driver identity** (the private route sentinel),
**Foreign session defaults** (what the session advertises), and **Effective
foreign settings** (what is confirmed in effect on a live trajectory).
_Avoid_: "explicit model" (that is the caller's selection, not its
admission), "model mediation" (a mid-session relabel; admission is
creation-only), "pure intent" (planning an operation is not a confirmed
`creationApplied` receipt).

**Binding checkpoint** — the native-side compaction summary for a bound
child, recording binding identity and state only (retained role, foreign
session, phase and acknowledgment markers) and noting that the full context
lives on the bound Foreign trajectory. Native history keeps a checkpoint;
the Foreign trajectory owns the conversation context. A checkpoint is
metadata, not conversation history, and can never stand in for the verified
ordered host context that **Transcript history seeding** requires.
_Avoid_: "context copy" (nothing is copied), "prompt summary" (it carries
no prompt-derived content).

**Effective foreign settings** — the model and thinking values in effect on a
**Foreign trajectory** as evidenced by that session's own confirmed catalog
responses or updates (`configOptions`, `config_option_update`). Distinct from
an **Explicit model selection** (caller request), **Execution-driver
identity** (private route sentinel), **Creation policy** (creation-time
admission target), turn-level actual-serving witness metadata, and the
separately evidenced **Delegated mode** policy.
_Avoid_: "native model" or "explicit model" for a foreign-reported setting,
"projected catalog row" (provider projection is not live session confirmation).

**Native ask** — a host `Permission.Service` pending request surfaced by the
`permission.asked` event and answered through native OpenCode permission UI
(`reject` denies; `once` and `always` answer under host session policy) during
native session mediation. Distinct from a foreign ACP
`session/request_permission` wire request and from a hosted **Permission
Request** inline card; native and hosted permission lifecycles are not
interchangeable.
_Avoid_: "permission card" (that is its rendering), "approval" (the answer,
not the ask), "Permission Request" (reserved for the foreign wire request and
hosted Lane card).

**Host-tool availability** — whether the OpenCode host exposes a local
host-owned tool surface in the current session (such as `@opencode/plugin-browser`
appending, after config and unconditionally, an ordered trailing `browser:* deny`
then `skill:browser deny` pair to every agent on v2.2.1 — the `browser:* deny`
rule alone through v2.1.4 — while a desktop browser attachment appends the
matching session-level allows (`browser:*`, plus `skill:browser` on v2.2.1)
after those denies without removing the agent-level signature; the session
allows are removed when the attachment releases), distinct from
**Effective permission intent** and **Foreign coverage channel** because local
tool unavailability hides a host tool without forbidding foreign agent work.
The host attaches no provenance field; `host-availability` is a local tag on the
exact ordered untagged trailing signature. For an untransformed caller array, an
identical final suffix is indistinguishable from the host append; the host
post-config pipeline places configured operator rules earlier, so real operator
config does not occupy those final slots.
_Avoid_: "foreign deny", "role restriction" (hiding an unattached local host
tool is not a user prohibition on foreign operations).

**Effective permission intent** — the user, operator, **Role agent**, and
session restrictions governing which foreign **ACP agent** actions are forbidden
or require approval before execution, resolved in host rule order
(`agent.permissions` before `session.permissions`,
last-match wins) after separating **Host-tool availability** rules by provenance
(`role-or-operator` and `session` versus `host-availability`).
_Avoid_: "raw permission array" (conflates host-tool availability transforms
with foreign-operation restrictions), "mode label" (**Delegated mode** is a
foreign wire setting, not the caller's restriction set).

**Foreign coverage channel** — an evidenced pre-action enforcement mechanism for
a specific **Harness** and `acpBinary` that intercepts restricted foreign actions
before their side effects, enforcing denial or mediated approval rather than
treating approval-required intent as permission
(`NativeForeignCoverageChannel`), distinct from **Effective permission intent**,
**Native observation part** rendering, and residual asks under **Delegated
mode**.
_Avoid_: "observation coverage" (`providerExecuted: true` observations display
activity without pre-action enforcement), "prompt guard".

**Client fs takeover** — the ACP client capability advertisement
(`clientCapabilities.fs: { readTextFile: true, writeTextFile: true }`) and
incoming JSON-RPC `fs/read_text_file` and `fs/write_text_file` request handling
on `src/acp.ts` for a **Harness** whose `acpBinary` is evidenced to route file
reads/writes through client `fs/*` (decision 114 item 5); flags or config alone
are never evidence, and unevidenced Harnesses — including production Devin and
Antigravity selections by default — advertise `fs: false`. Every incoming
`fs/read_text_file` (`action: "read"`, plus `external_directory` when the
resolved path escapes `cwd`) and `fs/write_text_file` (`action: "edit"`, plus
`external_directory` when escaping `cwd`) evaluates pre-action **Effective
permission intent** and mediates interactive approval through the OpenCode
host's `ctx.permission.assert` (`PermissionDomain.assert`) API before any
filesystem read or write occurs: `once` and `always` resolve in the host
`Permission.Service`, a `deny` rule or a user `reject` carrying feedback
rejects with `Permission.BlockedError`/`Permission.CorrectedError`, surfaced as
a `Tool.Error` whose message is returned to the foreign model as a JSON-RPC
error without touching disk, and a plain `reject` without feedback rejects with
the step-aborting `Permission.DeclinedError`. Covers only the `read`, `edit`, and
`external_directory` action classes actually routed through client `fs/*` on
that evidenced Harness; `shell` (`terminal/*`), `grep`, `webfetch`, and
unmapped vendor tools remain uncovered unless separately evidenced.
_Avoid_: "global permission bypass", "hosted fs serving" (hosted Lanes keep
their own lifecycle and `autoApprove: false`).

**Native observation part** — a native session transcript tool record emitted with
`providerExecuted: true` to display a foreign **ACP agent**'s tool call and
terminal result inside OpenCode history without local execution or pre-action
authorization. For non-subagent tools (`read`, `edit`, `bash`, `grep`, `glob`,
generic observations), the part records foreign activity only; for foreign
internal subagent calls (`run_subagent` on Devin or Antigravity equivalents), the
call additionally materializes a real **Foreign-subagent child** through the
supported host session API — the observation card itself does not link to it
(decisions 114 item 4, 116).
_Avoid_: "host-executed tool", "permission boundary" (recording an observation
neither centralizes execution in OpenCode nor enforces **Effective permission
intent**).

**Foreign workspace-boundary attribution** — the post-observation classification
of a failed foreign **ACP agent** tool call into one of three closed confidence
classes (`foreign-boundary` | `suspected-boundary` | `unclassified`) without
failing, pausing, cancelling, or retrying the turn (`REQ-NBR-21`, decision 117):
`foreign-boundary` only when the foreign client's own reported text matches a
pinned, vendor-specific denial phrase decoded through the **Wire dialect**;
`suspected-boundary` when the failed call reports a path outside the session
workspace (`cwd` plus decoded declared roots) with unknown text; and
`unclassified` otherwise. Host-decided refusals (scope admission or permission
bridge correlated by `toolCallId`) are suppressed and never reported as a
foreign-boundary denial. Warnings surface on existing surfaces (the hosted
**Lane**'s tool **Entry** derived at render time; the delegated child's tool
part plus a single text note on a foreground delegate result's `content` when
`>=1` confirmed denial exists) and are journaled once (`childSessionId +
toolCallId` dedupe, no binding state).
_Avoid_: "foreign sandbox verification" (ACP v1 exposes no structured denial
reason; silent skips and silent content truncation are undetectable), "host
permission denial" (host-decided refusals are distinct and suppressed).

**Foreign-subagent child** — a real OpenCode **Native child** session created
through the supported host session API (`ctx.session.create` with a
caller-supplied deterministic ID and `parentID` set to the owning native
session's ID; `location` is ignored under `parentID`) when a foreign session
reports an internal subagent call (`run_subagent` or Antigravity equivalent,
decision 114 item 4 as amended by decision 116, amending `MH-30`). The genuine
native parent-child relation makes it reachable in the stock subagent tree, and
parent completion surfaces through the stock subagent completion message — not
through a link on the original `providerExecuted` observation card. It carries
only foreign-reported task, progress, and result content, recorded through the
supported `ctx.session.synthetic` inbox and emitted by a plugin-owned replay-only
language model (zero new foreign sessions, zero prompts, zero additional model
inference, zero raw database writes, and zero fabricated model/agent/prompt
fields) and is durably marked as foreign-executed (`foreignExecuted: true`) in
plugin storage. While the foreign subagent is running, the child is non-admissible
for independent native turns; once idle (settled), resuming that child session ID
on its Harness starts a new foreign ACP session (`session/new`) seeded with that
child's foreign-reported transcript via **Transcript history seeding**.
_Avoid_: "synthetic ses_* ID" (the session is created by the host's own
`ctx.session.create`), "locally executed subagent" (its initial run was executed
inside the foreign Harness).

**Host-mediated MCP access** — the domain boundary in which a foreign **ACP
agent**'s own tool loop invokes an explicitly scoped OpenCode-owned MCP tool
while the host retains MCP server connections, OAuth state, permission
evaluation, and caller attribution. Distinct from direct MCP configuration or
token export, a foreign-owned MCP server connection, automatic OAuth or
permission inheritance, or a replacement host tool execution loop; the term
names the target boundary, not an approved bridge API or proven public-API
safety.
_Avoid_: "MCP passthrough", "MCP config export", "shared MCP client", "host
tool loop".

**Request kind** — the host's classification of a model request as primary,
compaction, title, or generate; only primary requests may create or advance
a bound foreign trajectory.
_Avoid_: "request type" (that is the wire type), "auxiliary call" (names
only the non-primary subset).

**Ownership release** — the protocol by which an Active writer yields a
foreign trajectory at an idle boundary so another client may become the
writer; a requested cancellation settles before release.
_Avoid_: "handoff" (reserved for document handoffs), "unlock" (that is the
Lock term's host metadata).

**Named client** — a Harness's declared normal native client (for Devin, its
desktop or CLI client) used for cross-client continuation proof; its name and version are pinned per
authorized run.
_Avoid_: "other client", "Devin Desktop" (an untested client is never
claimed).

**Child inventory** — a bounded frozen snapshot of the calling parent's
native direct children — ordinary, Harness-bound, and failed-before-binding
alike — newest first, paged by an opaque caller- and directory-bound cursor.
An observed listing artifact, distinct from the Native children themselves,
the foreign session catalog, and any live multi-page walk.
_Avoid_: "session list" (that is wire catalog data), "binding list" (Child
bindings are not the inventory), "live listing".

**Delegate config** — the versioned `config/delegate` document in the
plugin's global `ctx.storage` namespace: the durable record of Harness
registration (`enabled`/`rank`/`bin`), per-Harness Creation policy
(`models`), and opt-in `limits`. Read under precedence
per-project `ctx.options` → document → built-in defaults; per-project
variation exists only as the `ctx.options` overlay, never a second
storage scope. Retired public facets (`thinking`, `hosted`) remain
schema-known as **named-stale** entries: user data preserved verbatim but
carrying no weight in the effective policy.
_Avoid_: "settings file" (it is KV, not a file), "project config" (the
document itself is global per plugin id).

**Bound-but-unprompted child** — a Native child whose Child binding and
Foreign session exist but on which zero foreign turns have run: after
`session/new` and the persisted bound marker, before the first
`session/prompt`. It is an inventory row, and it is the only window in
which Creation policy writes may occur.
_Avoid_: "unbound child" (the binding already exists), "idle child"
(an idle bound child is past this window).

**Model slot chain** — a Creation policy's primary model plus up to two
ranked fallback slots, resolved against the freshly advertised catalog
inside the Bound-but-unprompted window: an unadvertised or misconfirmed
slot advances to the next; a transport/RPC failure fails the call, and
exhausted slots fail before any prompt. Same Harness, same child only.
_Avoid_: "fallback" alone (a supplied explicit model is never fallen
over), "retry" (nothing is resent).

**Outstanding child execution** — the counted unit of the opt-in
`limits` admission contract: a Native child execution in flight for the
calling parent's delegate-family calls, counted once regardless of how
many calls touch it, bucketed by effective mode (`observedMode ??
launchMode`).
_Avoid_: "running call" (a continuation joins counted work), "foreign
turn" (the unit is the host child, not the turn).

**Admission reservation** — the in-memory, same-process hold a
delegate-family call takes against an opt-in limit: pending-keyed by the
call's own id until a Call-provenance link keys it to a real child
`sessionID`, released only on genuine terminal or non-creation proof —
never by age or TTL.
_Avoid_: "lock" (no mutual exclusion of work), "lease" (no expiry).

**Call-provenance link** — the genuine call-correlated
`callID → childSessionID` correlation learned from the predecessor's own
`context.progress({sessionID, status:"running"})` emission, observed
through a pass-through wrapper on the call's `Tool.Context` and verified
against the child's `parentID`; never inferred from prompt text, title,
or approximate time, and never matched post hoc from a discovered
binding.
_Avoid_: "heuristic match" (only the causal emission correlates),
"result link" (the link exists even when the call's result is lost).

**Unledgered child** — a bound Native child created outside the
delegate-family tools (e.g. the builtin `subagent` bypass), never
admitted by the plugin and discoverable only by the Child binding sweep;
counted in both limit buckets once discovered.
_Avoid_: "orphan" (it may be healthy and intended), "untracked" (the
binding exists; only the admission record is missing).

**Side-effect tier** — the setup tool's action classification: static
read (`status`/`next`: filesystem stat, KV read, option parse — zero
engine spawns), bounded engine probe (`probe`: launch + `initialize`
returning initialize facts only — `initialize` carries no catalog —
no foreign turn, no writes), and document mutation (`choose`); install
and sign-in are consent-gated doctor actions outside the step tool.
_Avoid_: "read-only" for `probe` (a probe spawns a process and may
spend), "setup step" for the tier itself.

**Facts fingerprint** — the hash of the detected facts a `next` step was
planned over, revalidated inside the serialized write before `choose`
commits; a mismatched fingerprint means the facts changed under the
step and is a visible conflict.
_Avoid_: "version" (that is the document's `revision`), "checksum"
alone.

**Authored ACP worker** — a plugin-built child process speaking standard
ACP over stdio so `src/acp.ts` and the SpawnEngine seam drive it like any
other Harness engine. It is our artifact line with its own version pin
and attestation, fault isolation for engine failures, never upstream ACP
provenance and never a sandbox.
_Avoid_: "ACP agent" for the wrapper (the worker hosts an engine, it is
not the vendor's), "sandbox" (tool execution inside still runs with user
privileges).

**Derived SDK artifact** — the internally labelled build
`hmoc-freebuff-sdk@0.10.7-upstream.<sha>` of unmodified pinned
`CodebuffAI/freebuff` source: our build identity recorded as upstream
commit + `bun.lock` + Bun + bundler versions, never a new upstream
release and never published under its own name. Its compiled entry
(`dist/index.mjs`) ships as vendored package data under
`vendor/freebuff-sdk/`, beside the digest manifest and the loader that
gates the bytes. The shipped manifest records the identity it actually
carries — `sourceCommit`, package-relative `sdkDir`, `builtAt`
(tuple/Bun/build command), `artifacts` sha256 for the shipped entry, and
`omittedAssets` for audited-but-unshipped ripgrep/wasm digests; the full
pin tuple (commit + `bun.lock` + Bun/bundler versions) lives in
`.audit/2026-10-21-freebuff-fb01/build-identity.md`, not in the manifest.
_Avoid_: `@codebuff/sdk@0.10.7` for the derived build (the npm artifact
is a different, older tree), "vendored SDK" for bytes that arrive without
the digest manifest.

**Freebuff lease** — the server-admitted slot identity
(`freebuff_instance_id`) bound to a catalog row's handle and effort,
kept by the host-side lease layer (admission, heartbeat, stale-409
recovery). Distinct from the OpenCode child binding and from the opaque
SDK `RunState.sessionState`; turn-survival and `/session/reuse` are
client intent, not proven server guarantees.
_Avoid_: "session" (three different things carry that word), "slot
guarantee".

**Freebuff conversation** — the user's billable server-side unit behind a
**Freebuff lease**: an admitted, time-bound, credit-metered window keyed by
`freebuff_instance_id`, whose validity is reported by the server
(`GET /api/v1/freebuff/session`), never by local timers. Ending or letting
it lapse can forfeit the window and its credits, so it is never silently
created, rotated or ended. Distinct from the OpenCode **Native child**
session (our binding), the worker's foreign ACP session id (`fbws-*`) and
the opaque SDK `RunState.sessionState`.
_Avoid_: "session", "conversation" for the ACP or host-side objects.

**Catalog key** — the stable logical row id in the live Freebuff model
catalog (`m-…`): the selectable model id the picker and provider
projection present. Distinct from the row's `handle`.
_Avoid_: "model id" alone (the wire never carries the key).

**Catalog handle** — the opaque server-issued model string (`fbm1.…`)
on a live catalog row, sent on the wire as `AgentDefinition.model` and
`x-freebuff-model`. Rotation is not established; `fetchId` plus
`x-freebuff-catalog-protocol` is the staleness anchor and one
`409 freebuff_catalog_stale` earns one forced refetch plus retry.
_Avoid_: "rotated token" (structure is unverified), "model name" (it is
not display text).

**Delegation bridge** — the delegation-only path by which a foreign
**ACP agent** asks its owning host session for a child: exactly two
capabilities (`delegate`, `child_status`), one authority seam and one
fulfillment loop. Not a host-tool export and not **Host-mediated MCP
access**: nothing host-owned beyond those two capabilities crosses it, and a
missing capability is a visible refusal, never a silent fall back to vendor
tools.
_Avoid_: "bridge" alone (the permission bridge and the hosted routes are
different things), "MCP bridge" (the deferred optional leg), "tool export".

**Bridge grant** — the frozen authority a bridge request is checked against:
the verified **Root binding** or **Child binding** (parent session, the
**Foreign trajectory**'s session, calling Harness and **Role agent**), the
admissible **Delegation targets** and the binding's expiry. Resolved from the
binding when the bridge is enabled; a request supplies content only and never
names, widens or replaces it.
_Avoid_: "permission" (that is the host's own decision), "allowlist" alone (a
grant also carries targets, transports and expiry).

**Correlation nonce** — the per-binding opaque value seeded into the foreign
session and echoed by a bridge request so the host can match the two. It is
correlation, never authority: possession neither authenticates a caller nor
widens a grant, and never substitutes for the verified-binding check or the
host's own permission decision. Rotation re-seeds the **Bridge instruction
block**.
_Avoid_: "token", "secret" (both imply a bearer credential), "auth".

**Bridge directive** — the turn-scoped typed text a foreign model emits to
request one capability (the `delegate`/`child_status` fields plus the nonce).
Recognised only as a complete terminal block in the owning foreign session's
current assistant turn of a bridge-enabled binding; the same text in a user
prompt, quoted code, tool result, reasoning, earlier turn or replay is never a
directive. This is the `S2` leg; `S1` is a deferred optional leg into the same
request contract, not a second product.
_Avoid_: "command" (host commands and foreign slash commands are separate),
"tool call" (that is the host-side realization).

**Bridge instruction block** — the one-time disclosure seeded into a
bridge-enabled foreign session: the directive grammar, the two capability
schemas and the current nonce, as ordinary prompt content reusing the **Role
instruction block**'s delivery path. Content and correlation only — never a
tool definition, never a **Role agent** instruction set and never authority.
_Avoid_: "system prompt" (the Harness owns its own instructions), "tool
definition" (no host tool is advertised to the Harness).

**Bridge call** — one admitted bridge request realised as a real registered
host tool call: pending from the moment the driver emits it until its result
returns, carrying the host's own call identity (`toolCallId`), which alone
correlates the result arriving on the next host request. The result is
delivered once, as a prompt on the same foreign session. Every bridge call
settles exactly once — delivered, cancelled or failed — and a settled call is
never re-delivered, re-admitted, reset or replaced by a new foreign
conversation.
_Avoid_: "foreign tool call" (the foreign Harness keeps its own tool loop),
"subagent call" (vendor subagents stay foreign), "context reset" (a settled or
failed call never resets context or buys a replacement).

**Delegation target** — the one selection a bridge request names for its
work: a foreign **Harness** and its model (including the calling Harness, i.e.
self-delegation), a named host **Role agent**, or a native provider-model
reference (**Explicit model selection** with its **Thinking facet**). Matched
against the grant after the catalog validation the existing selector grammar
already performs; naming a target never widens the grant, and an unadvertised
or contradictory model/effort carrier refuses instead of being normalised into
something selectable.
_Avoid_: "route" (Harness rank and default routing), "OS target" (the cross-OS
artifact tuple), "model" alone (a target also carries Harness, agent and
variant).

## Hosted Lane

The `/devin` and `/antigravity` routes drive a foreign ACP session inside the TUI.

**Lane** — the per-(Harness, cwd) controller (`src/hosted/lane.ts`) that owns one
Harness ACP subprocess (its Engine), its current session binding (mutable via
`session/load`), entry history, busy state, permission queue, and the sink the
mounted view listens on. A lane outlives route mounts.
_Avoid_: "connection", "process wrapper", "session object" (the session belongs
to the Harness; the lane hosts it).

**Hosted session** — an ACP session driven inside the OpenCode TUI hosted routes
(`/devin`, `/antigravity`), as opposed to a session observed in a native CLI,
desktop app, or another ACP client.
_Avoid_: "embedded session" (vague), "chat".

**Hosted View** — a hosted route's full-screen surface (`/devin`, `/antigravity`)
(header, Entry Log scrollbox, Overlay strip, Composer) that renders one lane's state
and owns the route keymap and focus. The `session.panel` and `sidebar.content` claims
are secondary windows onto the same lane (ADR-005). Formerly "Hosted Devin View".
_Avoid_: "the TUI" (that's the host), "Devin screen" / "Antigravity screen",
"the plugin" (the plugin also owns the delegation surfaces).

**Session picker** — the listing surface over `session/list` used to choose an
existing foreign session to continue, or `+ New session in <cwd>` to start one.
It mounts inside the host's dialog chrome (titled frame, search field,
date-grouped rows, `●` bound marker, `⚿` lock badge); an unbound lane is
offered it automatically, a bound lane summons it with ctrl+o (ADR-002).
_Avoid_: "history view", "recent list".

**Overlay** — a focus-owning surface layered over the Composer strip inside the
Hosted View: the Permission Request card, a Config Option picker, or the
commands list (`ctrl+g`). At most one owns input at a time; esc dismisses it
before esc navigates. Inline completion popovers (file mentions and leading-`/`
discovery) are not Overlays — they never take textarea focus — and the Session
picker is not an Overlay either: it rides the host's dialog surface.
_Avoid_: "modal", "popup", "dialog" (the host's `ui.dialog` chrome is a
different surface).

**Composer** — the Hosted View's text field ("Message <Harness>…") where
prompts, File mentions, and `/<name>` commands are typed. It takes no input
while the lane is unbound or a bind is in flight, and its text survives failed
submits, failed binds, and failed mention resolution.
_Avoid_: "input box", "prompt field", "editor".

**Entry** — one rendered item in the lane's Entry Log: user prompt, agent text,
thought, tool call card, plan, permission card, or system notice.
_Avoid_: "message" (too narrow; a tool call is not a message).

**Entry Log** — the lane's complete ordered list of Entries. The lane owns it,
so unmount, remount, and session swaps never lose it (a swap clears it and
rebuilds from the new Replay); the view renders only a windowed tail.
_Avoid_: "chat history" (history belongs to the Harness; the log is the lane's
render model of it), "scrollback".

**Replay** — the stream of `session/update` notifications a Harness emits after a
successful `session/load`, reconstructing prior entries in order.
_Avoid_: "scrollback fetch", "history sync".

**Binding** — the association between a hosted **Lane** and a Harness
`sessionId`, made via `session/new` or `session/load` (a lane *binds* a
session). A lane with no binding is **unbound** and renders the Session
picker; a bound lane can re-bind — a session swap — only while idle. Distinct
from a native **Child binding** (`bindings/<childSessionID>`) and from any
**Native root session** association; hosted and native bindings are never
shared or interchanged.
_Avoid_: "attach" (reserved for view→lane sinks), "activate", "open session",
"Child binding".

**Pending binding** — a `session/load` in flight on a lane. Its sessionId is a
legitimate update tag alongside the bound sessionId until the load resolves or
fails, because ACP v1 replays the conversation *before* the load response.
_Avoid_: "loading session" (the session is not loading; the binding is pending).

**Session swap** — re-binding an idle bound lane to a different session via
`session/load` on the same engine; the Entry Log clears and rebuilds from the
new Replay.
_Avoid_: "switch" (that is host route switching), "reconnect".

**Stale update** — sessionId-tagged traffic (`session/update`,
`session/request_permission`, `available_commands_update`,
`config_option_update`, tagged `_cognition.ai/output`) whose tag is neither the
bound session nor the pending binding. Dropped before it can touch lane state.
_Avoid_: "orphan message", "ghost update".

**Session descriptor** — one `session/list` row after wire-boundary
translation: `sessionId`, `cwd`, `title`, `updatedAt`, plus `isLocked`,
`lockHolderPid`, `createdAt`, `requestingTabId`. The picker's row model.
_Avoid_: "session object" (the session lives in the Harness; the descriptor is only
a listing), "session entry".

**Lane registry** — the module-level `lanes` map behind `hostedLaneFor`
(`laneFor` underneath), keyed by `(Harness, cwd)` slot
(`${harnessId}:${cwd}:default`), never by `sessionId`. A session never owns a
registry key.
_Avoid_: "session map", "lane cache".

**Lock** — Devin's Wire dialect metadata (`isLocked` + `lockHolderPid`) marking a
session currently held by another live ACP client. A locked session is
listable; loading it is permitted but the UI must surface the lock state.
_Avoid_: "busy session" (busy means in-flight prompt in our lane).

**Permission Request** — a foreign `session/request_permission` wire request
the hosted **Lane** holds open until answered, rendered as an inline Entry
card with numbered options; answering sends `{outcome: "selected", optionId}`
or `cancelled` on esc. On the hosted lane only the human user answers it —
never auto-approved — and it is distinct from a host `Permission.Service`
**Native ask** in the native lifecycle.
_Avoid_: "Native ask" (that is the host `Permission.Service` request in the
native lifecycle), "approval dialog", "auth prompt", "permission dialog" (it
is an inline card, not a dialog).

**Persistence scope** — the lifetime and storage boundary over which a granted
permission choice (such as `allow_always`) is remembered by an ACP server process
(turn-scoped, session-scoped, or persistent across restarts). Defined by the
Harness or agent process, not by a client-side setting, protocol field, or wire
write-back method.
_Avoid_: "global grant" (unless verified persistent in the agent's store),
"client preference".

**File mention** — an `@<path>` token in the Composer that resolves to a
Resource reference block in `session/prompt`, including explicit user-authored
absolute `@/path` references to existing local files or directories.
_Avoid_: "attachment" (no upload occurs), "file tag".

**Resource reference** — a `resource_link` content block in a `session/prompt`
payload carrying a `file://` URI, the mechanism behind file mentions.
_Avoid_: "attachment" (no upload occurs), "context injection".

**Available command** — a foreign slash command delivered via
`available_commands_update` as `{name, description, input.hint, ...}`.
Invoked by sending `/<name> <args>` as ordinary prompt text; discovered via
leading `/` in the Composer or `ctrl+g`.
_Avoid_: "plugin command" (that is the host's keymap command), "slash handler".

**Hosted creation policy (`HostedCreationPolicy`)** — RETIRED (SET-08/NBR-01).
Hosted `session/new` performs zero automatic post-creation config writes: the
advertised catalog binds as-is, and `model`/`mode`/`thinking` change only
through the interactive pickers or the mediated `hivemind-hosted` targeted
request. A legacy `hosted.<harness>` block in `config/delegate` is preserved
user data, never applied; `session/load` continuations never write.
_Avoid_: "hosted defaults" (implies foreign session defaults), "delegated mode".

**Origin-labelled completion** — the Composer completion and command-list
taxonomy that badges every entry by its true source: `[foreign command]`
(from the bound foreign session's `available_commands_update`), `[client control]`
(local Hosted View controls executed with zero wire traffic), `[host skill]`
(discovered OpenCode skill definitions staged as bounded context), and
`[host reference]` (declared project reference paths staged as Resource
references).
_Avoid_: "slash commands" for all four origins, "ported skills".

**Tool lane** — the `devin_session` record in `index.ts` (an `Acp` plus a
session id keyed by `${sessionID}:${name}`). A different type sharing the lane
name; its lifecycle is per-tool-call, not per-route, and ADR-001 does not
govern it.
_Avoid_: reusing "lane" unqualified when both kinds are in scope.

## Verification

How runtime behavior is proven and built.

**Service restart** — restarting OpenCode V2's shared background server with
`opencode service restart`. The server owns sessions, plugins and tool execution;
restarting a terminal client is a different operation. A private `--standalone`
server is distinct from the shared service.
_Avoid_: "restart" alone when client versus service lifetime matters.

**Client restart** — exiting and reopening an OpenCode interface, such as closing
the TUI with Ctrl+C and launching it again. In shared-service mode this does not
by itself restart the background server.
_Avoid_: "service reload" for a client exit and reopen.

**Journey** — a declarative runtime drive consumed by the project-local verifier
on the installed host. Canonical journeys live in `verify-journeys/`; narrow
local recipes use separate readiness keys. Its asserting `proves:` names
identify exactly the behavior its receipt establishes.
_Avoid_: "e2e test" (that is the legacy `test/e2e/devin-pty.py` drive),
"scenario" (a fake-acp script, not a journey).

**Product snapshot** — an owned, reproducible build output placed under a
content-digest path. Its digest covers the declared payload (input bytes,
executable modes, lockfile identity); its manifest records External mutable
dependencies, and a digest-path directory it did not emit is unowned —
never adopted or overwritten.
_Avoid_: "build artifact" (no manifest or ownership is implied),
"installed copy".

**External mutable dependency** — a dependency recorded in a Product
snapshot's manifest whose current bytes lie outside the digest's coverage,
such as an installed dependency tree. Recorded for honesty, never claimed
immutable.
_Avoid_: "vendored input" (its bytes are not hashed), "locked dependency"
(lockfile identity is covered; the tree is not).

**Selected candidate** — the single plugin entry a snapshot build or
freshness check operates on at a time; a source-path selection is reported
as such rather than hashed. Distinct from the payload (the digest-covered
input set) and the emitted Product snapshot.
_Avoid_: "payload" (that is the covered input set), "snapshot" (that is
the output).
