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
