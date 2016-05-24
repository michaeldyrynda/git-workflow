# My Awesome Project
## Version 1.0.3

* [Feature development](#feature-development)
 * [Meaningful commit messages](#meaningful-commit-messages)
* [Release candidacy](#release-candidacy)
* [Resolving merge conflicts](#resolving-merge-conflicts)
 * [Rebasing feature branches](#rebasing-feature-branches)
* [Preparing the final release](#preparing-the-final-release)
* [Tagging a release](#tagging-a-release)

This is my awesome project.

It is being used to test out our proposed new development workflow.

We have one mainline branch, `master`.

<a name="feature-development"></a>
### Feature development

New feature development should be branched from `master`.

`git checkout -b feature/my-new-feature master`

Work is then completed on the `feature/my-new-feature` branch. When you're ready to have your code reviewed, make a Pull Request against the `master` branch.

```
git add -A
git commit -m 'add changes for my new feature'
git push -u origin feature/my-new-feature
```

<a name="meaningful-commit-messages"></a>
#### Meaningful commit messages

It is important when making commits to leave meaningful commit messages. Meaningful commit messages makes it easy for somebody not familiar with the code changes to have a good understanding of changes that are being made through the development process. It also makes the release manager's job simpler when it comes time for them to write release notes, as they are provided with a concise overview of everything that was changed, added, or removed, just by looking at the git log between the previous and current releases.

There are seven rules that ought to be followed when writing your commit messages:

> 1. Separate subject from the body with a blank line
> 2. Limit the subject line to 50 characters
> 3. Capitalise the subject line
> 4. Do not end the subject line with a period
> 5. Use the imperative mood in the subject line
> 6. Wrap the body at 72 characters
> 7. Use the body to explain *what* and *why* vs *how*

More information, and a breakdown of the seven rules, on writing meaningful commit messages can be found [here](http://chris.beams.io/posts/git-commit/).

View the project page within GitHub and create a new Pull Request, which targets the `master` branch.

This Pull Requests's code will be reviewed by one of the organisation team members responsible for QA.

![QA labelling](qa-labels.png)

* If the Pull Request passes QA, the `QA Passed` label will be applied to it.
* If the Pull Request fails QA, the `QA Failed` label will be applied to it.

<a name="release-candidacy"></a>
### Release candidacy

When a sufficent number of features are in the `QA Passed` stage, the designated release manager will be prepare a new release candidate. The release manager should clone a *fresh* copy of the remote repository when preparing *each* new release, rather than using an existing working copy.

The release candidate should be named for the next applicable version, based on the changes being implemented following the [SemVer](http://semver.org) versioning style.

> Given a version number MAJOR.MINOR.PATCH, increment the:
> 
> 1. MAJOR version when you make incompatible API changes,
> 2. MINOR version when you add functionality in a backwards-compatible manner, and
> 3. PATCH version when you make backwards-compatible bug fixes.
>
> Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

The pre-release label will be incremented sequentially - starting at `1` - for each release target.

* `release/1.0.1-rc1` is the first release candidate targeting the patch version `1.0.1`
* `release/1.0.1-rc2` is the second release candidate targeting the patch version `1.0.1`
* `release/1.1.0-rc1` is the first release candidate targeting the minor version `1.1.0`
* `release/2.0.0-rc1` is the first release candidate targeting the major version `2.0.0`

```
git checkout git@github.com:Hostworks/my-awesome-project.git my-awesome-project-rc
cd my-awesome-project-rc
git checkout -b release/1.0.1-rc1
git fetch --all
```

Feature branches are merged into this release candidate in the checked out working directory - not via GitHub - and using the `--no-ff` option.

`git merge --no-ff origin/feature/my-new-feature`

This allows additional features to continue development for the same release candidate simultaneously.

```
git fetch --all
git merge --no-ff origin/feature/about-release-candidacy
```

Whilst preparing a release candidate, the release manager will need to address any [merge conflicts](#resolve-merge-conflicts), which may occur when the same files are being modified by multiple Pull Requests. These merge conflicts should be trivial to resolve in most cases, and in some instances may be resolved by git automatically.

Once the release candidate is ready for user acceptance testing, the release branch should be pushed to GitHub.

`git push -u origin release/1.0.1-rc1`

This branch is then deployed to the staging environment for user acceptance testing, often using the relevant Jenkins deploy project.

Each individual feature should be marked with the `UAT Passed` or `UAT Failed` label as it runs the UAT gauntlet.

Any release candidate that has features which fail UAT should be deleted. GitHub will automatically close the accompanying Pull Request.

```
git checkout master
# Delete the unmerged release candidate branch locally
git branch -D release/1.0.1-rc1
# Delete the unmerged release candidate branch in GitHub
git push origin :release/1.0.1-rc1
```

If the release is to go ahead in the absence of the failed features, the release candidate process shall start over.

```
git checkout -b release/1.0.1-rc2 master
git merge --no-ff origin/feature/my-new-feature
git push -u origin release/1.0.1-rc2
```

<a name="resolving-merge-conflicts"></a>
### Resolving merge conflicts

From time to time when preparing a release candidate, you may encounter merge conflicts. This generally occurs only when you have multiple Pull Requests making modifications to the same file(s). It is at this point that the `Needs Rebase` should be applied.

An in-progress merge with conflicts can easily be dealth with:

`git merge --abort`

This will return the release candidate branch to it's unsullied state, ready for the release manager to continue merging other Pull Requests.

Push the release candidate to GitHub so that developers will be able to rebase their feature branches against it.

`git push -u origin release/1.0.0-rc1`

Add a note to the Pull Request in GitHub stating that you cannot merge the branch due to conflicts with other changes being implemented in the current release, then assign the Pull Request to the requesting developer.

> The developer will need to rebase their branch against the release candidate and push their changes, as their understanding of the changes will be better than that of the release manager. It is for this reason that it is the developer's responsibility to resolve conflicts in their branch before pushing their branch.

```
git checkout feature/about-release-candidacy
git fetch --all
git rebase origin/release/1.0.1-rc1
# A force push will be required as rebasing is effectively rewriting git history
git push -f
```

If you encounter any conflicts whilst rebasing, you must resolve these in your feature branch. Git will effectively pause the rebase, allowing you to resolve these conflicts. Once resolved, you then stage your changes continue the rebase. Repeat this process for as many conflicts as you encounter until the rebase is complete.

```
git add <file_in_conflict>
git rebase --continue
```

<a name="rebasing-feature-branches"></a>
#### Rebasing feature branches

There are two ways to resolve merge conflicts between a feature branch and pending release branches; merging and rebasing.

Using a `merge` to bring in upstream changes often results in an untidy commit history littered with 'merging' commit messages. These merge commit messages will usually be accompanied by 'resolving conflicts' style commits, as well.

Using a `rebase` for the same process gives you the benefit of a clean commit history, as well as a linear history for feature branches, without the clutter of 'merging' commit messages.

A rebase works in the following way:

> 1. Git rolls back your commits and sets them aside
> 2. Git advances the unmodified tail of your (feature) branch to the head of the master branch
> 3. Git replays your changes on top of this new tail
> 4. You resolve, step-by-step, any conflicts as those changes are replayed
> 5. You have the option to "squash" multiple commits into one commit as things are replayed, simplifying the branch

For the purposes of our workflow, you **must never** merge an upstream branch into a feature branch. **Always** use rebase.

Prior to submitting a Pull Request, you may also be inclined to use an interactive rebase in order to squash any progress commits you've created within your feature branch.

The common workflow when rebasing a feature branch is as follows:

> 1. Create a feature branch
> 2. When work on branch is complete, rebase the branch, squashing small commits (you may need to do this more often if the branch is long-lived)
> 3. Merge the rebased branch into master (which should be trivial, because the rebase effectively merged master into branch)

You can read more about `git rebase` [here](https://dotdev.co/git-rebase-for-reasonable-developers-26dc8776dc25).

<a name="preparing-the-final-release"></a>
### Preparing the final release

The final release is submitted as a Pull Request from the release branch targetting the `master` branch.

The only changes that maybe be implemented directly against the release branch, at the release manager's discretion, are bug fixes and version number changes as appropriate.

Any final changes should be pushed to the release candidate prior to the Pull Request being merged into `master`.

Upon merging of the release candidate branch into `master`, the merged feature branch Pull Requests will be automatically closed.

> **Note**: You must perform a merge, not a squash and merge, otherwise Pull Requests will not be closed automatically.

<a name="tagging-a-release"></a>
### Tagging a release

Once the release candidate is merged into `master`, a new release should be tagged matching the targeted release version.

![Version tagging screenshot](version-tagging.png)

Tag the release, which will be deployed via the relevant Jenkins deployment project.
