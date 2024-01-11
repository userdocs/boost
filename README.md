# boost mirror

A boost.org release mirror using Github workflows and the release system. This example repo has bootstrapped the `source` and `binary` folders of the remote with the exception `rc` or non release assets.

> [!NOTE]
> This solution aims to be an easy reproducible release mirror so if it's not a release asset we are not interested in it. `alpha`,`beta` and `rc` candidates are not releases.

## How it works

There are 2 pairs of workflows that achieve different goals for a mirror repository. Pairs are linked as `source` or `source` and `binary`

```txt
bootstrap_source.yml (pair 1)
bootstrap_source_binary.yml (pair 2)
check_new_release_source.yml (pair 1)
check_new_release_source_binary.yml (pair 2)
```

> [!IMPORTANT]
> The workflows `source` will only get a specific set of boost release source code archives from the remote `source` folders. If you need a header only or source only mirror.

> [!TIP]
> This is what most projects will be using. The packaged source archives with the headers or used to build the components they need.

> [!NOTE]
> The `source` workflows work with just the `github.token` and need no special permissions in a newly forked repo
> It takes just over 60 seconds to bootstrap the releases.

> [!IMPORTANT]
> The workflows `source and binary` will get all files except `alpha`,`beta` and `rc` files from the remote `source` and `binary` folders.

> [!CAUTION]
> The `source` and `binary` workflows will not work with just the `github.token` and need a PAT created with minimal permissions to overcome api rate limiting.
> It takes just around 5 minutes to bootstrap this and involves some large files.

Finally, you have the corresponding automated check release workflows which will create new releases in the same format as the selected bootstrap workflow.

After a repo is bootstrapped we focus on the new releases and will never use that workflow again so it is required to be manually triggered.

## Summary

In Summary, to bootstrap a source only repo:

1. The [bootstrap_source.yml](https://github.com/userdocs/boost/blob/main/.github/workflows/bootstrap_source.yml) will determine the latest release and create an array of versions from `1.63.0`, the earliest version hosted on either `boostorg.jfrog.io` or `archives.boost.io`, through to current the latest version.
2. This will fire a matrix job using that array and download the main assets and upload them as tagged releases, using a tag version that matches the <https://github.com/boostorg/boost/tags> format.
3. Remote latest release tag will be set as the local latest release tag.
4. A scheduled [check_new_release_source.yml](https://github.com/userdocs/boost/blob/main/.github/workflows/check_new_release_source.yml) will check for new releases and mirror them as they are released.

> [!NOTE]
> Source location redundancy starting with `boostorg.jfrog.io` falling back to `archives.boost.io`

So just fork the repo, run the boost [bootstrap.yml](https://github.com/userdocs/boost/blob/main/.github/workflows/bootstrap_source.yml)

Then edit the paired check release workflow to enable the scheduler.

You now have mirror that will update itself when new releases are available.

## Need to know

> [!WARNING]
> The scheduled part of the workflows is commented out until you decide which one to use.
>
> ```
>  # schedule:
>  #   - cron: "*/30 */1 * * *"
> ```

## Credits

Dynamic matrix arrays: https://www.kenmuse.com/blog/dynamic-build-matrices-in-github-actions/
