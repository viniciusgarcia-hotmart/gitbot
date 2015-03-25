# gitbot

gitbot lets you programmatically make changes to many git repositories.

## Motivation

Clever has a service-oriented architecture where each service is its own git repository.
This lets us develop quickly on changes that are local to a specific service.
However, there are some changes (see examples below) that we'd like to make across many services.
`gitbot` takes the pain out of making changes across many repos.

## Usage

`gitbot` takes in one argument: a path to a config file.
The config file is YAML of the following form:

```
# repos is a list of repositories to examine, e.g. "git@github.com:Clever/gitbot.git"
# the format of each value here must be passable to `git clone`
repos:
  - git@github.com:Clever/aviator.git
# change_cmd describes the program that will be invoked on each repo.
# a change command must conform to the following rules:
# - it takes in one positional argument: the path to a repo to examine
# - it either
#   (a) make changes to files within the repo, output a commit message to stdout, and exit with code 0
#   (b) exit with a nonzero exit code
change_cmd:
  - path: "/path/to/the/program"
    args: ["-a", "flag"]
# post_cmds is a list of programs to run on each repo if changes have been made.
# use post_cmds to do things like pushing branches to github, opening PRs, etc.
# post_cmds are run within the directory where a repository has been cloned.
# post_cmds are only run if the change command makes a change.
# post_cmds can assume that the change has been committed to HEAD.
# post_cmds are run with the same environment variables as `gitbot` itself.
post_cmds:
  - path: "git"
    args: ["push", "origin", "HEAD:add-something-trivial"]
  - path: "hub"
    args: ["pull-request", "-m", "Implemented feature X", "-b", "Clever:master", "-h", "Clever:add-something-trivial"]
```

## Note about using `hub`

[hub](https://github.com/github/hub) is very useful as a `post_cmd`.
However, it requires some setup.
Specifically you will need to create a file `~/.config/hub` that contains an oauth token:

```
github.com:
- user: <your username>
  oauth_token: <provision one by visiting https://github.com/settings/applications>
  protocol: https
```

## Install

(TODO) `gitbot` can be downloaded from the [releases]() page.

## Example use cases

- update the version of a dependency to a new version
- run gofmt/pylint on all *.go/*.py files
- add a license/contributing.md to many repos
- optimize images
- programmatically change a common configuration file present in many repos
