# `savi-lang/action-build-release`

Build Savi application release binaries for multiple platforms from a single CI runner.

## Example

If you have a repository with a Savi application manifest named `my-app`, then you could use a workflow like this one to give a workflow that can be triggered manually at any time to tag a new release and upload release binaries as assets attached to the release:

```yaml
name: release

on:
  workflow_dispatch:
    inputs:
      version-tag:
        description: |
          The name of the version to release (e.g. `v1.2.3` or `v0.20220131.0`).
        required: true

jobs:
  all:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: savi-lang/action-install@v1.0.0

      # Build the release binaries for all platforms and package in tarballs.
      - uses: savi-lang/action-build-release@v1
        id: my-app
        with:
          manifest-name: my-app
          tarball-name: my-app-${{ github.event.inputs.version-tag }}
          all-platforms: true
          macosx-accept-license: true
          windows-accept-license: true

      # Tag the new release and upload the files from the tarball directory.
      - uses: softprops/action-gh-release@v1
        if: ${{ github.event.inputs.version-tag != '' }}
        with:
          tag_name: ${{ github.event.inputs.version-tag }}
          generate_release_notes: true
          token: ${{ secrets.BOT_GITHUB_TOKEN }} # (allows triggering workflows)
          fail_on_unmatched_files: true
          files: |
            ${{ steps.my-app.outputs.tarball-directory }}/*
```
