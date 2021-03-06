# gitbot

gitbot lets you programmatically make changes to many git repositories.

## Motivation

Clever has a service-oriented architecture where each service is its own git repository.
This lets us quickly develop changes that are limited to a single service.
However, there are some changes (see examples below) that need to be made across many services.
`gitbot` takes the pain out of making changes across many repos.

## Usage

`gitbot` takes in one argument: a path to a config file.
The config file is YAML of the following form:

```yaml
# repos is a list of repositories to examine, e.g. "git@github.com:Clever/gitbot.git"
# the format of each value here must be passable to `git clone`
repos:
  - git@github.com:Clever/aviator.git
# Basepath to prepend temp directories. Not including this option is okay and the program will assume ""
base_path: "path/to/prepend/tmpdir"
# change_cmds describes the programs that will be invoked on each repo.
# a change command must conform to the following rules:
# - it takes in one positional argument: the path to a repo to examine
# - it either
#   (a) makes changes to files within the repo, outputs a commit message to stdout, and exits with code 0
#   (b) exits with a nonzero exit code
change_cmds:
  # command paths can either be absolute paths, or paths relative to the configuration file.
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
    args: ["pull-request", "-m", "Added something trivial", "-b", "Clever:master", "-h", "Clever:add-something-trivial"]
```

## Tips

* Start small: run on a single repository to start.
* Start with a single no-op `post_cmd` and run `gitbot` with `GITBOT_LEAVE_TEMPDIRS=1`.
This lets you examine the side effects of the change command without any consequences.
* Use `git diff HEAD^ HEAD` as a `post_cmd` to see the commit that your change generated.

## Note about using `hub`

[hub](https://github.com/github/hub) is very useful as a `post_cmd`, since it lets you open pull requests.
However, it requires some setup.

To install `hub`, download a tarball from the `hub` [releases](https://github.com/github/hub/releases) page.
After unpacking the tarball, navigate into the unpacked folder and run the `install` script: `sudo ./install`.
This will copy the `hub` binary into `/usr/local/bin/`, man pages into `/usr/local/share`, etc.
You can verify the installation by running `hub -h` or `man hub`.

In order for `hub` to work for private GitHub repositories, you will need to create a file `~/.config/hub` that contains a GitHub oauth token:

```
github.com:
- user: <your username>
  oauth_token: <provision one by visiting https://github.com/settings/applications>
  protocol: https
```

You can verify the setup by attempting to clone a private repository: `hub clone <user>/<private repo>`.


## Install

`gitbot` can be downloaded from the [releases](https://github.com/Clever/gitbot/releases) page.

## Example use cases

- update the version of a dependency to a new version
- run static analysis tools (e.g. linters)
- add a license/contributing.md to many repos
- optimize images
- programmatically change a common configuration file present in many repos


## Vendoring

Please view the [dev-handbook for instructions](https://github.com/Clever/dev-handbook/blob/master/golang/godep.md).
