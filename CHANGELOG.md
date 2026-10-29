# Changelog

## 0.4.2 — 2026-10-29

Patch release. No support floor is raised or lowered: the declared range stays
OpenCode v2.1.4 or later within v2 (below v3), and the OpenTUI peer declaration
stays `>=0.6.8`. The shipped `@opencode/client` runtime dependency moved to
2.2.1, and the OpenCode and OpenTUI packages this repo builds and tests against
moved to 2.2.1 and 0.7.0. This is not a general runtime compatibility claim.
Runtime evidence is partial and bounded. A bounded isolated test linked the
2.2.1 client to a real OpenCode 2.1.4 server and passed plugin registration,
session/RPC and TUI surfaces. Inference, ACP delegation and hosted attach were
not exercised, full live proof on 2.2.1 is still pending, runtime equivalence
with the host-bundled plugin and renderer copies is unproven, and the owner's
TUI UAT remains pending.

### Changed

- The shipped runtime dependency `@opencode/client` moved from 2.1.4 to
  2.2.1. Development pins moved to `@opencode/plugin` 2.2.1, `@opencode/theme`
  2.2.1, `@opentui/core` 0.7.0, `@opentui/solid` 0.7.0, and the
  `@opentui/core` theme override to 0.7.0. Peer declarations and the declared
  support range are unchanged.
- Declaration checks: the plugin TUI declaration file is byte-identical
  between 2.1.4 and 2.2.1, and the `@opentui/solid` `index.d.ts` /
  `components.d.ts` plus `@opentui/core` `index.d.ts` / `syntax-style.d.ts`
  are byte-identical between 0.6.8 and 0.7.0. The Promise plugin/client
  declarations gained members additively. `@opencode/theme/tui` gained the
  `migrateV1NewDesigns` export; no broader theme equivalence was checked.
  OpenTUI core 0.7.0 also moves native ABI, renderable, scroll-chain and
  stdin-parser declarations; only the surfaces the root suite covers were
  exercised, and the rest remain unverified.

### Fixed

- A fresh native delegation on an OpenCode 2.2.1 host is no longer refused as
  restricted by the host's own browser-availability marker. 2.2.1 appends an
  ordered `{browser:* deny}` + `{skill:"browser" deny}` pair to every agent
  until a desktop browser attaches, while 2.1.4 and earlier append the browser
  rule alone. OpenCode runs its config plugin before the browser plugin and no
  later internal plugin mutates `permissions`, so real operator configuration
  is appended before the availability signature and always keeps its intent.
  The tag is positional and tries the untagged ordered pair first, then an
  untagged trailing `{browser:* deny}` alone. A paired tail tags both rules. A
  reordered two-rule tail still tags only its trailing browser rule, leaving the
  earlier `skill` deny as role or operator intent, so a genuine skill
  restriction still refuses. A rule whose provenance the caller already
  supplied is never retagged, and a pre-tagged rule in a candidate suffix
  prevents that signature from matching. The merge receives raw arrays with no
  provenance bit, so an arbitrary untransformed caller array whose tail matches
  the signature is ambiguous and is read as availability. Previously, with the
  2.2.1 pair, neither rule was tagged, both read as intent, the `browser` and
  `skill` classes had no coverage channel, and admission refused before any
  `session/prompt`.
- Seeding a fresh trajectory no longer fails because a session's durable
  history carries rows this plugin does not author. The durable listing
  projection knew only the two synthetic carriers this plugin writes, so an
  instruction-file load, a steer/plan-mode/background/restart notice, a shell
  completion row, a failed compaction barrier, a prompt file/skill/agent
  sidecar or a background shell paused every later seed with "the durable
  transcript could not be verified". The projection now mirrors the host's own
  lowering, including per-mime sidecar decoding and host notices under their
  durable id, and scopes corruption to the visible post-barrier window.
  Binary attachments and genuinely malformed or unknown rows still fail closed.
- The packaged `bin/hmoc-steward-acp.mjs` ships executable mode 0755; earlier
  tarballs carried 0644.

