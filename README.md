# wererootops/actions

Reusable composite actions for wererootops pipelines.

## Philosophy

- One action, one responsibility
- Zero runtime dependencies beyond what GitHub Actions provides
- Structured logging: `key=value` syslog-style lines, parseable with `awk`
- All actions pinned by commit hash in consumers, never by tag
- Supply chain: every release tagged, Dependabot enabled

## Actions

### `checksum`

Generate SHA256 checksum for a file.

```yaml
- uses: wererootops/actions/checksum@HASH
  with:
    path: "path/to/binary"
    output_file: "binary.sha256"  # optional, default: sha256.txt
```

### `sign`

Sign a binary with cosign keyless via Sigstore OIDC. Requires `id-token: write` permission. Signature recorded in Rekor transparency log (public).

```yaml
- uses: wererootops/actions/sign@HASH
  with:
    path: "path/to/binary"
    bundle_output: "binary.cosign.bundle"
    cosign_version: "v3.0.6"
    cosign_sha256: "c956e5dfcac53d52bcf058360d579472f0c1d2d9b69f55209e256fe7783f4c74"
```

### `sast-rust`

Static analysis with `cargo audit` against the RustSec advisory database. Reads `.cargo/audit.toml` automatically if present.

```yaml
- uses: wererootops/actions/sast-rust@HASH
  with:
    manifest_path: "path/to/Cargo.lock"
    output_file: "binary.sast.json"  # optional
    deny_warnings: "false"           # optional
```

### `release`

Package a signed, verified artifact for handoff to CD. Produces a directory with consistent naming: `{name}-{version}-{arch}-{os}-{variant}`.

```yaml
- uses: wererootops/actions/release@HASH
  with:
    binary_path: "bundle/vaultwarden"
    binary_name: "vaultwarden"
    version: "1.35.7"
    sha256_path: "bundle/vaultwarden-1.35.7-x86_64-linux-static.sha256"
    sbom_path: "bundle/vaultwarden-1.35.7-x86_64-linux-static.sbom.cdx.json"
    cosign_bundle_path: "bundle/vaultwarden-1.35.7-x86_64-linux-static.cosign.bundle"
    sast_report_path: "bundle/vaultwarden-1.35.7-x86_64-linux-static.sast.json"
    output_dir: "rc"
```

### `pin`

Resolve a GitHub Action tag to its commit SHA for pinning.

```yaml
- uses: wererootops/actions/pin@HASH
  with:
    repo: "actions/checkout"
    tag: "v6.0.2"          # optional, resolves latest if omitted
```

Outputs: `sha`, `pin` (ready-to-paste reference with version comment).

## Pinning consumers

Always pin by commit hash, never by tag. Tags are mutable.

```yaml
# Wrong
uses: wererootops/actions/checksum@v1.0.0

# Correct
uses: wererootops/actions/checksum@8c1bc10217c5f7b1069036d3f71dad903a314c46  # v1.0.0
```

To resolve a hash for a new pin, use the `pin` action or:

```bash
gh api /repos/wererootops/actions/git/refs/tags/v1.0.0 --jq '.object.sha'
```

## Bump runbook

1. Dependabot opens a PR with the new hash
2. Review the diff and the release notes of the updated action
3. Merge if the change is safe
4. The pin comment `# vX.Y.Z` is updated automatically by Dependabot

## Artifact naming convention

```
{name}-{version}-{arch}-{os}-{variant}.{type}

vaultwarden-1.35.7-x86_64-linux-static
vaultwarden-1.35.7-x86_64-linux-static.sha256
vaultwarden-1.35.7-x86_64-linux-static.sbom.cdx.json
vaultwarden-1.35.7-x86_64-linux-static.cosign.bundle
vaultwarden-1.35.7-x86_64-linux-static.sast.json
vaultwarden-1.35.7-x86_64-linux-static.manifest.json
```
