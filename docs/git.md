# Git

The Git plugin in release-it, by default, does the following:

1. [Prerequisite checks](#prerequisite-checks)
1. [Files may be updated by other plugins and/or user commands/hooks]
1. `git add . --update`
1. `git commit -m "[git.commitMessage]"`
1. `git tag --annotate --message="[git.tagAnnotation]" [git.tagName]`
1. `git push [git.pushArgs] [git.pushRepo]`

When not in CI mode, release-it will ask for confirmation before each of the commit, tag, and push steps.

Configure the `[git.*]` options to modify the commands accordingly. See
[all options and their default values](../config/release-it.json).

The minimum required version of Git is v2.0.0.

## SSH keys & Git remotes

SSH keys and Git remotes are assumed to be configured correctly. If a manual `git push` from the command line works,
release-it should be able to do the same.

The following help pages might be useful:

- [SSH](https://help.github.com/articles/connecting-to-github-with-ssh/)
- [Managing Remotes](https://help.github.com/categories/managing-remotes/) (GitHub)
- [SSH keys](https://confluence.atlassian.com/bitbucket/ssh-keys-935365775.html) (Bitbucket)
- [SSH keys](https://gitlab.com/help/ssh/README.md) (GitLab)

## Remote repository

By default, `release-it` uses branch's tracking information, unless there isn't any, in which case it defaults to
`"origin"` as the remote name to push to. Use `git.pushRepo` to override this with a different remote name, or a
different git url.

## Tag name

The `v` prefix is automatically detected from the latest tag. No need to configure e.g. `git.tagName: "v${version}"`,
unless you specifically want to override or change this.

## Extra arguments

In case extra arguments should be provided to Git, these options are available:

- `git.commitArgs`
- `git.tagArgs`
- `git.pushArgs`

For example, use `"git.commitArgs": ["-S"]` to sign commits (also see
[#35](https://github.com/release-it/release-it/issues/350)).

Note that `["--follow-tags"]` is the default for `pushArgs`, so add this to the custom array of arguments.

## Skip Git steps

To skip the Git steps entirely (for instance, if you only want to `npm publish`), this shorthand is available:

```bash
release-it --no-git
```

Use e.g. `git.tag: false` or `--no-git.tag` to skip a single step.

## Untracked files

By default, untracked files are not added to the release commit. Use `git.addUntrackedFiles: true` to override this
behavior.

## Prerequisite checks

### Required branch

This is disabled by default, but release-it can exit the process when the current branch is not as configured:

```json
{
  "git": {
    "requireBranch": "master"
  }
}
```

Use an array to allow releases from more branch names.

### Clean working directory

The working directory should be clean (i.e. `git status` should say something like this:

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

Make sure to commit, stash, or revert the changes before running release-it. In case the currently staged changes should
be committed with the release commit, use `--no-git.requireCleanWorkingDir` or configure
`"git.requireCleanWorkingDir": false`.

### Upstream branch

If no upstream branch is known to Git, it does not know where to push the release commit and tag to, and halts.

Use `--no-git.requireUpstream` to add `--set-upstream [remote] [branch]` to the `git push` command, where `[remote]` is
the value of `git.pushRepo` ("origin" by default, if no upstream branch), and `[branch]` is the name of the current
branch. So if the current branch is `next` then the full command that release-it will execute becomes
`git push --follow-tags --set-upstream origin next`.

Configure `pushRepo` with either a remote name or a Git url to push the release to that remote instead of `origin`.

Disabling `git.requireUpstream` is useful when releasing from a different branch (that is not yet tracking cq present on
a remote). Or similar, when releasing a (new) project that did not push to the remote before. Please note that in
general you should not need this, as it is considered a best practice to release from the `master` branch only. Here is
an example use case and how it can be handled using release-it:

- After a major release (v2), a bug is found and a fix released in v2.0.1.
- The fix should be backported to v1, so a branch "v1" is made and the fix is cherry-picked.
- The release of v1.x.x can be done while still in this branch using `release-it --no-git.requireUpstream`.

### No commits

By default, release-it does not check the number of commits upfront to prevent "empty" releases. Configure
`"git.requireCommits": true` to exit the release-it process if there are no commits since the latest tag.

Also see the [Require Commits](./recipes/require-commits.md) recipe(s).

## Further custimizations

In case you need even more customizations, here is some inspiration:

```json
{
  "git": {
    "push": false
  },
  "hooks": {
    "after:git:release": "git push origin HEAD"
  }
}
```

Since the `after:git:release` hook runs after the Git commands, the `git.push` can be disabled, and replaced by a custom
script.