## 0.4.1 — 2026-10-27

Patch release. Live receipts on the development host cover the six non-paid
bridge cells (Devin and Antigravity mains delegating to Devin, Antigravity and
named OpenCode children), `self` and `sessionID` continuation, the
`child_status` and model-only routes, and one observed cross-provider model
switch. Freebuff headless inference is not fixed: the upstream service refuses
free-mode completions with `403 free_mode_cli_required` on both free and paid
catalog rows, and the Freebuff terms restrict free-tier inference to the
official interactive clients. The Freebuff harness therefore ships
unconfigured/experimental; supported alternatives are the paid Codebuff SDK or
BYOK inside the official client.

### Added

- Per-harness delegation options are configurable per project under
  `plugins[].options.harnesses.<id>` in `opencode.json`: `models`, `bin`,
  `limits`, `bridge` and `permissionPolicy`. Options parse once at plugin
  load, and an unknown facet inside a harness entry refuses the load by
  name instead of being ignored or coerced.
- `harnesses.<id>.bridge` is a strict boolean opt-in (default off) that
  lets that harness's delegated session call back into OpenCode with the
  `delegate` and `child_status` capabilities. The grant is minted once per
  new session binding, so enabling never reaches existing sessions, while
  disabling stops further bridge callbacks on existing bindings after the
  options reload without rewriting the stored record.
- `harnesses.<id>.permissionPolicy` chooses how that harness's foreign
  permission asks are answered: `ask` shows the host permission card and
  fails closed when the endpoint is unreachable, `allow_once`
  automatically answers every foreign ask with that ask's own
  `allow_once` option — cancelling when the ask offers none — and
  persists nothing, and `deny` refuses by name before any card. The
  global `delegate_approve` option is the fallback and resolves `ask` for
  any unrecognized value. No option value grants a saved or standing
  approval — there is no "always" vocabulary.
- Permission mediation outcomes — policy deny, endpoint unavailable,
  foreign-deny-wins, cancelled — surface as named refusals in the
  delegate-visible turn stream instead of silent stalls.
- The `delegate` bridge directive accepts an optional `sessionID` to continue
  an owned native child on the same target. The pointer is forwarded verbatim
  and `model` is always omitted, so the host binds the child's stored model
  and role; a `{model}` target, `reference: true` or a caller model selector
  refuses by name (`continuation-target-invalid`,
  `continuation-model-override`). Ownership stays the tool's verdict — a
  missing, wrong-parent or cross-provider session id refuses before any
  dispatch.
- `{agent:<id>}` bridge targets run named host agents in a plain OpenCode
  child on the agent's own configured model — never a foreign session. The
  call prefers the injected `hivemind-delegate` tool, falls back to the
  legacy `subagent` name, and refuses `tool-not-advertised` when neither is
  advertised; a `background` named-agent run rides the `subagent` builtin
  only. An unknown agent or a failed create, prompt or wait refuses as
  `agent-unavailable` or `agent-child-failed` instead of failing open onto
  a foreign stream. `{harness, agent}` qualifies the harness's own
  registered delegate agent or refuses `agent-not-on-harness`. A `self`
  target delegates the binding's confirmed selection.
- A selected host skill row delivers its body once on the bound trajectory
  under a host-origin label — never as system authority or user speech —
  and is never relabelled or replayed.
- Freebuff children gain host-mediated file access: `read_files`,
  `write_file` and `str_replace` are the declared exception to the
  deny-everything tool policy, each request routed through the host's
  client-fs mediation under the same permission pipeline — never a
  worker-local fs. Windowed reads never widen past the pinned bounds and a
  present-but-non-numeric bound refuses by name with zero fs requests.

### Changed

- Supported host baseline is OpenCode v2.1.4 or later within v2 (with
  OpenTUI 0.6.8); the pinned Antigravity ACP server build is 1.4.3
  (developed against Antigravity CLI 1.4.7).
