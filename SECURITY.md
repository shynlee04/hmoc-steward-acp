# Security

## Reporting

Open a private report via GitHub → Security → "Report a vulnerability", or file
a public issue without reproducing sensitive details. Do not post session IDs,
tokens, or captured wire data in reports.

## Threat model

This plugin spawns `devin acp` as a subprocess and relays its events into the
OpenCode TUI. Trust boundaries that matter:

- **Credentials stay with the Devin CLI.** The plugin never reads, stores, or
  transmits the Devin credentials store or any token — auth is
  `devin auth login` on the host, and auth failures surface as status text.
- **`devin_session` is a delegation surface.** It runs an
  unattended agent with the ambient process env in a cwd the caller chooses.
  `auto_approve` defaults to **false**; when set true, permission requests are
  answered by picking the first option whose label or `kind` is allow-shaped —
  deny-shaped labels are excluded first, and ambiguous sets resolve
  `cancelled`, never option zero. Grant `auto_approve` only for workloads you
  already trust.
- **`@file` mentions are containment-checked against realpath.** A symlink that
  escapes the lane cwd never becomes a resource link; the URI sent to Devin is
  the resolved target path.
- **Subprocess sandboxing is Devin's.** The plugin does not add a sandbox of
  its own; Devin's own permission model governs what the agent does.
- **The public repo is docs-only.** It ships user documentation, not the plugin
  source, tests, or CI. No real session captures, screenshots, or account data
  are published here.
