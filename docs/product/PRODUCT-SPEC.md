# Karkain Server Manager — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name:** Karkain Server Manager.

**Purpose and problem:** Operators of small or mixed Linux/Windows estates need a lightweight, text-native way to record servers, declare intended services, run scripted deployments and collect logs. Incumbent tools are browser GUIs or gateway services with deep OS integration. Karkain v1.2.0 lacks native process and resource-metric APIs, TLS and a concurrent server framework, so this product is deliberately scoped to inventory, declared state, scripted hooks and local log facts — with those gaps recorded as blockers.

**Vision:** A stateless, scriptable per-host management workspace whose output is stable text/JSON facts that other systems can consume, implemented solely in Karkain v1.2.0.

**Intended users and segments:** CLI-first system administrators; operators of text-only or air-gapped sites; small teams managing mixed Linux/Windows hosts; DevOps engineers wanting deployment records without an agent control plane.

**Usage scenarios:** (1) Register a server and declare its services and desired state. (2) Record a deployment run with artifact version and rollback note. (3) Collect and filter a local text log into JSON facts. (4) Query declared service state as text. (5) Connect to a host over a plain TCP client channel for scripted commands — without TLS, so limited to trusted networks.

**Market positioning:** A lightweight, CLI-first complement to full server-management GUIs, focused on declared configuration and log facts. Not a fleet orchestrator.

**Primary value proposition:** Stable text/JSON facts and scriptable remediation hooks with no browser GUI and no state duplication — while honestly excluding native OS metrics, secure remote transport and multi-host concurrency until cleared at language level.

**Explicit non-goals and boundaries:** No network device management, API gateway or network monitoring. No native GUI, TLS, WebSocket, concurrent multi-host agent service or OS process/resource metrics. No language/compiler modification.

## B. Functional architecture

| Module | Responsibility |
|---|---|
| Server Inventory | CRUD for host records (id, role, environment, owner, endpoint, credential reference). |
| Service Registry | Declared services, desired state, dependencies and text fact view. |
| Deployment Runs | Versioned artifacts, scripted hook records, step results and rollback notes. |
| Log Collection | Read/filter local text logs via `io`; emit structured text/JSON facts. |
| Resource Facts | Best-effort facts via PARTIAL shell wrappers; must be probed; native metrics BLOCKED. |
| Remote Channel | Plain TCP client transport for scripted commands; TLS BLOCKED. |
| Fact/Command API | Stable text/JSON facts and remediation hooks for external automation. |
| Config Loader | Declarative text config via `config`/`io`; validated before use. |
| CLI/TUI Frontend | Command parsing, text/JSON output. |

**Core workflows:** Register server → declare services → record deployment → collect logs → query facts.

**Admin workflows:** Edit host metadata, change declared desired state, review deployment history, export inventory, configure retention.

**Error/recovery workflows:** Unreachable host (TCP 5 s read timeout) yields a structured failure event; a failed deployment step records a rollback note and leaves prior records intact; a missing log file is reported without crashing; a PARTIAL shell wrapper that fails is recorded as "unknown fact", never a fabricated metric.

**Roles and permissions:** Local roles (read, edit, deploy, admin) gate inventory and deployment changes; no remote authentication is claimed because TLS is BLOCKED.

**Configuration/customization:** File-based declarative config; no database server required.

**Import/export:** Text/simple JSON inventory and deployment-history export; `json` is PARTIAL, so only simple field extraction is assumed.

**Integration requirements:** Emits text/JSON facts for external automation; integrations requiring TLS are BLOCKED.

## C. Competitor analysis

Researched competitors and sources are in `docs/competitor-analysis/COMPETITORS.md`, derived from portfolio research. Evidence labels: **[VF]** verified fact, **[RI]** research-based interpretation, **[PROP]** Karkain proposal, **[BLOCKED]** unattainable with v1.2.0, **[UA]** unverified assumption.

**Competitors:** Cockpit and Microsoft Windows Admin Center (WAC), with Webmin-style browser GUIs as context.

