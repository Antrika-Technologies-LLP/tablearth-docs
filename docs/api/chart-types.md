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
| `sunburst` | `levels[]` (column names, top level first), `valueKey` | long: one per leaf | rings: the first level innermost, each ring split by the next level |
| `graph` | `sourceKey`, `targetKey`, `valueKey` (optional) | one per link | a network: nodes sized by their total weight, links between them |
| `radar` | `labelKey`, `metrics[]` (`{ key, label }`) | one per entity, 3–8 metrics | one shape per row across one axis per metric |
| `pareto` | `labelKey`, `valueKey` | one per category; amounts not negative | bars largest first, with the running share of the total as a line on a 0–100% axis and a guide at `threshold` percent (default 80) |
| `lollipop` | `labelKey`, `valueKey` | one per category | a thin stem from zero to each value with a dot at its end, across (default) or up (`orientation`), sorted like bars |
| `dumbbell` | `labelKey`, `seriesKey`, `valueKey` | long: one per (category, group) | a dot per group joined from the lowest to the highest, widest gap first |
| `bar_in_bar` | `labelKey`, `valueKey`, `targetKey` | one per category | the value as a slim bar inside a wider bar for the target |
| `butterfly` | `labelKey`, `seriesKey`, `valueKey` | long: one per (category, group), exactly two groups | the first group's bars running left of a centre line and the second's right, on one scale |
| `diverging_bar` | `labelKey`, `seriesKey`, `valueKey` | long: one per (item, level), in the scale's order | the first half of the levels stacked left of zero and the second half right; with an odd count the middle level is neutral and split across zero |
| `slope` | `xKey`, `seriesKey`, `valueKey` | long: one per (period, entity) | a line per entity from its value in the first period to the last |
| `bump` | `xKey`, `seriesKey`, `valueKey` | long: one per (period, entity) | each entity's rank within each period (highest value first), rank 1 at the top |
| `radial_bar` | `labelKey`, `valueKey`, `seriesKey` (optional) | one per category | bars bent around a circle, the largest on the outside |
| `waffle` | `labelKey`, `valueKey` | one per category | a 10 × 10 grid of squares, one per percent of the whole, filled in reading order; with `total`, the squares past the values stay empty |
| `marimekko` | `labelKey`, `seriesKey`, `valueKey` | long: one per (column, segment) | columns as wide as their share of the total, each stacked to 100% by segment |
| `packed_bubbles` | `labelKey`, `valueKey`, `seriesKey` (optional) | one per category | circles packed together, each with an area proportional to its value |
| `dot_plot` | `labelKey`, `valueKey`, `seriesKey` (optional), `sizeKey` (optional) | one per detail row (several per category) | a circle per row along the category axis; `dodge` sets groups side by side, `jitter` spreads them sideways |
| `quadrant` | `xKey`, `yKey`, `labelKey` (optional), `sizeKey` (optional) | one per entity | a scatter divided into four by a line on each axis (at the averages unless `split` says `median` or `value`) |
| `control_chart` | `xKey`, `valueKey` | one per period or sample, in order | the values against their mean and control limits (mean ± `sigma` standard deviations, default 3), points outside the limits marked |
| `tile_map` | `regionKey`, `valueKey` | one per US state (or DC) or Indian state / union territory: a name, postal or ISO code, or an `US-` / `IN-` code | equal tiles at fixed spots in a map-like grid, coloured by value |

These are dashboard-only:

