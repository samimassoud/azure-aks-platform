# Application source

The workload, the [OpenTelemetry Astronomy Shop](https://github.com/open-telemetry/opentelemetry-demo), will be imported into `app/astronomy-shop/`. It hasn't been imported yet.

## Planned import

- **Method:** `git subtree add --prefix=app/astronomy-shop <remote> <ref> --squash`.
  - This records the exact upstream commit the import came from (a `git-subtree-split:` trailer).
  - Later upstream releases can be pulled in with `git subtree pull`.
  - `--squash` keeps the source and its provenance. It does **not** copy upstream's full commit history into this repository. That history remains in the upstream project.
- **Licence:** the application keeps its own Apache-2.0 `LICENSE`, copyright and attribution notices, unchanged.
- **Before importing:**
  - Inspect upstream's tracked environment files (`.env`, `.env.override`, `src/react-native-app/.env`).
  - Sanitise any credentials and keep example templates.
  - Run a Gitleaks scan of the imported tree.

`app/astronomy-shop/` is intentionally absent until then, because `git subtree add` requires the target directory not to exist.

The analysis of the application is in [`docs/astronomy-shop/`](../docs/astronomy-shop/).