**[VF]** Cockpit is a free open web interface for individual Linux servers using existing system APIs so CLI and GUI work side by side; WAC is Microsoft's free browser-based remote manager for Windows Server (physical, virtual, on-prem, Azure or hosted). **[RI]** Both are per-host administration GUIs, not fleet orchestrators. **[VF]** Cockpit offers health graphs, logs, services, network/firewall, storage, VMs, containers, updates and a terminal; WAC built-ins include Certificates, Events, Files, Firewall, Local users, Networks, Performance Monitor, PowerShell, Processes, Services, Storage, Updates and VMs/switches.

**Feature comparison matrix (Karkain = proposed):**

| Capability | Karkain (proposed) | Cockpit | Windows Admin Center |
|---|---|---|---|
| CLI/TUI-first (GUI **[BLOCKED]**) | **[PROP]** yes | Browser GUI + terminal | Browser GUI + PowerShell |
| Cross-platform Linux+Windows **[PROP]** | Text facts on both targets (execution NOT VERIFIED) | Linux hosts only | Windows/Azure focus |
| Text/JSON automation output | **[PROP]** stable facts | Web UI + CLI parity | Web UI + PowerShell |
| Lightweight stateless service | **[PROP]** yes | Yes (socket-activated) | Gateway service required |
| Secure remote transport **[BLOCKED]** | No | SSH/HTTPS | HTTPS + gateway |
| Native OS metrics/processes **[BLOCKED]** | No | Yes | Yes |
| Extension model | Scripted hooks/text config | `cockpit-*` packages | NuGet tool/solution/gateway plugins |

**Platform/deployment comparison [VF]:** Cockpit runs on Fedora, RHEL, CentOS, Debian, Arch, Ubuntu, openSUSE, SUSE Micro and CoreOS with browser access on port 9090; WAC uses a local gateway plus browser, with an Azure-portal variant and NuGet extensions. Karkain is planned as a single Karkain-built CLI/TUI service on the VERIFIED Windows x86-64 toolchain, with other targets unverified.

**Licensing comparison (verified):** **[VF]** Cockpit is free, GNU LGPL v2.1+ (its repository also lists GPL-3, BSD-3, CC-BY-SA-3 and MIT). **[VF]** Windows Admin Center is offered "at no additional cost". Karkain Server Manager is Apache-2.0. These are the verified licence facts; no unverified licence is asserted.

**Pricing [VF]:** Cockpit is free; WAC is at no additional cost. No prices are invented.

**Strengths/limitations [VF]/[RI]:** Cockpit is zero-cost with no duplicated state and CLI parity but limited to Linux hosts; WAC has deep Windows/Azure tooling but is Windows-centric and subject to extension churn. Karkain's strength is text-native automation output; its limitation is the BLOCKED OS metrics and secure transport, which prevent parity with either GUI's live host view.

**Market gaps [RI]:** Both incumbents require a browser GUI; little exists for CLI/TUI-first operators, mixed Linux/Windows estates or text-only, air-gapped management.

**Karkain differentiation [PROP]:** CLI/TUI-first, service-based server manager producing stable text/JSON facts and scriptable remediation hooks. **[BLOCKED]** no GUI, no concurrent server framework (no multi-host agent service), no TLS (no secure remote transport), no WebSocket (no live console).

**Features deliberately not copied:** Browser dashboards/graphs, systemd/Windows service control, interactive terminal/remote desktop, VM and container management, NuGet/cockpit extension ecosystems, and Azure hybrid tooling.

## D. Technical architecture

**Application architecture:** A single process with command dispatcher, domain modules (Inventory, Service Registry, Deployment, Logs) and text/db2 persistence; a foreground "service mode" is a CLI loop, not a daemon.

**Component boundaries:** Inventory, Services, Deployment, Logs and Remote Channel are separate modules. OS-facing calls are isolated in Resource Facts and Remote Channel so PARTIAL/BLOCKED capabilities remain confined.

**Process/service boundaries:** One process, one sequential command path. No multi-host agent, worker pool or daemon is claimed.

**UI architecture:** The UI is CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED). GUI is a planned layer contingent on ecosystem capability and is not in the MVP.

**Domain model:** Server, Service, DesiredState, DeploymentRun, LogSource, FactEvent, Role.

**Data storage strategy:** Text config for declared state plus embedded `db2` for deployment history and fact events; no central database.

**API/interface architecture:** Internal Karkain module calls; external interaction via CLI arguments and text/JSON output. TCP is a plain client channel only (no TLS).

