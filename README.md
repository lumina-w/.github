# .github

Special repository of the `lumina-w` GitHub organization.

## Current state

**Empty, with no effect on the organization.** Besides this README it holds no
files: no organization profile, no default issue or pull request templates, no
community health files and no workflow templates. Nothing here changes how any
other repository of the organization behaves.

## What GitHub uses this repository for

By GitHub convention, a public repository named `.github` in an organization
provides organization-wide defaults. None of these exist yet:

| Path | What GitHub does with it |
|---|---|
| `profile/README.md` | Shown on the organization's public profile page (github.com/lumina-w). |
| `.github/ISSUE_TEMPLATE/`, with an optional `config.yml` | Default issue forms and templates for every repository of the organization that has none of its own. |
| `.github/pull_request_template.md` | Default pull request body for repositories without their own template. |
| `CONTRIBUTING.md` | Default contributing guidelines, linked when someone opens an issue or pull request. |
| `CODE_OF_CONDUCT.md` | Default code of conduct. |
| `SECURITY.md` | Default security policy: how to report a vulnerability. |
| `SUPPORT.md` | Default pointers to where to get help. |
| `GOVERNANCE.md` | Default description of how the projects are governed. |
| `FUNDING.yml` (in `.github/`) | Default sponsor button. |
| `workflow-templates/` | Starter workflows offered in the Actions tab of the organization's repositories. |

How the defaults behave:

- The community health files (`CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`,
  `SECURITY.md`, `SUPPORT.md`, `GOVERNANCE.md`) can live at the root of this
  repository, in `docs/` or in `.github/`.
- A repository's own file always wins. A default only applies to a repository
  that does not have that file in its root, `docs/` or `.github/` folder.
- Defaults are not copied into the other repositories and do not show up in
  their file trees or clones.
- For the defaults to apply, this repository must stay public. Anything added
  here is visible to everyone.

## What does not belong here

- Reusable CI/CD workflows called with `uses:`. They are kept in a separate
  repository of the organization.
- Anything private: internal product information, secrets, internal hostnames
  or member-only documentation.

## Next steps

Nothing is planned yet. When the organization decides to add a profile,
templates or community health files, they go in the paths listed above and
this README is updated to list what exists.