| `type` | Keys | Rows | Draw it as |
|---|---|---|---|
| `kpi_trend` | `xKey`, `valueKey` | one per period, in order | the latest value with its change and a sparkline; with `aggregate` `sum` or `avg`, the total or average of all periods and no change |
| `kpi_compare` | `valueKey`, `previousKey` (optional) | one | the value with its change against `previousKey`; without it, the widget data carries the comparison rows in `compareData` (see [endpoints.md](endpoints.md#post-emp1apidashboardwidgetdata)) |
| `bullet` | `labelKey`, `valueKey`, `targetKey` (or a fixed `target`) | one per bar | a bar against its target, over bands at the `ranges` percentages of the target (default 50, 80, 100) |
| `rose` | `labelKey`, `valueKey` | one per category | a Nightingale rose: a pie whose petals grow with the value (`roseType` `radius` or `area`) |
| `text` | none; `template` holds Markdown | none, or any | the Markdown, where `{{column}}` is the first row's value and `{{#rows}}…{{/rows}}` repeats once per row; render it without raw HTML |
| `histogram` | the widget data holds `bin_start`, `bin_end`, `count` (+ `seriesKey`) | one per bin | bars over the bins |
| `box_plot` | the widget data holds `low`, `q1`, `median`, `q3`, `high`, `outliers` per group | one per group | a box per group with its outliers |
| `violin` | the widget data holds the `box_plot` fields plus `min`, `max`, `mean`, `density` (`[value, density]` pairs) and `log` per group | one per group | each group's density mirrored around its place on the axis, with the quartiles inside; on a log value axis when `log` is true |
| `ttest_table` | the widget data holds `metric`, the group, `mean`, `lift_pct`, `p_value`, `significant`, `control` | one per group per metric | a table of each group against the control |
| `icicle` | `levels[]`, `valueKey` | long: one per leaf | nested bands, one per level |
| `chord` | `sourceKey`, `targetKey`, `valueKey` | one per link | ribbons between entities around a circle |
| `tree` | `idKey`, `parentKey`, `labelKey` (optional), `valueKey` (optional) | one per node | a parent–child tree; a node with an empty parent is a root |
| `parallel` | `metrics[]`, `labelKey` (optional), `colorKey` (optional) | one per entity | one line per row across parallel axes |
| `word_cloud` | `labelKey`, `valueKey` | one per word | words sized by weight |
| `calendar` | `xKey` (a date), `valueKey` | one per day | a calendar grid coloured by value |
| `horizon` | `xKey`, `seriesKey`, `valueKey` | long: one per (x, series) | a compact band row per series |
| `gantt` | `labelKey`, `startKey`, `endKey`, `seriesKey` (optional) | one per bar | bars from start to end on a time axis |

`treemap` also accepts `levels[]` instead of `labelKey` for a nested treemap.

`heatmap` also accepts `sizeKey`: each cell is then a square sized by that measure
as well as coloured by the value, as in Tableau's heat map.

`motion` is dashboard-only: `frameKey` (the period), `xKey`, `yKey`, `labelKey` (the
entity), optional `sizeKey` and `seriesKey`; long rows, one per (period, entity). Draw
the entities as bubbles on fixed axes and let the viewer play through the periods
(Tableau's motion chart); `autoplay` and `trails` are display options.

## Tableau chart names

Types answer to the names Tableau users know. A widget may be written with a
Tableau alias as its `type`; it is stored under the canonical type:

| Alias | Stored as |
|---|---|
| `highlight_table` | `pivot_table` with `heatmap: true` (unless the widget sets it) |
| `text_table`, `crosstab` | `pivot_table` |
| `dual_axis`, `dual_line`, `dual_combination` | `combo` (put a series on the right axis with `"axis": "right"`) |
| `pareto_chart`, `lollipop_chart` | `pareto`, `lollipop` |
| `side_by_side_circles` | `dot_plot` with `dodge: true` |
| `strip_plot`, `jitter_plot` | `dot_plot` with `jitter: true` |
| `violin_plot`, `violin_chart` | `violin` |

Tableau chart names, and the type that draws each:

| Tableau | `type` |
|---|---|
| BAN with comparison | `kpi_compare` |
| BAN with sparkline | `kpi_trend` |
| Bar chart | `bar` |
| Bar-in-bar chart | `bar_in_bar` |
| Barbell chart | `dumbbell` |
| Big number (BAN) | `kpi` |
| Box-and-whisker plot | `box_plot` |
| Bullet graph | `bullet` |
| Bump chart | `bump` |
| Butterfly chart | `butterfly` |
| Calendar heat map | `calendar` |
| Chord diagram | `chord` |
| Circle views | `dot_plot` |
| Combination chart | `combo` |
| Continuous area chart | `area` |
| Continuous lines | `line` |
| Control chart | `control_chart` |
| Crosstab | `pivot_table` |
| Density map | `density_map` |
| Discrete area chart | `area` |
| Discrete lines | `line` |
| Diverging bar chart | `diverging_bar` |
| DNA chart | `dumbbell` |
| Donut chart | `donut` |
| Dual combination | `combo` |
| Dual lines | `combo` |
| Dual-axis map | `world_map` |
| Dumbbell chart | `dumbbell` |
| Filled map | `world_map`, `country_map` |
| Flow map | `path_map` |
| Funnel chart | `funnel` |
| Gantt chart | `gantt` |
| Heat map | `heatmap` |
| Hex map (tile grid) | `tile_map` |
| Highlight table | `pivot_table` |
| Histogram | `histogram` |
| Horizontal bars | `hbar` |
| Jitter plot | `dot_plot` |
| Likert chart | `diverging_bar` |
| Line chart | `line` |
| Lollipop chart | `lollipop` |
| Map with pie charts | `chart_map` |
| Marimekko chart | `marimekko` |
| Mekko chart | `marimekko` |
| Motion chart | `motion` |
| Network graph | `graph` |
| Origin-destination map | `arc_map` |
| Packed bubbles | `packed_bubbles` |
| Pareto chart | `pareto` |
| Pie chart | `pie` |
| Point distribution map | `point_map` |
| Population pyramid | `butterfly` |
| Quadrant chart | `quadrant` |
| Radar chart | `radar` |
| Radial bar chart | `radial_bar` |
| Sankey diagram | `sankey` |
| Scatter plot | `scatter` |
| Side-by-side bars | `grouped_bar`, `grouped_hbar` |
| Side-by-side circles | `dot_plot` with `dodge: true` |
| Slope chart | `slope` |
| Slopegraph | `slope` |
| Sparkline table | `time_table` |
| Spider map | `arc_map` |
| Stacked bars | `stacked_bar`, `stacked_hbar` |
| Strip plot | `dot_plot` |
| Sunburst | `sunburst` |
| Symbol map | `world_map`, `point_map` |
| Text table | `pivot_table` |
| Tile map | `tile_map` |
| Tornado chart | `butterfly` |
| Treemap | `treemap` |
| Variable-width bars | `marimekko` |
| Violin plot | `violin` |
| Waffle chart | `waffle` |
| Waterfall chart | `waterfall` |
| Word cloud | `word_cloud` |

## Maps

Request map types by name in `chartTypes`. `world_map` and `country_map` also
appear in chat answers; the others are dashboard-only.

| `type` | Keys | Rows | Draw it as |
|---|---|---|---|
| `world_map` | `regionKey` (a country name or ISO-2/ISO-3 code), `valueKey`, `sizeKey` (optional) | one per country | countries coloured by value, with bubbles sized by `sizeKey` |
| `country_map` | `regionKey` (an ISO 3166-2 code such as `IN-KA`, or a region name with the `country` option), `valueKey` | one per state or province | one country's regions coloured by value |
| `chart_map` | `regionKey`, or `latKey` + `lonKey`; `seriesKey`, `valueKey` | long: one per (place, slice) | a small pie per place |
| `point_map` | `latKey`, `lonKey`; `sizeKey`, `colorKey`, `labelKey` (optional) | one per place | points on a basemap; `cluster: true` merges nearby points into counted bubbles |
| `density_map` | `latKey`, `lonKey`, `weightKey` (optional) | one per event | a heat map (`style: "heat"`) or square screen cells (`"screen_grid"`) |
| `grid_map`, `hex_map` | `latKey`, `lonKey`, `weightKey` (optional) | one per event | events summed into square or hexagonal cells (`cellSize` in metres); `hex_map` draws 3D columns |
| `contour_map` | `latKey`, `lonKey`, `weightKey` (optional) | one per event | density contour bands (`bands`, default 5) |
| `arc_map` | `fromLatKey`, `fromLonKey`, `toLatKey`, `toLonKey`, or `sourceKey` + `targetKey` holding countries; `valueKey` (optional) | one per (origin, destination) | arcs between places, as wide as `valueKey` |
| `path_map` | `geometryKey`: a GeoJSON LineString, WKT `LINESTRING` or encoded polyline | one per route | lines |
| `polygon_map` | `geometryKey`: a GeoJSON Polygon/MultiPolygon or WKT `POLYGON`; `valueKey` (optional) | one per area | areas coloured by value; `extruded: true` raises them |
| `geojson_map` | `geometryKey`: any GeoJSON geometry or Feature | one per shape | points, lines and areas |
| `layers_map` | none; `layers` lists the ids of other map widgets on the dashboard | none | those map widgets drawn together on one basemap, the first at the bottom |

Latitude and longitude are decimal degrees. Point maps (`point_map` through
`contour_map`) hold up to 50,000 rows, shape maps up to 5,000.

**Basemap.** Maps on a basemap may carry `basemap`: `auto` (light or dark with
the page), `light`, `dark`, `streets` or `none`. tableArth.ai's own views load
the tiled basemaps from OpenFreeMap (OpenStreetMap data) in the viewer's
browser; with `none` they draw country outlines only and request no tiles.

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
| `threshold` | `pareto` | the cumulative percentage the guide line marks; the bars up to it keep the full colour (default 80, `0` hides it) |
| `orientation`, `dotSize` | `lollipop`, `dumbbell` (`orientation` also `bar_in_bar`) | `horizontal` (default) or `vertical`; the dot's diameter in pixels |
| `sort` | `dumbbell` | `gap_desc` (default), `gap_asc`, `value_desc`, `label` or `none` |
| `neutral`, `percent` | `diverging_bar` | `split` (default) or `hide` the middle level; show each item's answers as shares of its total (default `true`) |
| `colorBy` | `slope` | `direction` (default: rises and falls) or `entity` |
| `topN`, `rankOrder` | `bump` | lines for the top N of the last period (default 10); `desc` (default) ranks the highest value first, `asc` the lowest |
| `maxAngle` | `radial_bar` | the sweep of the largest bar, in degrees (default 270) |
| `total` | `waffle` | the whole the values are parts of (default: their sum) |
| `dodge`, `jitter`, `dotSize` | `dot_plot` | groups side by side; spread overlapping circles; circle size in pixels |
| `split`, `xSplit`, `ySplit`, `quadrantNames` | `quadrant` | `average` (default), `median` or `value` (with the two split values); four names, comma separated: top right, top left, bottom left, bottom right |
| `facetKey` | line, area, bar types, scatter, bubble, pie, donut, lollipop | a column of the rows: draw the chart once per value of it (small multiples); `sharedScale` (default `true`) gives the panels one value scale |
| `referenceLine`, `referenceValue`, `referencePercentile`, `referenceLabel` | line, area, bar types, combo, scatter, bubble, lollipop | a line across the values at `average`, `median`, `min`, `max`, `percentile` (at `referencePercentile`, default 90) or `value` (at `referenceValue`) |
| `referenceBand`, `bandFrom`, `bandTo`, `confidence` | as `referenceLine` | a shaded band: `stdev` (within one standard deviation of the mean), `iqr` (the middle 50%), `ci` (the mean's confidence interval at `confidence`: `95` default, `90` or `99`) or `value` (`bandFrom` to `bandTo`, each a number, `min`, `max`, `average` or `median` of the drawn values, or a percentile such as `p10`; `min` to `max` is Tableau's range band) |
| `trendline`, `trendDegree` | scatter, bubble, line | a fitted trend: `linear`, `logarithmic`, `exponential`, `power` or `polynomial` (of `trendDegree`, 2–5) |
| `forecast`, `forecastInterval`, `forecastSeason` | `line`, `area` | Tableau's forecast: how many periods to continue each series past its last one, by exponential smoothing with the trend and season that fit best, inside a shaded `forecastInterval` (`95` default, `90`, `99` or `none`). The x values must be dated periods (`2025-11`, `2025-Q3`, `2025-11-30`); `forecastSeason` is `auto` (default), `none` or the periods per season. The widget data holds the history only: a client that draws the forecast works it out from those rows |
| `clusters` | `scatter`, `bubble` | Tableau's clusters: `auto`, or a number from 2 to 8, colours the points by k-means cluster over the plotted measures |
| `annotations` | `line`, `area`, `bar`, grouped and stacked bars, `combo` | notes on the x axis: `{ "at": "2025-11", "label": "Sale" }` marks an event, `{ "from": "2025-06", "to": "2025-08", "label": "Monsoon" }` shades a period; values are those of the x column |
| `scale`, `bandwidth`, `showBox`, `logScale` | `violin` | violin sizes (`area` default, `width` or `count`); smoothing, as a multiple of the automatic bandwidth (default 1); the quartile box inside (default `true`); a log value axis, `auto` (default: when the values run over orders of magnitude), `on` or `off` |
| `sigma` | `control_chart` | how many standard deviations the control limits sit from the mean (default 3) |
| `country`, `shape` | `tile_map` | `auto` (from the region values), `us` or `india`; `square` (default) or `hexagon` tiles |
| `heatmapScope` | `pivot_table` | with `heatmap`: colour across the whole `table` (default), within each `row` or within each `column` |
| `heatmap` | `pivot_table` | colour each cell by its value (a Tableau highlight table) |

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
- **A map shows no points** — the latitude and longitude columns hold text or
  are swapped: latitude runs from −90 to 90, longitude from −180 to 180.
- **A layered map is empty** — the map widgets its `layers` named were removed
  from the dashboard.
