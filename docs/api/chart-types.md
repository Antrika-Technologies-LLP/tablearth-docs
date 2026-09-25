# Chart types

A chart arrives in two places: the `CHART` event of a chat stream (see
[sse-protocol.md](sse-protocol.md#chart-payload-shape)) and each widget of a
dashboard spec (see [endpoints.md](endpoints.md#dashboard-endpoints)). Both carry
a `type` plus the **column keys** that say which result column plays which role.
Every key's value is the name of a column in the rows you received.

## Which types you receive

tableArth.ai only sends chart types your client says it can draw. Send the list in
`chartTypes`:

| Where | How |
|---|---|
| Chat (`/emp/1/api/tableai/chat`, `/chat/v2`) | `"chartTypes": ["bar", "line", "waterfall"]` in the JSON body |
| Widget chat (`/emp/1/api/tableai/widget/chat`) | the same field inside the `data` wrapper |
| `POST /emp/1/api/dashboard/spec` | `"chartTypes": [...]` in the JSON body |
| `GET /emp/1/api/tableai/widget/dashboard/spec` | `chartTypes` parameter, comma-separated |

If you send nothing, you get the **base set** below and nothing else, so an
existing integration keeps working unchanged. A chart that your client didn't
list is replaced by the closest base type it can become (a waterfall becomes a
bar), or left out of a chat answer, whose text already carries the numbers.

## Base set

These are the types every client has always received.

| `type` | Chat keys | Dashboard keys | Rows |
|---|---|---|---|
| `bar` | `xKey`, `yKey` | `labelKey`, `valueKey` | one per category |
| `line`, `area` | `xKey`, `yKey` | `xKey`, `yKey` or `series[]` | one per x value |
| `pie`, `donut` | `xKey`, `yKey` | `labelKey`, `valueKey` | one per slice |
| `scatter` | `xKey`, `yKey` | `xKey`, `yKey` | one per point; both numeric |
| `stacked_bar` | `xKey`, `yKey`, `seriesKey` | `labelKey` + `series[]`, or `xKey`, `yKey`, `seriesKey` | long: one per (x, series) |
| `funnel` | `xKey`, `yKey` | `labelKey`, `valueKey` | one per stage, in order |
| `table` | — | — | any columns |

Dashboards also use `kpi`, `hbar`, `grouped_bar`, `grouped_hbar`,
`stacked_hbar`, `combo`, `gauge`, `treemap`, `heatmap`, `ranked_list` and
`stat_grid`; their keys follow the same pattern.

## Added types

Request these by name in `chartTypes`.

| `type` | Keys | Rows | Draw it as |
|---|---|---|---|
| `waterfall` | `labelKey`, `valueKey` | one per step, in order; the value is the signed change at that step | bars floating from the running total before the step to the one after, plus a total bar (unless `showTotal` is `false`) |
| `sankey` | `sourceKey`, `targetKey`, `valueKey` | one per (source, target) pair | flows between nodes; a source never equals its target, and the flows never loop back |

## Common problems

- **A chart you expected is missing from a chat answer** — your client didn't list
  its type in `chartTypes`, and it had no base-set equivalent. The answer text
  still contains the result.
- **Keys don't match any column** — compare the key names without regard to
  case: some databases return column names in upper case.
