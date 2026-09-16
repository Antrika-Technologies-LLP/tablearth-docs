# Apache Superset

TableArth connects to an existing Superset instance to browse governed datasets,
use saved metric definitions, work with existing charts, and display native
Superset dashboards inside the portal. Connections belong to the signed-in
TableArth user and their organization.

Saved TableArth dashboards can also be synced to Superset. Each sync refreshes
their data and updates the same destination dashboard.

## Chart coverage

Native dashboards are rendered by your Superset instance using the Superset
embedded SDK. This retains the chart plugins installed on that instance,
including its native tables, maps, pivot tables, time series and custom plugins.
TableArth does not translate these charts into a smaller chart library. A plugin
must already work in Superset, have its required data and external services,
and be compatible with Superset's embedded mode. Native support does not mean
TableArth's AI can create or convert every possible plugin configuration.

Existing charts can retain their native configuration when published in a
dashboard. Creating a new analysis from a structured metric query supports the
chart types offered by the TableArth publishing form. More complex charts can be
authored in Superset and then selected in TableArth.

Publication creates a new **draft dashboard** in Superset. Review and publish it
there when it is ready for your broader Superset audience.

## Sync a TableArth dashboard

1. Open a saved dashboard in a workbook or the dashboard viewer. Save any layout
   and filter changes first.
2. Choose **Sync to Superset**, select your connection, and review the chart checks
   and presentation warnings.
3. Choose **Sync dashboard**. Use **View in tableArth** or **Open in Superset** to
   inspect the result.
4. Choose **Refresh status** and sync again whenever you want fresh data. Repeated
   syncs update the same Superset dashboard. Retrying an interrupted request
   recovers its result without creating another dashboard.

All 23 current TableArth widget types have Superset mappings, including KPI,
tables, grouped/stacked bars, line/area, pie/donut, mixed charts, scatter, gauges,
progress, funnel, treemap and heatmap. Superset may use different formatting,
colors or category ordering. Ranked lists retain source order as tables with
cell bars; statistic grids become individual cards. Unsupported layouts or
configurations stop the sync with an explanation.

Local widget results become durable snapshots using the dashboard's **saved
filters**. They refresh only when you sync; they are not live connections to the
original workbook. The initial limits are 24 widgets, 60 expanded native charts,
10,000 rows per local widget, 200 columns and 20 MiB of total results. Empty or
truncated results are rejected. Sources requiring a viewer-specific tenant are
not eligible for a shared snapshot.

If the managed dashboard, charts or datasets are edited in Superset, the next
sync stops so those changes can be reviewed. Restore the last synced definitions
before retrying. Two-way editing and automatic conflict merging are not enabled.

## Add any Superset chart to a TableArth dashboard

Choose **Add Superset chart** on a saved dashboard, select a connection and choose
a chart. The tile uses native Superset rendering, including chart types and
custom plugins installed on that instance. **Create another chart in Superset**
opens the native authoring tools for advanced chart configuration.

These tiles retain their original datasets and use their own filters. The local
TableArth filter bar does not filter native tiles. Secure embedding must be
enabled to display them inside the dashboard; otherwise the tile offers an
**Open in Superset** link. When syncing a dashboard containing native tiles, they
must use the selected connection. Changing that connection's destination or
account requires re-adding the tiles; rotating its password does not.

## Prerequisites

- A reachable, supported Superset deployment. The repository includes a pinned
  **Superset 6.1.0** development fixture and verification script.
- HTTPS reachable from both the TableArth backend and the user's browser.
- A Superset account with access to the intended datasets, charts and dashboards.
  Use provider `db`, or `ldap` when that provider is configured by your instance.
  Interactive SSO-only logins need an approved API-compatible account.
- A durable TableArth backend encryption key for saved connection credentials.
- Superset administration access to enable native embedding and review its
  guest role if dashboard embedding will be used.

## Backend configuration

Set these variables on the **TableArth backend**, never in a public portal build:

| Variable | Purpose |
| --- | --- |
| `TABLEARTH_SUPERSET_SECRET_KEY` | Standard Base64 of 32 random bytes; encrypts saved Superset credentials. Keep stable across replicas and deployments. |
| `TABLEARTH_SUPERSET_ALLOWED_HOSTS` | Optional comma-separated exact hostnames, without scheme, port or path. When set, connections are restricted to these hosts. A private-network instance must be explicitly listed. |
| `TABLEARTH_SUPERSET_EMBED_HOSTS` | Exact hostnames approved by the operator for embedding after guest-role and dataset-isolation review. Required before a connection may enable embedding. |
| `TABLEARTH_SUPERSET_ALLOW_HTTP` | Defaults to disabled. `true` permits HTTP only for an explicitly allowlisted loopback host in local development. |
| `TABLEARTH_SUPERSET_SYNC_TARGETS` | Private JSON configuration binding each organization and Superset account to an approved PostgreSQL publication database, schema and writer credentials. Required for syncing local chart data. |

