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
| `pivot_table` | `rowKeys[]`, `columnKeys[]`, `metrics[]` (`{ key, label, format }`) | long: one per (row values, column values) group | a cross-tab: `rowKeys` as row headers, `columnKeys` across the top, each metric summed in the cells (`aggregate` may be `avg`, `count`, `min` or `max`), with row and column totals |
| `time_table` | `rowKey`, `xKey`, `valueKey` | long: one per (entity, period) | one row per entity: its latest value, a sparkline, the change against `compareLag` periods back (default 1) and the average |
| `bubble` | `xKey`, `yKey`, `sizeKey`, `labelKey` (optional), `seriesKey` (optional) | one per entity | points at (x, y) whose area grows with `sizeKey`, coloured by `seriesKey` |

These are dashboard-only:

| `type` | Keys | Rows | Draw it as |
|---|---|---|---|
| `kpi_trend` | `xKey`, `valueKey` | one per period, in order | the latest value with its change and a sparkline; with `aggregate` `sum` or `avg`, the total or average of all periods and no change |
| `kpi_compare` | `valueKey`, `previousKey` (optional) | one | the value with its change against `previousKey`; without it, the widget data carries the comparison rows in `compareData` (see [endpoints.md](endpoints.md#post-emp1apidashboardwidgetdata)) |
| `bullet` | `labelKey`, `valueKey`, `targetKey` (or a fixed `target`) | one per bar | a bar against its target, over bands at the `ranges` percentages of the target (default 50, 80, 100) |
| `rose` | `labelKey`, `valueKey` | one per category | a Nightingale rose: a pie whose petals grow with the value (`roseType` `radius` or `area`) |
| `text` | none; `template` holds Markdown | none, or any | the Markdown, where `{{column}}` is the first row's value and `{{#rows}}…{{/rows}}` repeats once per row; render it without raw HTML |

## Display options

Dashboard widgets may carry display options next to their keys. Draw what you
support and ignore the rest; the data is the same either way.

| Option | Types | Meaning |
|---|---|---|
| `curve` | `line`, `area`, `combo` | `smooth` (default), `straight` or `step` |
| `stack` | `area`, bar types | `none`, `stack`, `expand` (each bar or point as a % of its total) or, for `area`, `stream` |
| `labels` | most types | `true` prints each value on its mark; for `pie`, `donut` and `rose` a mode: `name`, `value`, `percent`, `name_percent` |
| `legend` | charts with a legend | `top`, `bottom`, `right` or `none` |
| `yMin`, `yMax`, `logScale` | cartesian types | value-axis bounds and a log scale |
| `sort` | bar types | `value_desc`, `value_asc` or `label` |
| `axis: "right"` | `combo` `bars[]` / `lines[]` items | draw that series on a second y axis |
| `min`, `max`, `bands`, `style` | `gauge` | the scale, coloured `[{ to, color }]` bands, `progress` ring or `pointer` dial |

Top N, running totals, moving averages, % change and contribution are applied
on the server: the widget data already holds the transformed rows.

## Tables

A client that lists `pivot_table` in `chartTypes` can hold any number of tables:
they arrive in `charts` with `type: "table"`, placed on the grid like any other
widget. A client that doesn't gets at most one table, in `table`. A table's
optional `config` carries `columns` (`[{ key, label, format, hidden }]`) and
`totals` (`true` for a totals row).

## Common problems

- **A chart you expected is missing from a chat answer** — your client didn't list
  its type in `chartTypes`, and it had no base-set equivalent. The answer text
  still contains the result.
- **Keys don't match any column** — compare the key names without regard to
  case: some databases return column names in upper case.
- **Only one table shows** — your client didn't list `pivot_table`, so it gets
  the single `table` slot. List it once you render tables from `charts`.