**Event/message architecture:** In-process fact events appended to a local log; no broker assumed.

**Logging/diagnostics/observability:** Structured text/JSON fact lines to stdout and an append-only event log; `karkain lint`/`check --format=json` and the `test` runner provide diagnostics.

**Configuration management:** Declarative text via `config`/`io`, validated before activation.

**Plugin/extension architecture:** Scripted hooks and text config are the extension point; no dynamic plugin loading. Any FFI/OS integration must be a Karkain-compatible native interface, never another language; none is required for MVP.

**Explicit blockers:** OS process/resource metrics, TLS, WebSocket, concurrent server framework, GUI and filesystem metadata are BLOCKED/PARTIAL and must not be claimed as solved.

## E. Data design

**Main entities:** Server, Service, DesiredState, DeploymentRun, LogSource, FactEvent, Role.

**Relationships:** Server has many Services and DeploymentRuns; Service has one DesiredState; DeploymentRun references a Server; FactEvent references a Server and optional LogSource.

**Ownership:** Inventory owns Server; Service Registry owns Service/DesiredState; Deployment owns DeploymentRun; Log Collection owns LogSource; Fact/Command API owns FactEvent.

**Validation:** Required host fields enforced; duplicate ids rejected; service references must resolve to a server; deployment artifacts must have a version; failed PARTIAL wrapper output is recorded as "unknown", never fabricated.

**Lifecycle:** Server active→retired; Service declared→applied→disabled; DeploymentRun started→succeeded/failed→rolled-back; FactEvent immutable.

**Persistence:** Text config for declared state; embedded `db2` for deployment/fact records. `db_begin`/`db_commit`/`db_rollback` exist; crash-atomicity UNVERIFIED.

**Transactions:** Declared-state changes staged then swapped; atomic rename UNVERIFIED — recorded as a constraint.

**Backup/restore:** Export config tree plus `db2` dump; restore into a fresh workspace and revalidate.

**Migration:** Schema version recorded from start; no pre-v1.2.0 compatibility required.

**Import/export formats:** Text and simple JSON; no full JSON schema dependency.

**Audit history:** Append-only event with actor/target/timestamp for every mutation; file-permission protection PARTIAL.

## F. Karkain implementation architecture

**Required capabilities:** Compiler v1.2.0 only; `io`, `config`, `db`/`db2`, `net` (client), `json` (PARTIAL), `string`, `collections`, `crypto`, `encoding`, `log`, `time` (PARTIAL), `system`/`env`/`process` (PARTIAL), `filesystem` (PARTIAL).

**Compiler/runtime requirements:** Karkain v1.2.0; the VERIFIED Windows x86-64 C-free backend with the Go front-end engine is the only currently working compile/run path on this host.

**stdlib modules actually present (per capability register):** `io` PRESENT (`io_read_all`, `io_read_lines`, `io_write_file`, `io_append_file`, `io_list_dir`, `io_run`); `db` PRESENT/PARTIAL; `net` PRESENT (5 s read timeout); `crypto` PRESENT; `encoding` PRESENT; `json` PARTIAL; `filesystem` PARTIAL (shells out to PowerShell); `system`/`env`/`process`/`time` PARTIAL (mostly `system("powershell -c ...")` wrappers; `get_args()` returns an empty array; string-returning wrappers may be non-functional and MUST be probed before any product claim).

**Required ecosystem components:** None external for MVP; Git may be used externally for config diffing but is not a runtime dependency.

**Missing dependencies:** Native OS process/resource metrics, TLS, WebSocket, concurrent HTTP server framework, portable filesystem metadata — all absent.

**Native implementation opportunities:** Karkain-native declarative state diff, log filtering and text fact rendering using `string`/`collections`/`io`.

**FFI/OS integration needs:** None justified for MVP. Any future OS integration must be a Karkain-compatible native interface, never a replacement language. The PARTIAL shell wrappers are a hint of future work, not a delivered capability.

**Build/packaging:** Single Karkain binary plus text config tree; `karkain build`/`karkain run` with the C-free target.

**Explicit blockers (each a BLOCKER, not solved):** no OS process/resource metrics (cannot deliver native CPU/memory/process facts); no TLS (no secure remote admin); no concurrent server framework (no multi-host agent service); no WebSocket (no live console); no GUI; filesystem/system PARTIAL and portability UNVERIFIED. No workaround may change Karkain or introduce another language.

