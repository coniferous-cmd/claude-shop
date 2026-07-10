# claude-code-mirror

Mirror [Claude Code](https://code.claude.com/) native binaries to GitHub Releases.

Upstream binaries are published at `https://downloads.claude.ai/claude-code-releases`.
This repo runs a scheduled GitHub Action to grab the latest version and republish
every platform's binary as a GitHub Release asset, so they're available as
direct `https://github.com/.../releases/download/...` URLs.

## Triggered by

- `cron: 0 8 * * *` UTC — every day at 08:00 UTC
- `workflow_dispatch` — manual run from the Actions tab

## What gets mirrored

All platforms from the upstream manifest are mirrored (automatically picks up
any new ones Anthropic adds):

- `darwin-arm64`
- `darwin-x64`
- `linux-arm64`
- `linux-x64`
- `linux-arm64-musl`
- `linux-x64-musl`
- `win32-arm64`
- `win32-x64`

## Release assets

A release tag follows the upstream version, e.g. upstream `2.1.206` → tag `v2.1.206`.

Assets in each release:

- `claude-<version>-<platform>[.exe]` — one per platform (win32 adds `.exe`)
- `manifest.json` — upstream manifest as-is
- `SHA256SUMS` — sha256 of the binary assets, `sha256sum -c SHA256SUMS` friendly

## Verifying a download

```bash
# From a release's "Assets" section, download SHA256SUMS + the binary you want.
sha256sum -c SHA256SUMS --ignore-missing
```

Every binary's checksum is also validated by the workflow against the upstream
manifest before upload — a mismatch fails the run and no release is created.

## Idempotency

Before downloading, the workflow checks if a `v<version>` release already exists.
If it does, the run exits early. So the daily cron is a no-op when upstream hasn't
shipped a new version.

## Notes

- Upstream (`downloads.claude.ai`) is geo-restricted to [Anthropic's supported
  countries](https://www.anthropic.com/supported-countries). GitHub-hosted
  runners run in Azure regions where the endpoint is reachable.
- The workflow does **not** install or execute Claude — it only downloads,
  verifies checksum, and republishes.