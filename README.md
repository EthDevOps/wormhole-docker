# wormhole-page Docker builds

Docker images of [wormhole-page](https://github.com/lucamartinetti/wormhole-page) — an end-to-end encrypted file transfer web UI on top of [Magic Wormhole](https://magic-wormhole.readthedocs.io/) — built automatically from upstream commits.

The goal is a stable image our deployments can pin and auto-update from. There are no code changes or patches: the images are upstream's code, built with upstream's own `Containerfile`.

## Images

```
ghcr.io/ethdevops/wormhole-docker:<short-sha>
```

| Tag         | Points to                                                     |
|-------------|---------------------------------------------------------------|
| `<short7>`  | exactly that upstream commit (e.g. `71b243b`)                |
| `latest`    | newest commit on upstream `main`                              |

Upstream has no releases or version tags, so the commit short-SHA is the only stable handle.

## How it works

The only file here is [`.github/workflows/docker.yaml`](.github/workflows/docker.yaml). Every 6 hours it:

1. Resolves the head of upstream `main` (plus other branches, but only `main` moves `latest`).
2. Stops if this repo already has a release named after that short-SHA.
3. Otherwise checks out upstream at that commit, builds the `Containerfile` (Rust server + WASM client), and pushes the image to GHCR.
4. Creates a GitHub release named `<short-sha>` here. Update tooling watching this repo's releases picks up each new build.

The release is created only after the image has been pushed, so a release always has a pullable image behind it.

## Manual runs

Actions → Docker → *Run workflow*. Leave `ref` empty to build the current `main` HEAD, or enter an upstream branch or full commit SHA to build something else. Builds of other refs or already-released commits do not move `latest`.

To rebuild a commit that has already been released, delete its GitHub release first. To rebuild without touching any tags (e.g. after a transient build failure), set `no_latest`: only `<short-sha>-<run-number>` is pushed and no release is created.

## Caveats

- Upstream has no releases; every `main` commit becomes a new image. If upstream breaks, pin to the last known good `<short-sha>` and skip forward until it's fixed.
- Only `main` is tracked on schedule. Fixes on upstream feature branches are built manually with `ref`.
- GitHub disables scheduled workflows in public repos after 60 days without commits. Re-enable it in the Actions tab, or commit something occasionally.
