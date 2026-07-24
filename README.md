# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset. Other repos inherit it
with:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>andykenward/renovate-config"]
}
```

## What it does

On top of `config:best-practices` (which already pins Docker digests via
`docker:pinDigests`), it adds custom managers for version pins that live inside
Dockerfile `RUN`/`ARG` lines — places Renovate's built-in managers can't see:

- **pnpm** — `npm install -g pnpm@<version>`, tracked against the npm datasource.
- **GitHub CLI** — `GH_VERSION=<version>`, tracked against `cli/cli` GitHub
  releases (the leading `v` on release tags is stripped via `extractVersion`).
- **`# renovate:`-annotated `ENV`/`ARG` pins** — via the upstream
  `customManagers:dockerfileVersions` preset, so lines like a
  `CLAUDE_CODE_VERSION` ARG tagged with a `# renovate: datasource=… depName=…`
  comment are kept up to date.

Everything else is handled by Renovate's native managers and needs no config:
Dockerfile `FROM` (tag + digest), `COPY --from` images (e.g. prek, cosign), and
Dev Container Features.

## Minimum release age

A `minimumReleaseAge` of **3 days** (with `internalChecksFilter: strict`) holds
back every update until the release has been public for three days — a
supply-chain cooling-off period in case a version is yanked or compromised
shortly after publish. `config:best-practices` already applies this to npm via
`security:minimumReleaseAgeNpm`; the top-level setting extends the same hold to
Docker images and GitHub releases (any datasource that exposes a release
timestamp).

## Note on file location

Renovate only auto-discovers its config at the **repo root** (`renovate.json`,
`.github/renovate.json`, etc.). The `node` dev container template ships its
`renovate.json` so that it lands at the applied repo's root — not under
`.devcontainer/` — for exactly this reason.
