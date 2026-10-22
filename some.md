## Platform UI 3.0 for Harness

A **pnpm monorepo** that builds Harness UI 3.0 microfrontends. Code lives in `apps/` (deployable applications) and `packages/` (shared libraries).

```
platform-ui-monorepo/
├── apps/
│   ├── kitchen-sink/     # Primary MFE app (deployed to harness-core-ui)
│   └── resilience/       # Chaos / DR / load MFE (deployed to harness-core-ui)
├── packages/             # Shared internal libraries
├── pnpm-workspace.yaml   # Workspace + catalog version pins
├── nx.json               # Task orchestration & caching
└── tsconfig.base.json    # Shared TypeScript config
```

> **Fdsfsdfsdf fdfsdfsdfsdf
---

### Setup

**Requirements**: Node 22+, pnpm 9.5.0 (enforced via `packageManager` field)

We use [Harness Artifact Registry](https://harness0.harness.io/ng/account/l7B_kbSEQD2wjrM7PShm5w/module/har/registries/harness-github-npm-registry/packages) for private NPM packages.

```bash
pnpm i
```

### Run the app

```bash
pnpm dev      # Standalone dev server (Vite, port 5173)
pnpm dev:mfe  # MFE dev server (Webpack, port 4000)
```

Other apps in `apps/` expose their own scripts — run them from dsfsdfsthe app directory
(e.g. `cd apps/resilience && pnpm dev:mfe`). See ea's README for details.

**IaCM UI 3.0** (`apps/iacm`) is an MFE loaded by harness-core-ui. To run it against QA (Node versions, Unified UI, `CDS_IACM_UI3_MFE`), see [apps/iacm/docs/local-dev-qa.md](./apps/iacm/docs/local-dev-qa.md).

### Common commands

| Command          | Description                                     |
| ---------------- | ----------------------------------------------- |
| `pnpm dev`       | Kitchen-sink standalone dev server (Vite :5173) |
| `pnpm dev:mfe`   | Kitchen-sink MFE dev server (Webpack :4000)     |
| `pnpm build:mfe` | Production MFE build (kitchen-sink)             |
| `pnpm test`      | Run all tests                                   |
| `pnpm lint`      | Lint all apps/packages                          |
| `pnpm typecheck` | Type-check dsf all apps/packages                    |
| `pnpm pretty`    | Check Oxfmt formatting                          |
| `pnpm typecheck` | Type-check dsf all apps/packages                    |
| `pnpm pretty`    | Check Oxfmt formatting                          |

---

## Build and Release

Builds and deployments to QA run automatically on every commit through [platformUIRelease](https://harness0.harness.io/ng/account/l7B_kbSEQD2wjrM7PShm5w/all/orgs/PROD/projects/Harness_Commons/pipelines/platform_ui_release/pipeline-studio/?storeType=INLINE) via a cron based trigger [dailyRelease](https://harness0.harness.io/ng/account/l7B_kbSEQD2wjrM7PShm5w/all/orgs/PROD/projects/Harness_Commons/pipelines/platform_ui_release/triggers/dailyRelease/detail?storeType=INLINE).

## Known Issues & Gotchas

### react-router-dom v5 vs v6 Compatibilitydsfsd f

platformUI uses **react-router-dom v5**, but `@harnessio/ui` components expect **v6-style NavLink API** (where `className` can be a function receiving `{ isActive }`).

**Solution**: The `NavLinkV5Wrapper` in `src/components/navigation/providers/router-provider.tsx` adapts v6-style props to v5. When providing router context to `@harnessio/ui` components, use this wrapper:

```tsx
import { NavLinkV5Wrapper } from '@components/navigation/providers/router-provider'

<RouterContextProvider
  Link={Link}
  NavLink={NavLinkV5Wrapper as any}
  navigate={history.push as any}
  location={location as any}
>
```

**Key differences handled**:

- `className` as function: `({ isActive }) => string`
- `end` prop (v6) → `exact` prop (v5)
- `caseSensitive` prop (v6) → `sensitive` prop (v5)

### CSS Cascade Conflicts in MFE Mode

When running as a microfrontend inside `harness-core-ui`, CSS from the host application can override platformUI's Tailwind classes due to cascade order (host CSS loads after remote CSS).

**Example**: The host's `.hidden { display: none }` can override platformUI's `.md:block` because both have the same specificity and the host loads later.

**Solutions**:

1. **Preferred**: Create custom utility clasfdsfdsfsdfdsses in canary's `styles.css` with explicit media queries:

   ```css
   @layer utilities {
     .cn-sidebar-root-responsive {
       display: none;
     }
     @media (min-width: 768px) {
       .cn-sidebar-root-responsive {
         display: block;
       }
     }
   }
   ```

2. **Fallback**: Use `!important` overrides in `App.css` (less preferred, use sparingly)

**Debugging tip**: If elements are unexpectedly hidden in MFE mode but work in standalone, inspect the computed styles to check for cascade conflicts.

### Working with Local Canary

To test changes in both `canary` and `platformUI` simultaneously:

1. **Build canary packages**:

   ```bash
   cd /path/to/canary
   pnpm -r -F './packages/*' build
   ```

2. **Link or publish locally** (choose one):

   - Use `pnpm link` to symlink local packages
   - Or publish to a local registry / use `workspace:*` protocol

3. **Run platformUI** pointing to local canary:

   ```bash
   cd /path/to/platformUI
   pnpm dev      # standalone mode
   pnpm dev:mfe  # MFE mode (to test with harness-core-ui)
   ```


