# setup-norcube

Install the [Norcube CLI](https://github.com/norcubeplatform/cli) (`norcube`,
alias `nrc`) in a GitHub Actions job, verified against the release's
checksums, and optionally give it a CI token.

```yaml
- uses: norcubeplatform/setup-norcube@v1
  with:
    token: ${{ secrets.NORCUBE_TOKEN }}
- run: nrc langsync sync
```

Works on Linux and macOS runners (x64 and ARM64) and Windows (x64).

## Inputs

| Input | Default | Description |
|---|---|---|
| `version` | `latest` | CLI version, e.g. `v0.18.0` or `0.18.0`, or `latest`. Pin it for reproducible builds. |
| `token` | none | A Norcube CI token (`nrc_ci_…`). Exported as `NORCUBE_TOKEN` for the rest of the job and masked in logs. |

## Outputs

| Output | Description |
|---|---|
| `version` | The installed version, e.g. `v0.18.0`. |
| `path` | Directory with `norcube` and `nrc`, added to `PATH`. |

## Getting a CI token

A CI token belongs to an organization, carries only the scopes you give it
and always expires. Create one in the dashboard under IAM › CI tokens, or
with the CLI:

```bash
nrc auth ci-token create --name "GitHub Actions" --scopes langsync.sync -o text | gh secret set NORCUBE_TOKEN
```

`nrc auth ci-token scopes` lists the scopes. Give each workflow its own
token with the narrowest scopes it needs, so you can revoke one without
touching the others.

## Examples

Keep translation files in sync on every push to `main`:

```yaml
name: translations
on:
  push:
    branches: [main]
jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: norcubeplatform/setup-norcube@v1
        with:
          token: ${{ secrets.NORCUBE_TOKEN }}
      - run: nrc langsync sync
      - name: Commit updated translations
        run: |
          git config user.name github-actions
          git config user.email github-actions@users.noreply.github.com
          git add -A locales
          git diff --cached --quiet || { git commit -m "Update translations"; git push; }
```

Fail a nightly job when a restore check of any database didn't pass:

```yaml
- uses: norcubeplatform/setup-norcube@v1
  with:
    token: ${{ secrets.NORCUBE_TOKEN }} # scope: backup.read
- run: |
    failing=$(nrc backup health -o json --query "length([?restoreCheckPassed < restoreCheckTotal])")
    test "$failing" -eq 0
```

## Security

- The archive is checked against the release's `checksums.txt` (SHA-256)
  before anything runs.
- The token is masked in logs and only written to the job's environment.
- Pin this action to a release tag or a commit SHA, like any third-party
  action.

## License

Apache 2.0.
