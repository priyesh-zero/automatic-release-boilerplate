# Automatic Release POC

> Priyesh Shrivastava

[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)

## Why?

Maintaining different versions of the code is a standardized practice, with lot of conventions like semantic-versioning, conventional-commits, change-logs widely used by most development teams. This helps keep the code organized into different versions and proper commits. Having proper versioning gives it an even bigger boost of having code categorized into sets of commits that shows the type of change is withing them. With a proper committing strategy, it makes easier for reviews and other developers to understand the type, and reason for the change made in the commit.

Doing all this manually, and even more enforcing it on the whole team to do, and follow the proper convention is not easy, hence automating as much as possible makes the most sense.

## How?

To automate the process, we use as much of existing technologies available to us like, git-hooks, ci-pipelines and npm-packages. To achieve this the following things have been setup

### Conventional Commits

The first step is to make sure the commits that are made to the repository are in the correct format, to this there is a `commit-msg` hook in git, that runs after the commit message as been set, it checks the commit is in convention format and aborts the commits if it is not. For working with `git-hooks`, `husky` has been used.

But convention commits have some learning curve based into it. So to make it easier for the developers, there is `commitizen` which is an interactive way of making a commit, buy asking a set of questions and then creating a properly formatted commit that can pass the `commit-msg` hook.

More info:

