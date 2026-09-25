# Changelog

## Unreleased

### 2026-09-26 — Distribution, hierarchy and flow charts

- **New chart types:** `sunburst`, `graph` and `radar` (chat and dashboards);
  `histogram`, `box_plot`, `ttest_table`, `icicle`, `chord`, `tree`,
  `parallel`, `word_cloud`, `calendar`, `horizon` and `gantt` (dashboards);
  nested `treemap` with `levels[]`.
- **Server statistics:** histogram, box-plot and t-test widgets arrive already
  aggregated in the widget data.

### 2026-09-26 — Bubble and rose charts, display options

- **New chart types:** `bubble` (chat and dashboards) and `rose` (dashboards),
  in the [chart types](docs/api/chart-types.md) reference.
- **Display options** documented for dashboard widgets: curve, stacking
  (including 100%), value labels, legend position, axis bounds, sort, a second
  y axis and gauge bands.
- **Server-side transforms:** top N with "Others", running totals, moving
  averages, % change and contribution arrive already applied in the widget data.

### 2026-09-26 — KPI and table chart types

- **New chart types:** `pivot_table` and `time_table` (chat and dashboards),
  `kpi_trend`, `kpi_compare`, `bullet` and `text` (dashboards), each with its
  column keys in the [chart types](docs/api/chart-types.md) reference.
- **Several tables per dashboard** for clients that list `pivot_table`; other
  clients keep the single `table`.
- **Server-side table sort, search and totals:** the widget-data routes accept
  `sortKey`, `sortDir`, `search`, `searchColumns` and `totals`, and return
  `totals`, `compareData` and `compareLabel`.
- **Dashboard spec example corrected:** widgets sit under `dashboard`, and a
  single table under `table`, not `tables`.

### 2026-09-26 — Chart types beyond the base set

- **New `chartTypes` request field** on chat, widget chat and the dashboard-spec
  routes. Clients list the chart types they draw; without it they keep receiving
  only the base set.
- **Waterfall and Sankey** charts, with their column keys, in the new
  [chart types](docs/api/chart-types.md) reference.
- **`chartData` example corrected:** it is an array of row objects, not
  `{ x: [...], y: [...] }`.

### 2026-06-15 — Accuracy pass against shipped code

- **Rebrand:** product is now **tableArth.ai** throughout (HTTP paths keep the
  `tableai` slug).
- **Removed the Excel add-in** docs — it is a pre-rebrand prototype, not a shipped
  surface.
- **Widget auth rewritten** to the shipped model: the host mints an SSO JWT and
  calls `window.antrika('syncUser', …)` to establish the `antrikaSSO` session; the
  widget signs each request internally with `widget_id`. Removed the non-existent
  `ssoToken` render argument and `setSsoToken` helper.
- **Endpoints corrected:** `apikey/validate` → `extension/validate` (+ new
  `extension/domains`); API-key `chat` body is flat (only the widget wraps it in
  `data`); upload returns `tableFileId` / `dashboardSpec` / `remoteSessionId`;
  history items use `message` (not `answer`); added the widget dashboard routes and
  the `/v2` per-user API routes.
- **SSE:** documented the `STATUS` event and the real `CHART` payload
  (`chartConfig` + `chartData`, JSON strings).
- **Auth/admin:** widget signs with the tenant **SSO signing key** (not the agent
  API key); feature flags documented (`agent-data-query`, `apiAccess`,
  `widgetIntegration`, `singleSignOn`); session `source` labels
  (`API`/`Widget`/`Extension`/`Admin`).
- **SSO token examples** (Node, Java) now mint the `syncUser` token (claim `id`,
  not `sub`).

### Initial set

- `README.md` with surface picker (widget / Chrome extension / REST).
- `docs/concepts.md` — agents, sessions, `remoteId`, lifecycle.
- Integration guides for the JS widget, Chrome extension, and server-to-server REST.
- API reference: auth modes, endpoint reference, SSE protocol.
- Admin guide for one-time agent setup.
- Troubleshooting and SSO-token-minting examples (Node, Java).
