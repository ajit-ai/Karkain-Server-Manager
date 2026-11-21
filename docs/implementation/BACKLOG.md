# Karkain Server Manager — First Implementation Backlog

## First vertical slice

**Slice goal:** Register two servers from an explicit text config and list them as deterministic text facts. This is real work — config parsing, validation, `db2` persistence and text rendering — and avoids every BLOCKED capability.

**Karkain modules/files expected:** `src/main.kark` (command dispatch), `src/config/loader.kark` (config read/validate), `src/inventory/registry.kark` (CRUD), `src/storage/store.kark` (`db2` init/insert/select), `src/ui/fact_output.kark` (text/JSON facts), `configs/default.kark`, `examples/servers.kark` (sample config).

**Exact input:** `examples/servers.kark` declaring two hosts: `id = "web-01"`, `role = "web"`, `environment = "prod"`, `endpoint = "10.0.1.10"` and `id = "db-01"`, `role = "db"`, `environment = "prod"`, `endpoint = "10.0.1.11"`. Commands: `karkain run src/main.kark load examples/servers.kark` then `karkain run src/main.kark servers list`.

**Exact expected observable output:** two lines, each containing id, role, environment and endpoint in input order, followed by `servers: 2`. An invalid host record is reported with its position and no record is inserted. A missing optional field is rendered as `unknown`, never as a fabricated value.

**Verification command pattern (Karkain v1.2.0):** with the VERIFIED Go front-end engine and C-free Windows target selected, run `karkain lint src/main.kark`, then `karkain build src/main.kark -o srv-mgr-slice` (or `karkain run src/main.kark ...`), execute the binary and compare stdout to the expected fixture. Commands become active only when implementation starts.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| SRV-001 | Config loader + validation | `src/config/loader.kark` | config, io | Valid config loads; invalid field reported with position | Unit | M | None | Lint-clean; fixture passes |
| SRV-002 | Server CRUD + validation | `src/inventory/registry.kark`, `src/storage/store.kark` | db2, SRV-001 | 2 records list correctly; duplicate id rejected | Unit, integration | M | None (db2 PRESENT) | Inventory fixture passes |
| SRV-003 | Text/JSON fact output | `src/ui/fact_output.kark` | SRV-002 | Identical data in both modes; deterministic order | Unit, E2E | S | json PARTIAL for JSON mode | Fact fixtures pass |
| SRV-004 | Service registry + desired state | `src/services/registry.kark` | SRV-002 | Services resolve to a server; desired state query correct | Unit, integration | M | OS service control BLOCKED — declared state only | Service fixture passes |
| SRV-005 | Deployment run records | `src/deploy/runs.kark` | SRV-002, db2 | Versioned run recorded with steps and rollback note | Integration, regression | M | Live process control BLOCKED | Deployment fixture passes |
| SRV-006 | Scripted deployment hooks | `src/deploy/hooks.kark` | SRV-005, io | Hook invoked and result captured as text; failure recorded | Integration | L | process/system PARTIAL — probe before use; no OS control claimed | Hook fixture passes |
| SRV-007 | Local log collection + filter | `src/logs/collect.kark` | SRV-001, io | 10k-line log filtered to expected JSON fact set | Unit, performance | M | None (io PRESENT) | Log fixture passes |
| SRV-008 | Fact/command API | `src/facts/api.kark` | SRV-003, SRV-004 | Stable fact keys; remediation hook emits one event | Integration | M | TLS BLOCKED — trusted networks only | API fixture passes |
| SRV-009 | Resource fact probe | `src/facts/resources.kark` | SRV-008, system/time PARTIAL | PARTIAL wrappers probed; failures recorded as `unknown`, never fabricated | Security, platform-specific | L | OS metrics/process APIs BLOCKED; system/time PARTIAL | Probe report written; no false metrics |
| SRV-010 | Plain TCP remote channel | `src/remote/channel.kark` | SRV-008, net | Unreachable host yields structured failure after timeout | Failure/recovery | L | TLS BLOCKED; WebSocket BLOCKED — no live console | Failure path leaves state intact |
| SRV-011 | Workspace export/import | `src/inventory/workspace.kark` | SRV-002, SRV-005, SRV-007 | Export→import equals original | E2E, compatibility | L | json PARTIAL | Round-trip equivalence |
| SRV-012 | Native slice verification harness | `tests/slice_verification.kark`, `scripts/verify.ps1` | SRV-002, SRV-003 | Build produces executable; stdout matches fixture; exit 0 | Actual-compiled-executable | M | Default kcc engine BLOCKED — use Go engine + C-free target | Harness passes end-to-end |

## Dependency order and gates

1. SRV-001 → gate: loader fixture passes. 2. SRV-002 → SRV-003. 3. SRV-004, SRV-005 → SRV-006. 4. SRV-007. 5. SRV-008 → SRV-009 (probe before any metric claim) → SRV-010. 6. SRV-011 → SRV-012. No item proceeds until its gate passes and dependencies are lint-clean.

## Status

All items **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run for this application; no application binary exists; all BLOCKED capabilities (OS metrics/process APIs, TLS, WebSocket, concurrent agent service, GUI) remain blockers and must not be claimed as solved.