-   [ commit-msg hook ](https://gerrit-review.googlesource.com/Documentation/cmd-hook-commit-msg.html)
-   [ Convention Commits ](https://www.conventionalcommits.org/)
-   [ Commitizen ](https://commitizen-tools.github.io/commitizen/)
-   [ Husky ](https://typicode.github.io/husky/#/)

### Releases

The next part is creating releases, this was achieved using a library called `release-it` which creates a release, creates a change log, updates the version in the `package.json` file, creates tags and pushes the tag and the commit to the remote (it also can publish to npm and create a release on `GitHub` and `Gitlab`. To achieve it just enable it in the config and update values accordingly). You can manually enter or it can automatically (based on commits) detect if the given versions is a `major` `minor` or a `patch` release and updates

To automate this, `bitbucket-pipeline` is used which runs anytime there is a merge on the `develop` branch (`develop` was chosen for convenience , ideally you should use the branch that points to production, like `master` or `main`).

But `release-it` on its own, only creates a new release, if a change log is needed for every release, `release-it` provides a plugin, `@release-it/conventional-changelog`. This plugin allows release it to create a change-log based on the commits that were made. The change-log has information like version name, version-date, types of commits in the version, if mentioned in the commit then all the tickets/issues connected to the commits and also provides links to those commits and issues.

More info:

-   [ release-it ](https://github.com/release-it/release-it)
-   [ bitbucket-pipelines ](https://bitbucket.org/product/features/pipelines)
-   [ @release-it/conventional-changelog ](https://github.com/release-it/conventional-changelog)

## Sounds great? Let's set it up!

The setup is easy, it is divided into different parts so, can if only a part of it interests you, you can take only that part.
_NOTE: Some parts will be dependent on other so make sure the dependencies match up!_

### Husky

Husky can be installed and initialized with one command

```bash
npx husky-init && yarn
```

### Commitizen

First, install the Commitizen CLI tools, you can install globally, since this would be needed to create commits from this point.

```bash
yarn add -D commitizen
```

Next, initialize your project to use the `cz-conventional-changelog` adapter by typing:

```bash
# npm
commitizen init cz-conventional-changelog --save-dev --save-exact
# yarn
commitizen init cz-conventional-changelog --yarn --dev --exact
# pnpm
commitizen init cz-conventional-changelog --pnpm --save-dev --save-exact
```

Now if you want to set up a local script to run `commitizen` without installing it globally, add a script in the `package.json`

```json
	"scripts": {
		...
		"commit": "cz"
	}
```

You can also run it directly using

```bash
yarn cz
# or
npx cz
```

Now you can make conventional commits. Yay!

### Commitllint

To enforce that any user does not use `git commit` to make a non-convention commit, we will use [commitlint](https://commitlint.js.org/#/) to lint our commit messages.
Install commitlint:

```bash
yarn add -D @commitlint/cli @commitlint/config-conventional
```

Configure commitlint, create a file names `commitlint.config.js` and add `module.exports = {extends: ['@commitlint/config-conventional']}` as its content:

```
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

Now, create a `commit-msg` hook using `husky`

```bash
npx husky add .husky/commit-msg 'npx commitlint --edit "$1"'
```

### Release-it

Installing `release-it`:

```bash
yarn add -D release-it
```

You can use release it at this point but configuration will make it more customizable. For this you can create a `.release-it.json` file and add relative config. You can find all the config options [here](https://github.com/release-it/release-it/blob/main/config/release-it.json).
Following have explained few of the options,

```json
    "git": {
		    // the format in which the commit should show
        "commitMessage": "chore(release): 🔖 RC - v${version} [skip ci]",
        // tag name format
        "tagName": "v${version}",
        // tag description, you can get changelog here as well
        "tagAnnotation": "🔖 RC - v${version}"
    },
    "github": {
		    // release it on github
        "release": false
    },
    "npm": {
		    // release it on npm
        "publish": false
    },
```

Now to create a release you can use

```bash
npx release-it # interactive release
# or
npx release-it --ci # non-interactive release
```

Also for ease of use in CI pipeline add it in the `package.json` scripts.

```json
	"scripts": {
		...
		"release": "release-it --ci"
	}
```

#### Change-log plugin

Release-it has a plugin that create a changelog on every release, based on the commits that were made since last release.
To start, first install the plugin

```bash
yarn add -D @release-it/conventional-changelog
```

Now in the release-it config add a `plugins` key

```json
	"plugins": {
	  "@release-it/conventional-changelog": {
	    "preset": "conventionalcommits",
	    // the file which will have the changelog
	    "infile": "CHANGELOG.md"
	  }
}
```

You can check all the available options [here](https://github.com/release-it/conventional-changelog)
Here is a sample configuration:

```json
    "plugins": {
        "@release-it/conventional-changelog": {
            "preset": {
                "name": "conventionalcommits",
                // Types of commits you want to show/hide in CHANGELOG.md and section titles
                "types": [
                    {
                        "type": "feat",
                        "section": "✨ Features"
                    },
                    {
                        "type": "fix",
                        "section": "🐛 Bug Fixes"
                    },
                    {
                        "type": "chore",
                        "hidden": true
                    },
                    {
                        "type": "docs",
                        "section": "📚 Documentation"
                    },
                    {
                        "type": "style",
                        "section": "💎 UI & Styling"
                    },
                    {
                        "type": "refactor",
                        "section": "⚙️  Refactor"
                    },
                    {
                        "type": "perf",
                        "section": "🚀 Performance Improvements"
                    },
                    {
                        "type": "test",
                        "hidden": true
                    }
                ],
                // changed the commit link url, it is differnt for Github/Gitlab/Bitbucket
                "commitUrlFormat": "{{host}}/{{owner}}/{{repository}}/commits/{{hash}}",
                // changed the tag link url, it is differnt for Github/Gitlab/Bitbucket
                "compareUrlFormat": "{{host}}/{{owner}}/{{repository}}/commits/tag/{{currentTag}}",
                // url for tracking software like JIRA, Trello
                "issueUrlFormat": "https://heady.atlassian.net/browse/{{id}}",
                // automatically detect issue based on the following patterns from the commits
                "issuePrefixes": ["#", "KA-", "ka-"]
            },
            // Changelog file
            "infile": "CHANGELOG.md",
            // Changelog file header
            "header": "# Changelog"
        }
    }

```

### Auto-Release on Merge

#### Bitbucket

This is achieve using `bitbucket-pipelines`, here is a sample pipeline code.

```yaml
image: node:lts

pipelines:
    branches:
        develop:
            - step:
                  caches:
                      - node
                  name: Installing dependencies using yarn
                  script:
                      - yarn install
            - step:
                  name: Compiling and Building the application
                  script:
                      - echo "Building the application scripts";
            - step:
                  name: Creating a release
                  caches:
                      - node
                  script:
                      - yarn release
```

#### Github

The following is a github action which tags and releases the code, when `github.release: true` in the `.release-it.json` file.

```yml
name: Release Action
run-name: ${{ github.actor }} has created a new release 🚀

on:
    pull_request:
        types:
            - closed
        branches:
            - 'develop'
permissions:
    contents: write

jobs:
    release:
        if: github.event.pull_request.merged == true
        runs-on: ubuntu-20.04
        steps:
            - name: Check out the repository to the runner
              uses: actions/checkout@v4
            - uses: actions/setup-node@v3
              with:
                  node-version: 18
            - name: Install the dependencies
              run: yarn
            - name: Setting Repository Details
              run: |
                  git config --local user.name 'Priyesh Shrivastava'
                  git config --local user.email 'priyesh.zero@gmail.com'
            - name: Create a release
              run: yarn release
              env:
                  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN}}
```
