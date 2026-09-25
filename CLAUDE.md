# tableArth.ai docs

Public integration handbook: plain Markdown with no build, lint or CI. This repository is PUBLIC, and so is every branch, commit message and PR.

- Document only shipped, public behaviour (CONTRIBUTING rule 1). Never mention internal repositories, source paths, class names, internal hosts, environment details or unreleased work in pages, commit messages or PR text.
- Brand in prose: **tableArth.ai** (lower-case t, capital A, `.ai`), never TableAI, TableArth or Tablearth. Wire identifiers keep their old names; copy them verbatim: `/emp/1/api/tableai/...`, `window.antrika(...)`, `antrikaSSO`, `.antrika-table-ai-widget`, `widget_id`, `data-id`.
- Placeholders: tenant `https://yourcompany.tableArth.ai`, widget bundle host `https://<your-widget-host>`, host app `https://app.example.com`, ids `WIDGET_ID` / `YOUR_TENANT_ID`. The Excel add-in is not a shipped surface; don't document it.
- Where things go: a new endpoint goes in docs/api/endpoints.md AND the relevant integration page. A new surface goes in docs/integrations/, plus the README path picker and the CONTRIBUTING file-layout tree. Auth modes extend docs/api/auth.md (don't fork it). Examples go in docs/examples/.
- Every user-visible change gets a CHANGELOG entry under `## Unreleased`, as a dated `### YYYY-MM-DD — Title` block with bullets that start with a bold phrase. Spelling and tone fixes don't need one.
- Style: second person, present tense, sentence-case headings, fenced code with a language tag (`jsonc` for commented JSON), tables for three or more parallel options, the top failure modes on every page, no marketing adjectives. Prose is hard-wrapped at ~80 columns.
- Links are relative to the current file. GitHub anchor slugs are lower case, with punctuation dropped and spaces turned into hyphens (e.g. "`remoteSessionId` / `remoteId`" → `#remotesessionid--remoteid`). Before finishing, check with a short script that every relative link and anchor resolves.
- Git: work on `development`; merging it into `main` (the GitHub default) publishes. Subjects are imperative and sentence case, with a body explaining why. No AI attribution: no `Co-Authored-By` trailer, no "Generated with Claude Code" line, no session links (`.claude/settings.json` also turns these off).
