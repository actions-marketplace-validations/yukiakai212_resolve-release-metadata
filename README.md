# Resolve Release Metadata

A GitHub Action that resolves release metadata from either a raw version string or a supported package manifest.

After an optional prefix is removed, the resolved value must be a valid semantic version. The action exposes the normalized version and its parsed components for use by later release, publish, or deployment steps.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `raw-version` | No | — | Version string to resolve directly. When non-empty, it takes precedence over `package-type` and `package-directory`. |
| `package-type` | In package mode | — | Manifest type to inspect when `raw-version` is blank. Supported values: `node` and `cargo`. |
| `package-directory` | No | `.` | Directory containing the selected package manifest. Used only when `raw-version` is blank. |
| `prefix-to-remove` | No | — | Exact prefix to remove once from the resolved version, such as `v`. If configured, the resolved version must begin with this prefix. |

## Outputs

| Output | Description |
| --- | --- |
| `release-tag` | Git release tag derived from the normalized version. |
| `full` | Full semantic version, including build metadata when present. |
| `version` | Semantic version without build metadata, for example `1.2.3-beta.1`. |
| `major` | Major version number. |
| `minor` | Minor version number. |
| `patch` | Patch version number. |
| `build` | Build metadata, or an empty string when it is absent. |

## Usage

Replace `<version>` with a released version of this action.

### Resolve a raw version

```yaml
- id: metadata
  uses: yukiakai212/resolve-release-metadata@<version>
  with:
    raw-version: v1.2.3-beta.1+build.7
    prefix-to-remove: v

- run: |
    echo "version=${{ steps.metadata.outputs.version }}"
    echo "full=${{ steps.metadata.outputs.full }}"
    echo "release-tag=${{ steps.metadata.outputs.release-tag }}"
```

### Resolve a Node.js package version

This reads the `version` field from `package.json` in the repository root.

```yaml
steps:
  - uses: actions/checkout@v7

  - id: metadata
    uses: yukiakai212/resolve-release-metadata@<version>
    with:
      package-type: node
```

### Resolve a Cargo package version

This reads `[package].version` from `crates/cli/Cargo.toml`.

```yaml
steps:
  - uses: actions/checkout@v7

  - id: metadata
    uses: yukiakai212/resolve-release-metadata@<version>
    with:
      package-type: cargo
      package-directory: crates/cli
```

## Resolution rules

1. If `raw-version` is non-empty, the action uses it and ignores the package inputs.
2. Otherwise, it reads the selected package manifest:
   - `node`: `<package-directory>/package.json`
   - `cargo`: `<package-directory>/Cargo.toml`
3. If `prefix-to-remove` is set, the action removes that exact prefix once. A missing prefix is an error.
4. The resulting value is parsed as a semantic version and exposed through the outputs above.

## License

MIT
