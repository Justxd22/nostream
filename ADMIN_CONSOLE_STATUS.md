# Admin Console — Status Audit

**Master issue:** [cameri/nostream#594 — feat: Admin Console for relay operators](https://github.com/cameri/nostream/issues/594) (open, authored by @Justxd22, assigned to @Ferryx349)

**Audit date:** 2026-07-06
**Upstream `main`:** `237b1a4` (v3.0.0; release PR [#658](https://github.com/cameri/nostream/pull/658) for v3.1.0 pending)

The console is planned as 8 phases, each tracked by a sub-issue, plus external NIP dependencies
(NIP-98 auth, NIP-50 search, NIP-89 relay operations) expected from other contributors.

---

## Phase scoreboard

| Phase | Tracking issue | Status | PRs |
|---|---|---|---|
| 1. Admin backend foundation | [#631](https://github.com/cameri/nostream/issues/631) | ✅ **Done** (closed 2026-07-04) | [#641](https://github.com/cameri/nostream/pull/641) merged |
| 2. Live metrics dashboard | [#632](https://github.com/cameri/nostream/issues/632) | 🟡 **In progress (~70%)** | [#653](https://github.com/cameri/nostream/pull/653) merged, [#661](https://github.com/cameri/nostream/pull/661) merged, [#662](https://github.com/cameri/nostream/pull/662) **open**, UI ([#663](https://github.com/cameri/nostream/issues/663)) not started |
| 3. Visual settings editor | [#633](https://github.com/cameri/nostream/issues/633) | 🟠 **Planned** (sub-issues filed) | none yet ([#664](https://github.com/cameri/nostream/issues/664), [#665](https://github.com/cameri/nostream/issues/665) scoped) |
| 4. Moderation tools | [#634](https://github.com/cameri/nostream/issues/634) | ⚪ Not started | — |
| 5. Event search & inspection | [#635](https://github.com/cameri/nostream/issues/635) | ⚪ Not started (**unblocked** — NIP-50 merged) | — |
| 6. Payments & admissions view | [#636](https://github.com/cameri/nostream/issues/636) | ⚪ Not started | — |
| 7. Webhooks & notifications | [#637](https://github.com/cameri/nostream/issues/637) | ⚪ Not started | — |
| 8. System, logs & operations | [#638](https://github.com/cameri/nostream/issues/638) | ⚪ Not started (**blocked** on NIP-89) | — |

### Dependency status

| Dependency | Needed by | Status |
|---|---|---|
| NIP-50 full-text search | Phase 5 | ✅ Merged — [#587](https://github.com/cameri/nostream/pull/587) (2026-07-05, in v3.1.0 release PR) |
| NIP-42 client auth | general | ✅ Merged — [#622](https://github.com/cameri/nostream/pull/622) |
| NIP-98 HTTP auth (replaces password login) | Phase 1 hardening | ❌ **No PR exists upstream yet** |
| NIP-89 relay operations | Phase 8 | ❌ **No PR exists upstream yet** |

---

## What has been achieved (merged to upstream `main`)

### Phase 1 — Admin backend foundation ✅ (PR #641, merged 2026-07-04)

Everything in the Phase 1 checklist from #594 landed:

- `admin.enabled: false` (disabled by default) and `admin.sessionTtlSeconds: 86400` in `default-settings.yaml`
- Protected `/admin` Express router that is feature-gated (`admin-enabled-middleware` returns 404 when disabled)
- Replaceable auth interface `IAdminAuthProvider` (`src/@types/admin.ts`) with a temporary
  `PasswordAdminAuthProvider` (`src/admin/password-admin-auth-provider.ts`) — designed to be swapped for NIP-98 later
- Endpoints: `POST /admin/login`, `GET /admin/session`, `GET /admin/health`
- Admin-specific Redis-backed rate limits (30 req/min admin, 10 login attempts / 15 min) plus admin IP whitelist config
- Session handling (`admin-session.ts`), password utils, health util
- Unit tests: `test/unit/routes/admin.spec.ts`, `admin-rate-limit.spec.ts`, `admin-session.spec.ts`

### Phase 2 — Metrics groundwork ✅ (PRs #653 + #661, merged 2026-06-29 / 2026-07-05)

- OpenTelemetry metrics bootstrap with OTLP export → OTEL Collector sidecar → Prometheus
  (`src/telemetry/metrics.ts`, `otel-collector-config.yaml`, wired into startup/shutdown)
- Handler instrumentation (`src/telemetry/event-metrics.ts`):
  `nostream.events.accepted_total` / `nostream.events.rejected_total` on event OK responses,
  `nostream.websocket.connections` on connect/close

### Related merged work useful to later phases

- NIP-50 full-text search (#587) — `search` REQ filter, GIN index, ranked results → Phase 5's search backend
- NIP-43 invite-code foundation (#650), NIP-70 protected events (#644) — adjacent moderation/admission primitives
- Earlier merged prerequisites referenced in #594: NWC payments processor (#539), event retention purge (#412)

---

## In flight right now

- **PR [#662](https://github.com/cameri/nostream/pull/662) — `GET /admin/metrics` SSE endpoint** (fixes #655), by @Ferryx349.
  Authenticated SSE stream; periodic snapshots from PromQL (EPS, accepted/rejected, active WS connections,
  CPU/RAM per worker) plus DB/Redis health per snapshot; cluster-wide view via Prometheus label aggregation.
  Marked ready for review 2026-07-04, force-pushes addressing Copilot/CodeQL feedback through 2026-07-06.
  No merge conflicts reported. **This is the next merge.**
- **Release PR [#658](https://github.com/cameri/nostream/pull/658) (nostream v3.1.0)** — packages #641, #653, #661, #587,
  #650, #644. Merging it publishes the admin foundation in an official release.

---

## Remaining work (in dependency order)

### Phase 2 — finish live metrics ([#632](https://github.com/cameri/nostream/issues/632))
- [ ] Merge PR #662 (`/admin/metrics` SSE)
- [ ] Observability dashboard UI ([#663](https://github.com/cameri/nostream/issues/663)): admin HTML shell at `/admin/`
      (login/session/logout), static assets at `/admin/assets/`, SSE KPI cards (EPS, rejections, connections,
      CPU, memory, DB/Redis/Prometheus health), Grafana in `docker-compose` with provisioned Prometheus
      datasource + `nostream-overview` dashboard, embedded in `/admin/`
- [ ] Worker count/health surfacing (from #594 checklist; partially covered by `nostream_process_*` gauges)

### Phase 3 — visual settings editor ([#633](https://github.com/cameri/nostream/issues/633))
- [ ] Extract shared settings-config module from `src/cli/utils/config.ts` + export guided field schema from
      the CLI configure TUI ([#664](https://github.com/cameri/nostream/issues/664))
- [ ] `GET /admin/settings` (read current settings as JSON) and `PUT /admin/settings`
      ([#665](https://github.com/cameri/nostream/issues/665))
- [ ] Schema-aware (joi) validation before any write; preview diffs
- [ ] Atomic write to `.nostr/settings.yaml` with timestamped backup before each write
- [ ] Audit log entry for every applied change
- [ ] Trigger the existing hot-reload path so changes apply without restart
- [ ] Settings tab in the dashboard UI

### Phase 4 — moderation tools ([#634](https://github.com/cameri/nostream/issues/634)) — no sub-issues/PRs yet
- [ ] Admin endpoints to read/write pubkey whitelist/blacklist
- [ ] Admin endpoints to read/write IP rate-limit bypass list
- [ ] Admin endpoints to read/write event-kind whitelist/blacklist
- [ ] Views: rejected events, top publishing pubkeys, top client IPs, rate-limit hits, spam/duplicate indicators
- [ ] Ban/unban pubkey and whitelist/unwhitelist IP actions (+ UI)

### Phase 5 — event search & inspection ([#635](https://github.com/cameri/nostream/issues/635)) — **now unblocked**
- [ ] `GET /admin/events` with query params: event id, author pubkey, kind, tag filters, created-at range, limit
- [ ] JSON results + inspection UI
- [ ] Wire NIP-50 full-text search (merged in #587) into admin search

### Phase 6 — payments & admissions view ([#636](https://github.com/cameri/nostream/issues/636)) — not started
- [ ] Total admission fees recorded; pending/paid/expired/failed invoice lists
- [ ] Recent admissions by pubkey; manual invoice lookup by id
- [ ] Payment processor health check; callback failure history
- [ ] Admission whitelist management

### Phase 7 — webhooks & notifications ([#637](https://github.com/cameri/nostream/issues/637)) — not started
- [ ] Targets: Discord, Telegram, Slack, generic HTTP
- [ ] Per-event enable/disable, test-delivery button, configurable retry policy, delivery history
- [ ] Trigger events: invoice created/paid/failed, settings changed, relay restart

### Phase 8 — system, logs & operations ([#638](https://github.com/cameri/nostream/issues/638)) — not started
- [ ] Read-only log tail via `GET /admin/logs`
- [ ] Worker health/count and relay uptime views
- [ ] NIP-89 relay management integration — **blocked: no NIP-89 PR exists upstream yet**

### Cross-cutting / security (from #594)
- [ ] Swap password login for **NIP-98** auth — **blocked: no NIP-98 PR exists upstream yet**
      (console stays disabled-by-default until then; replaceable `IAdminAuthProvider` is already in place)
- [ ] Keep enforcing: no secrets exposure (`.env`, processor creds, NWC URLs), timing-safe comparisons,
      audit records for all mutating actions, HTTPS/reverse-proxy guidance docs
- [ ] Operator documentation for enabling and securing the admin console

---

## Note on this fork

`Justxd22/nostream` `main` is **141 commits behind** `cameri/nostream` `main` (v2.1.0-era vs v3.0.0) and
contains **none** of the admin console work. To build on or test the admin console here, sync the fork with
upstream first (`git fetch upstream && git merge upstream/main` or GitHub's "Sync fork").