- Foreign permission asks and client-fs mediation ride the host's own
  permission pipeline — configured rules, saved allows and evaluate hooks
  decide — replacing the duplicated local evaluator that had drifted from
  host semantics. Devin delegated sessions pin the advertised
  `accept-edits` mode, Antigravity asks route the same way and previously
  silent cancels surface by name. The `delegate_approve` option remains
  the fallback for residual asks only; nothing gains a saved or standing
  approval.
- A binding whose child answers under a different Harness no longer refuses
  before foreign work: the mismatch becomes a mid-session switch — a fresh
  foreign session starts at turn 0 and receives every unconsumed user and
  skill row plus the complete stamp history, while the replaced trajectory
  row is preserved.

### Fixed

- Replayed provider-executed tool history could kill a session switched off
  a Harness provider: the host replayed persisted observation pairs
  verbatim and the destination adapter rejected the assistant `tool-result`
  content, so every later turn and compaction failed for the new provider.
  A sanitizer on the `context`, `compaction` and `generate` session hooks
  drops a provider-executed `tool-result` and its paired call
  when the pair is marked for a carrier other than the destination — and,
  because carrier pairs lose their mark on replay, also drops every
  unmarked provider-executed pair at a non-carrier destination, including
  pairs an unrelated provider may have produced. Pairs marked for the
  destination itself, non-carrier-marked parts and non-provider-executed
  parts pass through, and the hook is best-effort and never throws. The
  emit itself stays host-owned — the defect is documented locally, not
  reported upstream — and the mitigation is proven on one observed
  provider switch plus its contract tests, not a general cross-provider
  guarantee.
- A bridge caller whose owned nonce expired or was already replayed now gets
  exactly one visible `[bridge] refused: <reason>` note instead of a silent
  delegation stall; forgery-shaped and cross-binding classes stay
  journal-only.
- An ambiguously answered permission create is never replayed: an ask the
  host already answered, or a consent card whose delivery is uncertain,
  resolves through a bounded reject-release instead of a duplicate card, and
  transient host-permission endpoint stalls recover. Bare foreign asks are
  enriched with the observed tool-call facts before the card is shown.
- A permission ask held past the Freebuff worker's request ceiling no longer
  produces an early `FREEBUFF_FS_DENIED` while still pending: the fs channel
  carries no timer and settles only on the response, the bound turn's abort
  or worker close.
- A re-asked paid admission under an identical tool call never replays a
  settled verdict, and a refunded purchase classifies from the server's
  canonical message: the dead lease clears, the conversation retires as
  `freebuff-conversation-refunded`, and every later prompt, model write or
  load refuses until a fresh explicit admission; the retired session leaves
  the listing. An unselected turn names the eligible catalog keys it can
  see — or says none is eligible — and points at the free doctor session
  bootstrap rather than silently choosing one.
- Repeated mid-session Harness switches preserve every switched-out
  trajectory generation under its own foreign session instead of skipping
  or overwriting the first generation's row, and a lost settlement write
  heals from the verified prompt echo plus a replayed assistant tail instead
  of pausing every later turn; unprovable states still pause by name.
- Generated tier aliases no longer squash a real wire carrier's effort menu,
  scope-stale catalog rows are eagerly withdrawn on an auth-epoch advance,
  and the broker epoch seeds from recorded auth state; an owner-reported
  epoch never re-scopes an external-credential Harness's inventory, and the
  bridge no longer pre-gates carrier admission — the delegate tool owns
  that check.
- The deny-mode Freebuff provider no longer projects its sentinel as a
  selectable main model.
- Freebuff delegates show readable provider display names instead of
  internal identifiers in agent-facing listings. The public model id stays
  the opaque catalog key, and harness ids, defaults and stored config keep
  their previous values.
- Freebuff requests use the upstream header names and signing set. The
  catalog GET carries `x-freebuff-catalog-protocol: 1` and
  `x-freebuff-client: cli`; scoped session requests carry the protocol
  header and the device signature headers are `x-freebuff-device-ts` and
  `x-freebuff-device-sig`. The catalog fetch id
  `x-freebuff-catalog-fetch` rides inside the Ed25519-signed header set on
  every session and completion request, so the id the signature covers is
  the id the request sends.