Without a host allowlist, public HTTPS destinations still undergo address
validation. Cloud metadata and other prohibited network addresses remain blocked.
An allowlist is not a routing service: a private instance must also be reachable
from the backend and browser, normally through your approved network or proxy.

Generate the encryption key using a cryptographically secure secret manager or
`openssl rand -base64 32`. Store it as a protected secret. Back it up with your
credential-recovery procedure; losing it prevents decrypting existing saved
connections. Key rotation needs a planned re-encryption or reconnection process.

For dashboard sync, provision separate publication storage per organization.
Give TableArth a writer account and Superset a read-only account; deny access to
other organizations' databases. Register that publication database in Superset
and configure its exact database ID, organization ID, Superset URL/account,
PostgreSQL JDBC URL, writer credentials and schema in `TABLEARTH_SUPERSET_SYNC_TARGETS`.
The backend implementation includes the full SQL setup and JSON example in
`antrika-backend/docs/SUPERSET_DASHBOARD_SYNC.md`. Existing snapshots remain
immutable; operators must plan storage retention and remove only unreferenced
artifacts. Disconnecting a connection does not delete published data.

## Connect

Open the portal's **Superset** area and add a connection with a descriptive name,
Superset base URL, username, password and authentication provider. Credentials
are handled by the backend. Choose only the datasets and dashboards your account
is authorized to use.

Use saved metrics when asking questions to keep definitions aligned with
Superset. Queries use the dataset/chart API with bounded result sizes; SQL Lab
is not a fallback for bypassing dataset policy. Errors should be resolved in the
connection, dataset permissions or metric definition before retrying.

## Enable native dashboards

Your Superset administrator must:

1. Enable `FEATURE_FLAGS["EMBEDDED_SUPERSET"]`.
2. Set strong, separate `SECRET_KEY` and `GUEST_TOKEN_JWT_SECRET` secrets, and an
   explicit `GUEST_TOKEN_JWT_AUDIENCE` shared by Superset processes.
3. Create a dedicated role, for example `TableArthEmbeddedGuest`, and configure
   `GUEST_ROLE_NAME` to that role. Grant the necessary read capabilities for
   dashboard, chart, dataset, current-user and supporting viewer endpoints.
   Do not copy administrator privileges or broadly grant source database access.
4. Grant `can_grant_guest_token` on `SecurityRestApi` to the account used for
   this embedding connection only after installing the issuer-authorization
   hook below. The upstream permission alone is broader than the account's
   normal dashboard access.
5. Enable embedding on each intended dashboard and set its allowed domains to
   exact portal origins, including scheme and port. Keep this list nonempty.
6. Add the portal origins to the Superset response CSP `frame-ancestors` policy
   while retaining the rest of its CSP. Allow the Superset origin in the
   TableArth portal's `frame-src` policy. Preserve an origin-bearing `Referer`
   header for Superset's embedding checks.
7. Review each embedded dataset's row policy, then add the instance hostname to
   `TABLEARTH_SUPERSET_EMBED_HOSTS` and opt in the TableArth connection.

Guest tokens represent Superset's configured **guest role**. They do not inherit
the connected Superset account's roles or row-level policies merely because the
token contains that person's username. TableArth limits token resources to the
selected dashboard, but dataset rows must also be restricted by the reviewed
guest policy or by datasets that are already constrained to the intended tenant.
Do not enable this flow for a shared dataset whose tenant isolation depends only
on ordinary logged-in user roles. Arbitrary SQL policies supplied by a browser
are not a trusted tenant boundary.

Superset 6.1.0's default guest-token resource validation checks existence, not
whether the token issuer can access the dashboard. Normal dashboard visibility
can also be granted when only one of its datasets is accessible. Require both
dashboard access and access to **every dataset** before allowing delegation.
The local fixture uses this released configuration hook; place the equivalent
in your production `superset_config.py` after reviewing its dataset policies:

