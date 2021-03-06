# Git branches

## One Flow

We use the [OneFlow variation](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow#variation-develop-master) model for organizing branches. We have elected to use the "develop + master variation". There are two long-running branches, `master`, and `develop`.
From there we branch off to create _feature_ branches and _hotfix_ branches. We use git tags extensively to mark deployable commits.

Using this approach, the `develop` branch reflects the current state of the product,
and all integrations for features will be against this branch. The `master` branch
is used as a repository of tagged stable releases. This allows users who are less
experienced, and just want to grab the stable version to use `master` as the source,
and more experienced users to pull from develop.

MoOde has not been releasing hotfixes in the past, and may or may not do so in
the future. If they are issued against a released version, they will be updated
in `master`.

Hotfixes usually increment the patch point of the current version they branched from, AKA they go from `3.12.1` to `3.12.2`.  
Releases made from `develop` increase either the minor or the major point, depending on their nature, for instance from `3.12.2` to `3.13.0`.

The following two aliases might help to get the system right.  
The `hotfix` one especially helps to not forget to branch off a point in the past when creating a hotfix branch.

TODO: These are untested. They were written for checkouts from `master`, but need
to account for baseline of `develop`
```
[alias]
  feature = "!f() { git checkout -b 'feature/'$(git config user.name | cut -d' ' -f1 | awk '{print tolower($0)}')'-'$1; }; f develop"
  hotfix = "!f() { [ x$# != x2 ]&&echo "Two arguments required!" >&2 || git checkout -b 'hotfix/'$(git config user.name | cut -d' ' -f1 | awk '{print tolower($0)}')'-'$1 $2; }; f develop"

```

And if you accidently create a branch based off of `master` rather than `develop`
you can use the following commands to get back in sync:
```
git branch --set-upstream-to=develop my-feature
TODO - figure out the rest :-)
```

Note that we might like to prefix branches using the name of the author, to see at a glance who to ask about branches.


## Pulling changes

Whenever you have local commits, but the remote also includes new work, you have diverged.

When doing a regular `git pull` you will be dropped into your editor to create a merge commit.

- ABORT immediately. Clear out the commit message and quit your editor.
- Use `git merge --abort` to discard your merge state.
- Use `git pull --rebase` instead to pull changes but avoid the noise of a merge commit.

## Lifecycle of a feature branch

- Optimally, create an issue to describe the work, and assign it to yourself.
this avoids having two team members working on the same topic without knowing of
the other's efforts.
- Start the feature: `git checkout -b my-feature`
- The convention for my-feature is feature/name-description
- Do your commits.
- Rebase frequently to incorporate upstream changes.
  - `git fetch origin`
  - `git rebase origin/develop`
- Push to the remote: `git push -u origin my-feature`
- Create a Pull Request on Github.
- When the PR has been approved, use interactive rebasing to clean up your history (see **Cleaning up your branch **below).
- Merge the PR and delete the branch, preferably using Github's **Rebase and Merge** option, in order to avoid the noise of a merge commit.

## Lifecycle of a hotfix branch

It's exactly the same as starting a feature, with the notable difference you will want to fork from some previously released version.

Let's say the current production version is tagged `1.50.4` and you've found a critical bug.

TODO: I think the tagging will happen on `develop` and then a copy will be made on
`master`. So a hotfix would need to be made against `develop` at the point of the tag
and then released to `master`, but then at what point will the integration/merge take
place in `develop`. Or will the hotfix be to `master` with a port back into `develop`?

- Create a hotfix branch, branched off the previous production version: `git checkout -b my-fix 1.50.4`
- Do your commits as usual.
- Push to the remote: `git push -u origin my-fix`
- Create a Pull Request on Github. Mention this is a hotfix.
- When the PR has been approved, tag your HEAD: `git tag 1.50.5`
- Merge the branch, either on Github or on the CLI, but make sure to use a regular merge, with a commit: `git merge my-fix`
- Push the tag: `git push origin --tags`
- If applicable, remove the remote branch: `git push origin :my-fix`
- Push `master`: `git push origin master`

You can now deploy the new tag to production.

## Rebasing

- You're working on a feature branch
- Meanwhile, develop has been updated and you wish to include the changes in your branch
- Go back to develop: `git checkout develop`
- Pull develop to make sure you're up-to-date: `git pull origin develop`
- Return to the feature branch: `git checkout my-feature`
- `git rebase develop`
- If there are conflicts, resolve them and use:
  - `git add .` But don't commit!
  - `git rebase --continue`
- If there's a remote tracking your branch, use `git push --force-with-lease`. Only force-push if you're sure there's nobody else working on the same branch!
- Note: git will complain about being diverged every time you change history (whenever you amend a commit or rebase a branch). It will suggest you should pull then push, but DO NOT PULL. It's YOUR feature branch, you know nobody else has worked on it. If not, you shouldn't be changing history.

## Cleaning up your branch

💡 Pro-tip: grab this git alias to drop into interactive rebase starting at the first commit of your branch: [https://github.com/harmenjanssen/dotfiles/blob/master/gitconfig#L46](https://github.com/harmenjanssen/dotfiles/blob/master/gitconfig#L46)

For example `git rbi develop` would find the commit where you branched off `develop` and allows you to rebase all commits from that point on.

Git will drop you into your configured editor where it will list the commits within the specified range.

By default all will be prefixed with the word `pick`. If you would leave this alone and simply save and quit the file, everything would stay the same.

You can, however, use various other keywords to change the history of your branch:

`pick` = use commit

`reword` = use commit, but edit the commit message

`edit` = use commit, but stop for amending

`squash` = use commit, but meld into previous commit

`fixup` = like "squash", but discard this commit's log message

`exec` = run command (the rest of the line) using shell

`drop` = remove commit

After saving the file Git will replay or drop your commits and/or smoosh 'm together like you instructed. When using `squash` or `reword` you will be prompted to come up with a new commit message.

Note: see the general advice above in the **Rebasing** section about force-pushing et cetera.

Note: you can swap the order of commits by simply reordering them in the rebase file.

Note: if you want to edit a commit message, you have to choose `reword`. Changing the commit message directly in the rebase file won't work. You will be prompted for a new message after saving the rebase file.