- Freebuff session bodies are decoded strictly at the boundary. The
  conversation-facts response and the admission response accept only known
  statuses, and an `active` status must carry its bound `instanceId`,
  `model`, `admittedAt`, `expiresAt` and `remainingMs`. A malformed body
  fails closed as `freebuff-session-facts-invalid` or
  `freebuff-admission-refused` instead of passing a cast. The first facts
  read after admission must prove the same bound instance and an unexpired
  server window before a turn rides the lease.
- The Freebuff catalog reads `issuedAt`, `refreshAt` and `opensAt` as
  finite epoch-millisecond numbers. A present non-numeric value is refused,
  and a malformed `opensAt` row stays ineligible, so a gated row cannot
  unlock through a string value.
- An unselected Freebuff `session/new` no longer emits a `select` config
  option without a current value. The model selector stays out of
  `configOptions`, and the catalog travels on the standard top-level
  `_meta["freebuff/catalog"]` carrier for `session/new`, `session/load` and
  `session/set_config_option` responses plus `config_option_update`
  frames. Unselected discovery stays valid, no default model is invented,
  and a refused `freebuff/model` or `freebuff/effort` selection creates no
  session and spends nothing.
- Freebuff lease and conversation facts travel on standard ACP carriers:
  `session_info_update` `_meta["freebuff/session"]` and the prompt result's
  `_meta["freebuff/run"]` for usage and the final snapshot. The invented
  session-update variants are gone. Native delegation records each
  observation for its owned child, rejects an `instanceId` or model
  mismatch, and exposes the freshest one through child inspection with a
  `liveAttached` marker. That marker is `false` after a restart, so a
  recorded observation never presents a stale lease as live and never
  becomes catalog evidence or an entitlement claim.
- The admission consent card shows the validated decision facts for the
  selected model: its key and readable label, the server's declared
  session price when present as a finite number, the available balance and
  the daily reset instant. An absent or null fact renders `unknown`, never
  zero and never free. A prospective new-window duration is not shown
  because the server does not provide one, and an active-window expiry is
  not inflated into one.
- The setup effort (`thinking`) check validates a configured `model#effort`
  value against that model row's own metadata instead of an unattributed
  global menu. A row whose metadata omits efforts stays unknown, so a
  configured effort from another model is reported as stale rather than
  accepted, while legacy rows without metadata keep the generic check. No
  full or live setup pass is claimed.
- ACP engine spawn and close now run through one ownership seam:
  process-group ownership is proven before any group signal, concurrent
  hosts are spared during reconcile, and a throwing close or reap reports
  instead of silently succeeding. Abandoned late spawns and timed-out CLI
  probes are reaped and reported.
- An interrupted foreign turn records as an interruption instead of an
  unknown-finish failure.
- A Freebuff child re-binds its owned session reference on `session/load`
  without buying a new lease and replays the persisted lease on the
  standard carrier; a refused attach persists its stop fact only when a
  fold exists.
- `hivemind-doctor` refuses `sign-out` on harnesses whose credentials are
  managed externally.
- Applied reasoning effort no longer looks like a silent model. Each settled
  native turn records its non-empty thought-frame count and size in the
  existing journal, and a turn whose effort is confirmed but which received no
  reasoning says so as ordinary text. Received reasoning stays verbatim; no
  placeholder is invented.
- A repeated Freebuff admission cannot inherit an earlier approval. An unknown
  session price, an automatically decided permission result or a settled
  approval from another admission refuses, and every new paid admission needs
  its own explicit human confirmation.
- The local build freshness check compares the current dereferenced dependency
  directory with the identity the manifest recorded, so a plugin installed from
  a symlinked `node_modules` reports fresh while a retargeted or missing
  dependency still refuses by name.
- Release and CI workflows pin the same Bun 1.4.4 runtime the development
  baseline declares, and a regression refuses any future stage that drifts.