```python
def validate_tablearth_guest_resources(body):
    from superset import security_manager
    from superset.daos.dashboard import EmbeddedDashboardDAO
    from superset.models.dashboard import Dashboard

    resources = body.get("resources", [])
    if not resources:
        return False
    for resource in resources:
        if resource.get("type") != "dashboard":
            return False
        identity = str(resource.get("id", ""))
        if identity.isdecimal():
            dashboard = Dashboard.get(identity)
        else:
            embedded = EmbeddedDashboardDAO.find_by_id(identity)
            dashboard = embedded.dashboard if embedded else None
        if dashboard is None or not security_manager.can_access_dashboard(dashboard):
            return False
        if not all(security_manager.can_access_datasource(dataset)
                   for dataset in dashboard.datasources):
            return False
    return True

GUEST_TOKEN_VALIDATOR_HOOK = validate_tablearth_guest_resources
```

This verifies the issuing account independently of browser-supplied token user
attributes. It does not copy that account's RLS into the guest role. Keep the
guest-role and dataset constraints described above, including a review of
datasets referenced by custom or composite plugins. Validate the hook against
your exact version before upgrading. See the released
[security manager](https://github.com/apache/superset/blob/6.1.0/superset/security/manager.py)
and [guest-token API](https://github.com/apache/superset/blob/6.1.0/superset/security/api.py).

The browser receives short-lived guest tokens through the authenticated
TableArth backend and refreshes them while the dashboard is mounted. It must
never receive the connection password or a privileged Superset access token.

## Deployment and acceptance

Before production enablement, validate the full customer path against the exact
Superset version and installed plugins:

- Two TableArth users in different organizations and two Superset identities
  cannot access each other's connection, datasets, charts, saved links or jobs.
- Saved metrics return expected results; filters, time ranges and row limits are
  preserved; unauthorized and unavailable sources fail closed.
- Native dashboards render every plugin used by that customer. Check native
  filters, cross-filter interactions, layout, custom plugins and external map
  dependencies in the embedded browser context.
- Guest-token refresh, expiry, connection deletion, revoked dataset access and
  denied embedding origins behave as expected. Check ordinary viewer and guest
  policies separately, using a browser session without a Superset admin login.
- Direct Superset guest-token requests for foreign dashboards are denied. A
  mixed-dataset dashboard that the issuer can partially view cannot be delegated
  until the issuer has access to every dataset and the guest policies are safe.
- Publishing preserves charts and creates correct dashboard relationships.
  Repeated requests and interrupted jobs do not create duplicate publications.
- Production uses durable PostgreSQL metadata, tested backups, stable secrets,
  scoped source credentials, TLS, access/error monitoring and restricted egress.
  Configure Superset workers, result storage, cache and scheduler for any enabled
  async queries, native reports or exports; the small local fixture does not
  exercise those production services.

Superset needs network access to durable source data. Dashboard sync publishes
file, federated and database widget results to the configured publication storage.
Native embedded charts do not automatically become standalone JavaScript-widget
or Excel charts, offline exports or publicly shared dashboards.

For a local start, use `antrika-backend/dev/superset/setup.sh`. It generates unique
local secrets, two synthetic tenants, saved metrics and five chart types, and
includes real HTTP isolation checks. No production deployment is performed.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Connection rejected before login | URL format, backend hostname allowlists, TLS and network reachability. |
| Data is missing or denied | The connected identity's dataset permissions and saved metric definitions. |
| Embedding is unavailable | Operator embedding allowlist, connection opt-in, feature flag and dashboard embedding configuration. |
| Guest token denied | `can_grant_guest_token`, dedicated guest role permissions, resource UUID and audience configuration. |
| Frame is blocked | Superset allowed origins, both applications' CSP, HTTPS and browser referrer policy. |
| Chart appears but its query fails | Guest-role RLS and dataset access; this differs from the normal logged-in account. |
| One chart plugin fails | Confirm the same plugin and dataset work in Superset and inspect its embedded-mode service/CSP requirements. |
| Sync requires publication setup | Configure the exact organization/account binding and publication database in `TABLEARTH_SUPERSET_SYNC_TARGETS`. |
| Sync reports a conflict | Refresh status; review and restore external changes to the managed Superset resources before retrying. |
| Native tile reports a changed connection | Re-add the chart using the intended connection and account. |

References: [Superset 6.1.0 release](https://github.com/apache/superset/releases/tag/6.1.0),
[released embedding SDK](https://github.com/apache/superset/blob/6.1.0/superset-embedded-sdk/README.md),
[released configuration](https://github.com/apache/superset/blob/6.1.0/superset/config.py).
