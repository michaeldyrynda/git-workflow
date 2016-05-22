# My Awesome Project
## Version 1.0.0

This is my awesome project.

It is being used to test out our proposed new development workflow.

We have one mainline branch, `master`.

### Feature development

New feature development should be branched from `master`.

`git checkout -b feature/my-new-feature master`

Work is then completed on the `feature/my-new-feature` branch. When you're ready to have your code reviewed, make a Pull Request against the `master` branch.

```
git add -A
git commit -m 'add changes for my new feature'
git push -u origin feature/my-new-feature
```

View the project page within GitHub and create a new Pull Request, which targets the `master` branch.

This Pull Requests's code will be reviewed by one of the organisation team members responsible for QA.

![QA labelling](qa-labels.png)

* If the Pull Request passes QA, the `QA Passed` label will be applied to it.
* If the Pull Request fails QA, the `QA Failed` label will be applied to it.

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

Feature branches are merged into this release candidate in the checked out working directory - not via GitHub.

`git merge origin/feature/my-new-feature`

This allows additional features to continue development for the same release candidate simultaneously.

```
git fetch --all
git merge origin/feature/about-release-candidacy
```

It is at this point that the release manager will need to address any merge conflicts, which should occur only when the same files are being modified by multiple Pull Requests as in this readme file. These merge conflicts should be trivial to resolve in most cases, and in some instances git may automatically resolve them for you.

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
git merge origin/feature/my-new-feature
git push -u origin release/1.0.1-rc2
```
