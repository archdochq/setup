# archdochq/setup

Installs the [ArchDoc](https://github.com/archdochq/archdoc) CLI and puts it on
`PATH`.

```yaml
- uses: archdochq/setup@v1
  with:
    version: v0.2.0
- run: archdoc lint
```

## Inputs

| Input | Default | |
| --- | --- | --- |
| `version` | `latest` | The release to install, such as `v0.2.0` |

Pin it. A specification repository should not have the rules it is held to
change underneath it, which is the same reason the workflow `archdoc init`
scaffolds carries a pinned version rather than tracking the latest release.

## Outputs

| Output | |
| --- | --- |
| `version` | The release that was installed, as a tag. Resolved from the latest release where `version` was `latest` |
| `path` | The directory the binary was installed into |

## What it does

Works out the runner's platform, downloads the matching archive from the
release, **verifies it against the release's `checksums.txt`**, extracts the
binary into the runner's temporary directory and adds that directory to
`PATH`. Linux, macOS and Windows, on amd64 and arm64.

The checksum is not optional. The binary is about to run over a repository,
and the verification is what says it is the one the release published. A
tampered archive fails the step rather than being installed.

The download happens outside the checkout. `curl -O` writes into the working
directory, which is the tree `actions/checkout` populated, and it opens with
`fopen("wb")`, which writes through an existing symbolic link: one committed
under the archive's name would otherwise redirect it.

Nothing is installed to a system path, so the action needs no privileges, and
what it writes goes with the runner.

## Full example

```yaml
name: archdoc

on: [push, pull_request]

permissions:
  contents: read

jobs:
  archdoc:
    runs-on: ubuntu-latest
    steps:
      # The full history is needed: the frozen-document rule compares every
      # terminal document against the branch, and a shallow checkout has no
      # commits to compare against.
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - uses: archdochq/setup@v1
        with:
          version: v0.2.0

      - run: archdoc lint
      - run: archdoc index --check
```

Where the specification lives in a subdirectory, give those steps a
`working-directory`, or run `archdoc -C <dir> lint`.

## Licence

MIT. ArchDoc itself is AGPL-3.0-or-later; this action only downloads a
published release and imposes nothing on what you run it against.
