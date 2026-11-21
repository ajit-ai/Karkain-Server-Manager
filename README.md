# Karkain Server Manager

A CLI/TUI-first server inventory, service and deployment-management service built exclusively on Karkain Language v1.2.0.

## Status

| Item | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain Server Manager maintains a server inventory, declares the services that should run on each host, records deployment runs and collects local logs into stable text/JSON facts for scripting. It is a per-host administration and automation tool, not a network device manager, not an API gateway and not a network monitor. It is deliberately scoped to inventory, declared state, scripted deployment hooks and log collection; native process enumeration and OS resource metrics are NOT available in Karkain v1.2.0 and are recorded as blockers, not claimed as features.

## Who it serves

It serves CLI-first operators, mixed Linux/Windows estates, and air-gapped or text-only environments wanting lightweight, stateless management facts without a browser GUI or an agent control plane. Engineers who prefer scriptable remediation hooks over a web console are the primary audience.

## Planned features

All items below are **(planned)** and are specification targets, not implemented behaviour.

- **Server inventory (planned):** host records with role, environment, owner, endpoint and credential reference; import/export.
- **Service registry (planned):** declared services, desired state, dependencies and a text fact view; actual OS service control is BLOCKED.
- **Deployment runs (planned):** versioned artifacts, scripted hooks, recorded steps and rollback notes; no live process control.
- **Log collection (planned):** read and filter local text logs via `io`; structured text/JSON output.
- **Resource facts (planned, constrained):** best-effort facts only through PARTIAL `system`/`env` shell wrappers, which must be probed; native CPU/memory/process metrics are BLOCKED.
- **Remote admin channel (planned):** TCP client transport only; TLS is BLOCKED, so no secure remote claim is made.
- **Fact/command API (planned):** stable text/JSON facts and scriptable remediation hooks.
- **CLI/TUI admin (planned):** declarative config, text/JSON output and no GUI.

## Explicit non-goals

- Not a network device manager (Karkain Network Manager).
- Not an API gateway (Karkain API Manager).
- Not a network performance monitor (Karkain Network Monitor).
- No native GUI, no TLS, no WebSocket, no concurrent multi-host agent service, no native OS process/resource metrics.
- No modification of the Karkain language, compiler or standard library.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

- Windows x86-64 toolchain: **VERIFIED** (Karkain v1.2.0 compiles and runs a native executable on this host).
- Linux cross-target output: **PRESENT but execution NOT VERIFIED** (cross-compilation output exists; it was not executed on this host).
- macOS: **NOT VERIFIED**.

## Build and run status

No application binary exists yet. The intended generic commands, which become active when implementation starts, are:

```
karkain build src/main.kark -o <name>
karkain run src/main.kark
```

These are placeholders for the planned source layout, not verified commands for this application.

## Known blockers

- **OS process/resource metrics APIs** — no native CPU/memory/process enumeration; only PARTIAL shell wrappers that must be probed (BLOCKED/PARTIAL).
- **TLS** — no secure remote administration transport (BLOCKED).
- **Concurrent HTTP server framework** — no multi-host agent service (BLOCKED).
- **WebSocket** — no live browser/remote console (BLOCKED).
- **GUI** — no native GUI toolkit; GUI is a planned, blocked layer.
- **Filesystem metadata / system wrappers** — PARTIAL and portability UNVERIFIED.

## Repository

- Source: https://github.com/ajit-ai/Karkain-Server-Manager
- Issues: https://github.com/ajit-ai/Karkain-Server-Manager/issues
- License: Apache-2.0
