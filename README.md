# Resolve Release Metadata

A GitHub Action that resolves release metadata from either a raw version string or a supported package manifest.

It optionally removes one exact prefix, then parses the remaining text as a semantic version. The action exposes the semantic version and its parsed components for use by later release, publish, or deployment steps.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `raw-version` | No | - | Version text to parse directly. When non-empty, it takes precedence over `package-type` and `package-directory`. Whitespace is not trimmed. |
| `package-type` | In package mode | - | Manifest type to inspect when `raw-version` is blank. Supported values: `node` and `cargo`. |
| `package-directory` | No | `.` | Directory containing the selected package manifest. Used only when `raw-version` is blank. |
| `prefix-to-remove` | No | - | Exact literal prefix required at the start of the version and removed once, such as `v`. Whitespace is not trimmed. |

## Outputs

| Output | Description |
| --- | --- |
| `release-tag` | Git release tag derived from the semantic version. |
| `full` | Full semantic version, including build metadata when present. |
| `version` | Semantic version without build metadata, for example `1.2.3-beta.1`. |
| `major` | Major version number. |
| `minor` | Minor version number. |
| `patch` | Patch version number. |
| `build` | Build metadata, or an empty string when it is absent. |

## Strict input handling

Version text and `prefix-to-remove` are used exactly as supplied; they are never trimmed. This keeps invalid release data visible instead of silently correcting it.

For example, `" v1.2.3 "` fails rather than becoming `v1.2.3`, and `prefix-to-remove: " v"` represents a prefix with a literal leading space.

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
3. Version text and `prefix-to-remove` are not trimmed.
4. If `prefix-to-remove` is set, the action removes that exact prefix once. A missing prefix is an error.
5. The resulting value is parsed as a semantic version and exposed through the outputs above.

## License

MIT