- A foreign harness can ask the host to read a declared reference instead of
  being granted the directory. `delegate` accepts an optional
  `reference: true`, admitted only for a configured native role whose effective
  host rules are deny-all plus path-scoped `read`/`external_directory` inside
  the declared roots, with its own model and no parent-session widening. Any
  other reader, a caller-supplied model, a disabled bridge or a refused target
  reports `reference-roots-unavailable` with the reason and how to fix it.
  Ordinary delegation is unchanged, and a reader is never inferred from a role
  name or a prompt.

## 0.4.0 — 2026-10-22

Breaking pre-1.0 cleanup.

### Removed / unsupported

- `devin_run` and its one-shot `devin` CLI interface are removed: `-p`,
  `--cloud`, `--agent-type`, `--permission-mode` and
  `--respect-workspace-trust`, plus the one-shot kill/timeout path, are
  unsupported **as removed one-shot interface only**. The retained
  `devin_session` tool still takes its own `cloud` and `agent_type` inputs
  through the shared `devinAcpArgv` builder — those flags are gone with the
  one-shot CLI, not with the ACP session surface. The one-shot
  `--respect-workspace-trust` behavior and the exact one-shot kill/timeout
  semantics have **no replacement** on the ACP surface.
- The duplicate `devin_doctor` tool is removed with no aliases or
  compatibility shims; `hivemind-doctor` is the sole setup-diagnostic
  doctor surface — engine checks are doctor verbs, not setup verbs.
  `devin_session` (persistent ACP sessions), `delegate-devin`,
  `hivemind-delegate`, `devin-children` and `hivemind-setup` remain the
  public agent surface.

### Changed

- `hivemind-setup` exposes exactly three actions — `health`, `next`,
  `choose`. `health` and `next` are static reads with zero writes and zero
  engine spawns; `choose` remains the only mutation and persists settings
  only when a setting actually changes. `status`, `verify`, `probe` and
  `preflight` leave the public surface; engine checks run through
  `hivemind-doctor` under its existing consent guard.

### Release process

- Private CI verifies and publishes the same frozen tarball. npm
  provenance is not claimed for private source. Publishing is separate
  from owner UAT.

## 0.3.0

- Ships the OpenCode 2.1.2 / OpenTUI 0.6.6 compatibility repair as a
  published release: the hosted route no longer blanks on hosts bundling
  `@opentui/core 0.6.6`, whose syntax constructors require the renderer's
  resource owner. The old 0.5.x call shape is still used when the host
  exposes no owner, so one build serves both generations. Published `0.2.2`
  and the `testing` tag `0.2.2-test.5` predate this repair.
- Fixes first-run bind: a retired `delegate.devin.thinking` key in a saved
  config document no longer blocks a first bind, and the armed
  config-defect notice now survives the bind's log reset and the load's
  cwd-retry reset instead of being silently dropped.
- Adds the Freebuff delegate Harness (`freebuff-native`): a task delegated
  with `harness: "freebuff"` runs in a genuine native child session driving
  the pinned Freebuff SDK. The compiled pinned-source artifact ships inside
  the package (`vendor/freebuff-sdk/dist/index.mjs`, Apache-2.0 with its
  LICENSE and NOTICE), resolves package-relative behind a recorded-digest
  gate, and runs against eight declared runtime dependencies. The lane is
  deny-only: every tool is denied, it never auto-enables, and it never
  takes a rank fallback — explicit owner opt-in is the only admission.
- The Freebuff credential resolves from `CODEBUFF_API_KEY` only — never
  argv, never the Freebuff CLI's keychain item. The credential egress fence
  permits a first-party HTTPS endpoint or loopback only and refuses before
  any Authorization header is attached. The live entitled turn is
  **unproven**: no call with a real `CODEBUFF_API_KEY` has been driven, so
  live use requires an entitled key and remains unverified.

## 0.2.2

- Fixes the crash that blanked the hosted route on OpenCode 2.1.1: the host
  bundles `@opentui/core 0.6.6`, whose syntax constructors require the renderer's
  resource owner. The old 0.5.x call shape is still used when the host exposes no
  owner, so one build serves both generations.
