# Backstage Governance UI Reference

This reference demonstrates a custom Governance page inside an existing Backstage application. All sample API/event data is synthetic, and all governance rules are proposals, not approved bank standards.

## Start Here

For a quick preview, open `backstage-files/packages/app/public/governance-report/index.html` directly in a browser. No installation, Node.js, WSL, or Backstage is needed to view that generated file.

To integrate it into your own Backstage application, follow the instructions below. This archive is not a complete Backstage app and must not replace your project's package manifest or App configuration.

## Included Files

```text
backstage-files/
  packages/app/src/modules/governance/index.tsx
  packages/app/public/governance-report/index.html
  packages/app/public/governance-report/results.json
integration/
  App.registration.tsx.example
  dependencies.reference.json
governance-poc/
  package.json / package-lock.json
  src/ / test/ / samples/
  README.md
third-party/
  lucide-LICENSE
MANIFEST.json
```

- `index.tsx` is the actual custom Backstage page module.
- The generated HTML contains its own styles, icons, JavaScript, and scenario data; it makes no runtime requests to load its report data.
- `results.json` is a machine-readable companion artifact. Editing it alone does not update the HTML's embedded data.
- `src/report.mjs` generates the HTML shell; `src/report-client.mjs` implements UI interactions; `src/report.css` controls styling.
- The remaining runner source and tests provide context and allow reproducible generation. You do not have to install them merely to embed the existing HTML.
- `MANIFEST.json` lists packaged file sizes and SHA-256 hashes, excluding itself.

## Integrate Into Backstage

### 1. Check Your Frontend System

The included module targets Backstage's **new frontend system**, using `createApp` from `@backstage/frontend-defaults` and `PageBlueprint` from `@backstage/frontend-plugin-api`.

It was tested with an app declaring `@backstage/frontend-plugin-api ^0.18.2`, `@backstage/frontend-defaults ^0.5.7`, React 18, and `@material-ui/icons ^4.9.1`. These are reference versions, not an instruction to downgrade or replace your own dependency tree. Verify compatibility with your Backstage release and use its package manager.

An older application based on `FlatRoutes` and legacy `createApp` APIs needs a legacy route/sidebar adapter. Do not paste the new frontend feature registration into a legacy App unchanged. The standalone HTML remains reusable in either system.

### 2. Copy The Page And Assets

Merge the contents of `backstage-files/` into your Backstage repository, retaining these paths:

```text
packages/app/src/modules/governance/index.tsx
packages/app/public/governance-report/index.html
packages/app/public/governance-report/results.json
```

The page's iframe uses `/governance-report/index.html`, served by the frontend. This assumes the application is hosted at the origin root. For a subpath deployment, adjust the asset URL and routing to match your frontend base path.

### 3. Register The Feature

Add this import to your existing `packages/app/src/App.tsx`:

```tsx
import { governancePlugin } from './modules/governance';
```

Append `governancePlugin` to the existing `createApp({ features: [...] })` list. Keep all existing features; do not replace the App with our sample. A minimal illustration is in `integration/App.registration.tsx.example`.

The new frontend can derive navigation from page extensions. In the source app, its sidebar's `nav.rest(...)` displays this page automatically. If your sidebar filters or manually enumerates entries, configure it to include the new page. Avoid adding a second entry if one is already generated.

### 4. Check Dependencies And Run

The module imports `@backstage/frontend-plugin-api` and `@material-ui/icons/VerifiedUser`. These must be available to your app workspace. If you use another icon library, replace that icon with an appropriate JSX icon element compatible with your Backstage release. The blueprint expects `icon: <VerifiedUserIcon />`, not the component constructor.

Run your app's normal typecheck and start commands, then open `/governance`. The source demo used `http://localhost:3001/governance`, but your app can use its own port.

Inspect the browser console and network panel. The iframe URL should return HTML, not a 404 or the Backstage app shell. Your deployment's CSP must permit same-origin framing. Restrict access to report assets as appropriate; placing files in `public/` is not a security boundary or an authorization check.

No additional plugin backend, cloud credentials, WSL-specific launcher, or community Spectral plugin is needed for this custom report page. WSL was only the source machine's Backstage development environment.

## Regenerate The Report (Optional)

Use a supported Node.js runtime satisfying the pinned dependencies; Node.js 24 was used in the original demo. From `governance-poc/`:

```powershell
npm ci
npm test
npm run demo
```

Only continue if those commands succeed. Copy the new `reports/index.html` and `reports/results.json` into your Backstage app's `packages/app/public/governance-report/`, replacing the previous generated artifacts. Then reload/rebuild/restart according to your app's development or deployment workflow.

The generator uses the existing `samples/target` and `samples/scenarios` snapshots and preserves them. All ten demonstration outcomes should match expectations; some scenarios intentionally FAIL governance. A successful `npm run demo` means the expected scenario outcomes matched, not that every scenario passed validation.

The report footer shows pinned tool versions, and the renderer includes a fixed demonstration baseline label of ten legacy findings. Adapt that label or make it dynamic before using a different baseline or live data.

The bundled `publish-backstage.mjs` and `backstage-publication.mjs` are optional original catalog-export references. `publish-backstage.mjs` assumes a sibling `backstage-portal/` and writes there; it is **not** needed to regenerate or integrate this page and should not be run unchanged against a different repository layout.

## What The Page Demonstrates

- Scenario selection, PASS/FAIL results, blocking issues, existing debt, warnings, and resolved findings.
- Search and state filters for findings.
- Target-versus-proposed specification source views.
- A local catalog-metadata inventory and a proposed rule catalogue.
- JSON export of the selected scenario.

The report uses real previously computed Spectral and structural-validator findings. Selecting a scenario in the browser **does not execute validation or a CI pipeline**. Refreshing this page does not regenerate results. The custom inventory is a report table, not a live query of Backstage's catalog.

## Governance And Security Boundaries

- This custom page is separate from the community API Spectral Linter's per-entity Linter tab; that package and adapter are deliberately not included in the UI integration.
- Our runner applies baseline comparison in Node.js. The browser only displays the saved outcome.
- CI enforcement requires a validation command with a failing exit code and the appropriate required Azure Repos branch policy. Embedding this report does not block PRs.
- API definitions, before/after source text, owners, and findings are embedded in the HTML. Review and sanitize your own data before publishing artifacts. Do not put credentials or sensitive production specifications in publicly served reports.
- Generated report HTML contains executable JavaScript. Publish only trusted build artifacts; do not serve arbitrary uploaded HTML through this page.
- No production authentication, live validation endpoint, duplicate detection, breaking-change analysis, or expiring exception workflow is supplied by this reference.

## Suggested Demo

1. Select **Compliant new API**: PASS with ten existing findings.
2. Select **Missing owner**: a new ownership issue blocks the change.
3. Use **Changes** to inspect source differences and **Findings** to inspect the rule.
4. Select **Legacy issue resolved**: existing debt reduces to nine.
5. Select **Shared schema regression**: dependencies are revalidated even though only the shared schema changed.

## Attribution

Icons in the report are from `lucide-static`; its upstream license is included in `third-party/lucide-LICENSE`. npm dependency licenses remain governed by their respective upstream packages. No dependency installations or third-party source trees are bundled.