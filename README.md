# publish-nix-image

Reusable GitHub Actions workflow that publishes a Nix flake's `dockerTools` image as a multi-arch OCI manifest. Builds on native amd64 and arm64 runners, combines into a single manifest, pushes to your registry.

## Usage

```yaml
name: Docker
on:
  push:
    branches: [master]
jobs:
  publish:
    uses: nicolaschan/publish-nix-image/.github/workflows/build-push.yml@master
    permissions:
      contents: read
      packages: write
    with:
      image: ghcr.io/nicolaschan/my-app
```

Your flake needs an attribute (default `.#docker`) that produces a `dockerTools` image:

```nix
packages.x86_64-linux.docker = pkgs.dockerTools.buildImage {
  name = "my-app";
  config.Cmd = [ "${self.packages.x86_64-linux.default}/bin/my-app" ];
};
```

Each run pushes two tags: `:latest` and `:<ref>-<run_id>`.

## Inputs

| Input        | Required | Default     | Description                                       |
| ------------ | -------- | ----------- | ------------------------------------------------- |
| `image`      | yes      | —           | Destination image, e.g. `ghcr.io/org/name`        |
| `flake-attr` | no       | `.#docker`  | Flake attribute producing a `dockerTools` image   |

## Custom flake attribute

```yaml
with:
  image: ghcr.io/nicolaschan/my-app
  flake-attr: .#packages.x86_64-linux.container
```

## Notes

- The caller must grant `permissions: packages: write`. Authentication uses `secrets.GITHUB_TOKEN`.
- **`:latest` is pushed on every run, including PRs.** Gate with `if: github.event_name == 'push'` on the calling job if that's not what you want.
- Per-arch images are pushed to an ephemeral in-job registry first, so only the combined manifest ever appears on your public registry — no `tag-amd64` / `tag-arm64` clutter.
- Uses `ubuntu-24.04-arm` for arm64: free on public repos, varies by plan on private ones.