- Two publish attempts (`test.2`, `test.4`) were staged first and only became
  downloadable after the registry finished replicating them. The registry account
  publishes with a legacy 2FA-bypass token, which npm now restricts by staging
  versions instead of accepting them directly, so a version can report success
  while its tarball is briefly unavailable. CI publishes only on manual dispatch.

## 0.2.2-test.3

- Republished as `0.2.2-test.3` because the `0.2.2-test.2` tarball was not yet
  downloadable when that version was tagged; it replicated later. Prefer the
  `@testing` tag or a version whose tarball resolves.
- The installer now reports the version it actually ships. It previously carried
  a hardcoded `0.2.1` string.

## 0.2.2-test.2

- Testing prerelease for external-machine acceptance. Publishes under the
  `testing` dist tag so `latest` keeps pointing at `0.2.1` until the owner
  accepts the behaviour UAT.
- Carries the landed disable policy: a disabled or thinking-refused harness
  refuses new foreign work at the shared config-write owner, not only at the
  management route, while inspection and running turns survive.
- Carries the child-session projection: foreign subagents appear as OpenCode
  children with durable lineage, and their recorded rows become visible after
  the child's first turn.

## 0.2.1

- Rename the published CLI to `bin/hmoc-steward-acp.mjs`. The npm file list no
  longer shows the old `hivemind-steward-oc-devin` name.
- Rewrite the README as the page agents actually read: install, one-time setup,
  daily use, health, requirements, failure recovery, and safety. English and
  Vietnamese stay side by side.
- Add a support section pointing at the project portal, Ko-fi and Buy Me a
  Coffee. Donations are optional and unrelated to installation.
- Keep the plugin id and its `config/delegate` namespace unchanged so saved
  setup survives the package rename.

## 0.2.0

- Compile the npm TUI with the Solid transform before packing. The exported
  `dist/tui.js` loads from `node_modules` without React or repository JSX config.
- Check the extracted npm package by opening a hosted view and rendering a
  streamed reply from the deterministic ACP test agent.
- Keep the saved setup when unchanged, allow explicit setting changes, and
  finish setup when foreign model defaults are selected.
- Install from npm through OpenCode. GitHub is the front page and user guides,
  not an alternative installation source. Correct both language guides for
  model variants, ACP authentication and supported managed server bundles.
- Advanced tools still have direct schemas. Code Mode thinning and a dedicated
  TUI setup wizard are deferred, not claimed as shipped.

## 0.1.0

Initial npm release. Registry readback now confirms `0.1.0` is published.
Its raw TSX package entry can fail with a missing React import; use `0.2.1`.

Onboarding is now persistent and one-step:

- `hivemind-setup {action:"health"}` is the only start-of-use check: one
  static read (filesystem stat + KV read + option parse; zero spawns, zero
  writes) answering `ready`, `action-required` or `blocked` with the one
  remedy. The stored `config/delegate` document is the receipt, so a ready
  setup is never re-walked at session start.
- `/hivemind-setup` asks one question per setting with the host `question`
  tool, in English or Vietnamese, and only runs the doctor, install/sign-in
  or an opt-in probe when health names them.
- The installer accepts `--scope global|project`; entries append after
  existing plugins so a later plugin can override an earlier one, and
  conflicting scopes refuse before any write.
- README and quickstart lead with a plain-language English/Vietnamese
  walkthrough of what you type, what you need and what persists.

Initial public drop of the OpenCode V2 plugin: `/devin` and `/antigravity`
hosted routes with live ACP streaming, session picker and session swap,
permission cards, `@file` mentions, dock overlays, palette commands, and the
`hivemind-delegate`, `delegate-devin`, `devin-children`, and `hivemind-doctor`
agent tools (legacy `devin_run` / `devin_session` retained).

Notable hardening in this drop:

- `auto_approve` on the delegation tools now defaults to **false** and picks
  permission options by label semantics — deny-shaped labels are never
  auto-selected.
- `@` mention containment resolves symlinks before the cwd check.
