# .github

Workflows compartidos de la organización.

## CI reutilizable (`ci-reusable.yml`)

El CI de cada repo de producto vive **una sola vez** aquí. Cada repo tiene solo un `ci.yml` corto que lo llama y le pasa lo que es propio del repo: label del runner, stack y comandos. El workflow no nombra ningún repo.

Todos los jobs corren en los runners self-hosted de build, elegidos con `[self-hosted, build, <repo-label>]`.

### Qué job corre en cada evento

| Job | `pull_request` | `push` | `schedule` / `workflow_dispatch` |
|---|---|---|---|
| `lint-build-test` | sí | | |
| `gitleaks` | sí | | |
| `dependency-audit` | base `dev`, `stg` o `main` | `dev`, `stg`, `main` | sí |
| `codeql` | base `dev`, `stg` o `main` | `dev`, `stg`, `main` | sí |
| `pr-title` | sí | | |
| `validate-pr-base` | sí | | |
| `commit-lint` | | cualquier rama | |

Los demás jobs de cada evento quedan en *skipped*. `dependency-audit` (salvo `audit-blocking: true`) y `codeql` son informativos: no ponen el check en rojo.

### Pendiente: `docker-build`

El check de build de imagen (sin push) queda fuera de este workflow hasta que esté el diseño con kaniko. Se agregará después como un job aparte, separado de los de arriba. Mientras tanto ningún repo que use este workflow tiene ese check en sus PR.

### Caller mínimo (`.github/workflows/ci.yml` en cada repo)

```yaml
name: CI
on:
  pull_request:
    types: [opened, edited, synchronize, reopened]
  push:
    branches: ["**"]
  schedule:
    - cron: "0 6 * * 1"
  workflow_dispatch:
permissions: {}
concurrency:
  group: ci-${{ github.event_name == 'pull_request' && (github.event.action != 'edited' || github.event.changes.title || github.event.changes.base) && github.ref || github.run_id }}
  cancel-in-progress: true
jobs:
  ci:
    name: ${{ github.event_name }}
    if: github.event.action != 'edited' || github.event.changes.title || github.event.changes.base
    uses: lumina-w/.github/.github/workflows/ci-reusable.yml@main
    permissions:
      contents: read
      pull-requests: read
      security-events: write
      actions: read
    with:
      repo-label: <label-del-runner>
      toolchain: npm
      node-version-file: .nvmrc
      install-command: npm ci
      format-command: npm run format:check
      lint-command: npm run lint
      test-command: npm run test
      build-command: npm run build
      audit-command: npm audit --audit-level=high
      codeql-languages: '["javascript-typescript"]'
    secrets:
      GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}
```

Por qué el caller tiene esa forma:

- `name: ${{ github.event_name }}`: los checks quedan como `<evento> / <job>` (por ejemplo `pull_request / Lint, build & test`). Así los jobs *skipped* de un run de `push` no pisan los checks del PR sobre el mismo commit. Un check *skipped* cuenta como verde en branch protection.
- `if` y `concurrency`: una edición del PR que no cambia el título ni la base no corre nada ni cancela el run en curso. Cambiar el título o la base sí vuelve a correr todo.
- `permissions`: es el máximo que pueden pedir los jobs del workflow (`security-events: write` y `actions: read` son de CodeQL, `pull-requests: read` de gitleaks).
- Los checks requeridos en branch protection se configuran con el nombre nuevo (`pull_request / ...`).

### Inputs

| Input | Obligatorio | Default | Uso |
|---|---|---|---|
| `repo-label` | sí | | Label del runner del repo: `[self-hosted, build, <repo-label>]` |
| `toolchain` | sí | | `npm`, `pnpm`, `python` o `none` |
| `node-version` / `node-version-file` | | | Versión de Node, o archivo como `.nvmrc` |
| `python-version` | | | Versión de Python |
| `python-cache-path` | | | Archivo de requirements para la caché de pip. Vacío: sin caché |
| `env-vars` | | | `CLAVE=valor` por línea para install, checks y audit. Nunca secrets |
| `install-command` | | | `npm ci`, `pnpm install --frozen-lockfile`, `pip install -r ...` |
| `format-command` | | | Check de formato |
| `lint-command` | | | Linter |
| `typecheck-command` | | | Chequeo de tipos |
| `test-command` | | | Tests |
| `build-command` | | | Build |
| `audit-command` | | | Auditoría de dependencias (`npm audit --audit-level=high`, `pnpm audit --audit-level=high`, `pip install pip-audit && pip-audit -r ...`). Vacío: no corre el job |
| `audit-blocking` | | `false` | `true` hace que la auditoría ponga el check en rojo |
| `codeql-languages` | | | Arreglo JSON de lenguajes de CodeQL (`'["javascript-typescript"]'`, `'["python"]'`). Vacío: no corre el job |
| `commitlint-config` | | `.commitlintrc.json` | Config de `pr-title` y `commit-lint`. Vacío: no corren |
| `gitleaks` | | `true` | Escaneo de secretos en cada PR |

Cada comando vacío salta su paso. Un comando puede tener varias líneas: corren en orden y el paso falla con la primera que falle.

### Secrets

| Secret | Obligatorio | Uso |
|---|---|---|
| `GITLEAKS_LICENSE` | para repos de la organización | Licencia de gitleaks-action. Debe existir también en los secrets de Dependabot, o los PR de Dependabot fallan en `gitleaks` |

### Notas

- `commit-lint` y `pr-title` usan el CLI de commitlint, no `wagoid/commitlint-github-action`: esa action corre en un contenedor Docker y ataría el job a un daemon de Docker en el runner. El rango de commits es el mismo: `before..after` del push, o los commits del payload en una rama nueva.
- `validate-pr-base` aplica el flujo `dev -> stg -> main` de la organización.