## G. Non-functional requirements

| Target | Workload + measurement method |
|---|---|
| Performance/latency | Filter a 10,000-line log into JSON facts; measure wall-clock via a Karkain-native timing harness (PARTIAL `time` — probe first). |
| Memory | Load 100 servers + 500 services; measure process RSS with an external OS tool. |
| CPU | Run the 10,000-line filter; measure elapsed CPU via `karkain prof` where applicable. |
| Scalability | 100 servers/500 declared services; confirm fact queries stay bounded. |
| Reliability | Simulate unreachable host and failed deployment step; confirm clean failure, prior records intact. |
| Security | Reject path traversal in log paths and oversized input; TLS absent — transport security NOT provided. |
| Privacy | Credential references stored as names, never plaintext secrets in exported facts. |
| Portability | Build with C-free Windows target; cross-compile for Linux and record execution NOT VERIFIED; probe PARTIAL shell wrappers. |
| Maintainability | Each module lint-clean with documented interfaces. |
| Accessibility | All output text/JSON; no GUI dependency. |
| Observability | Every mutation emits a structured fact event. |
| Recovery | Restore config + `db2` dump into empty workspace and revalidate. |

Practical machine constraint: ~4 GB RAM; workloads sized accordingly.

## H. Compatibility and portability

**Supported OS:** Windows x86-64 toolchain VERIFIED; Linux cross-target output PRESENT but execution NOT VERIFIED; macOS NOT VERIFIED. **Architectures:** x86-64 for the VERIFIED target; others unverified. **Karkain v1.2.0 compatibility matrix:** v1.2.0 only. **Binary/package formats:** single native executable plus text config tree. **Install/uninstall:** copy/unpack binary and config; uninstall by removal. **Config migration:** schema version field planned; no pre-v1.2.0 compatibility required. **Data migration:** documented text/JSON export/import. **Backward compatibility:** none required before v1.2.0. **Upgrade/rollback:** keep prior binary/config; rollback by restore + revalidate. **Cross-platform differences:** `filesystem`/`system`/`process` shell-outs vary and are UNVERIFIED. **Unsupported:** native OS metrics/process enumeration, TLS, WebSocket, concurrent serving, GUI.

## I. Testing strategy

All tests are **PLANNED**; none have been run for this application.

- **Unit:** State diff, log filter, fact renderer; criterion — known inputs yield exact outputs.
- **Integration:** Register server → declare service → record deployment → collect log → query fact; criterion — one event per mutation.
- **E2E:** Export/import inventory and deployment history; criterion — fixtures match.
- **Platform-specific:** C-free Windows build executes (exit 0); Linux/macOS execution NOT VERIFIED; PARTIAL wrappers probed before use.
- **Security:** path traversal/oversized input rejected; criterion — no file written outside workspace.
- **Performance:** 10,000-line log filter within G workload; bounded memory.
- **Regression:** prior fixes remain correct.
- **Compatibility:** lint/build on v1.2.0; zero errors.
- **Actual-compiled-executable:** build with `karkain build`, run binary, exit 0 and expected output.
- **Failure/recovery:** unreachable host and failed deployment; clean failure, prior records intact.

## J. Implementation roadmap

**MVP:** Server inventory, config loader, CLI/TUI, fact output; gate — first vertical slice compiles and runs.

**Core functional release:** Service Registry, Deployment Runs, Log Collection; gate — unit/integration/E2E criteria pass.

**Advanced release:** Resource Facts (only after probing PARTIAL wrappers), remediation hooks, retention; gate — security/recovery criteria pass and no fabricated metrics.

**Production-readiness release:** hardening and platform probes; gate — all G targets measured and BLOCKED capabilities either resolved at language level or still explicitly documented.

**Feature dependencies:** Services/Deployments depend on Inventory; Log facts depend on `io`; Resource Facts depend on a passing PARTIAL-wrapper probe; secure remote depends on a future TLS capability.

**Milestones/gates:** each release proceeds only after its gate; no fictional dates or completion claims. **Risks/mitigations:** BLOCKED primitives kept as blockers (never faked); PARTIAL modules probed before reliance; single-threaded design documented as a constraint.
