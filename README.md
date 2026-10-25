# boost mirror

A boost.org release mirror using GitHub workflows and the release system. This repository mirrors the `source` (and optionally `binaries`) folders from `archives.boost.io`, excluding `alpha`, `beta`, and `rc` non-release candidates.

> [!NOTE]
> This solution aims to be an easy, reproducible release mirror. `alpha`, `beta`, and `rc` candidates are not mirrored.

## How it works

Two workflows handle initial bootstrapping and ongoing release checks:

```txt
bootstrap.yml
check_new_release.yml
```

Both workflows support two modes via the `include_binaries` input (or the `INCLUDE_BINARIES` repository variable for scheduled runs):

> [!IMPORTANT]
> **Source only (`include_binaries: false`, default)**
> Downloads the standard Boost release source archives (`.7z`, `.tar.bz2`, `.tar.gz`, `.zip` and their `.json` checksums) from the remote `source` folder.
> Works out of the box with the default `github.token` (no extra permissions or PAT needed in a newly forked repo) and takes just over 60 seconds to bootstrap.

> [!CAUTION]
> **Source and binaries (`include_binaries: true`)**
> Downloads all release files (excluding `_rc`) from both the remote `source` and `binaries` folders.
> Requires a Personal Access Token (PAT) with minimal `contents: write` permissions to avoid API rate limiting (~5 minutes to bootstrap large binary files).
>
> In your repository settings:
> 1. Create a repository secret named `BOOST` containing your PAT (`${{ secrets.BOOST }}`).
> 2. For scheduled runs with binaries, create a repository variable named `INCLUDE_BINARIES` set to `true`.

## Summary

To bootstrap and maintain a mirror:

1. Fork the repository and run [bootstrap.yml](.github/workflows/bootstrap.yml) manually (`workflow_dispatch`). It discovers all Boost releases from `1.63.0` to the current latest release on `boostorg/boost`, downloads the assets from `archives.boost.io` in parallel matrix jobs, creates tagged releases (`boost-1.x.0`), and marks the newest version as the latest release.
2. Enable the `schedule` trigger in [check_new_release.yml](.github/workflows/check_new_release.yml) to automatically poll for and mirror new Boost releases as they become available:

```yaml
  schedule:
    - cron: "*/30 */1 * * *"
```

## Credits

Dynamic matrix arrays: https://www.kenmuse.com/blog/dynamic-build-matrices-in-github-actions/
