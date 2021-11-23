![logo](docs/git_spr_logo.png)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build](https://github.com/ejoffe/spr/actions/workflows/ci.yml/badge.svg)](https://github.com/ejoffe/spr/actions/workflows/ci.yml)
[![ReportCard](https://goreportcard.com/badge/github.com/ejoffe/spr)](https://goreportcard.com/report/github.com/ejoffe/spr)
[![Doc](https://godoc.org/github.com/ejoffe/spr?status.svg)](https://godoc.org/github.com/ejoffe/spr)
[![Release](https://img.shields.io/github/release/ejoffe/spr.svg)](https://GitHub.com/ejoffe/spr/releases/) 
[![Join the chat at https://gitter.im/ejoffe-spr/community](https://badges.gitter.im/ejoffe-spr/community.svg)](https://gitter.im/ejoffe-spr/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

![terminal cast](docs/git_spr_cast.gif)
# Stacked Pull Requests on GitHub

Easily manage stacks of pull requests on GitHub. 
**git spr** is a client side tool that achieves a simple streamlined stacked diff workflow using github pull requests and branches. **git spr** manages your pull request stacks for you, so you don't have to. 

With **git spr** each commit becomes a pull request, and each branch becomes a stack of pull requests. This allows for multiple commits to be stacked on top of each other in a single branch, avoiding the overhead of starting a new branch for every new change or feature. Small changes and pull requests are easy and fast to achieve. One doesn't have to worry about stacked branches on top of each other and managing complicated pull request stacks. The end result is a more streamlined faster software development cycle.

Installation 
------------

### Brew
```bash
> brew tap ejoffe/homebrew-tap
> brew install spr
```

### Apt
```bash
> echo "deb [trusted=yes] https://apt.fury.io/inigolabs/ /" > /etc/apt/sources.list.d/inigolabs.list
> sudo apt update 
> sudo apt install spr
```

### Manual
Download the pre-compiled binaries from the [releases page](https://github.com/ejoffe/spr/releases) and copy to your bin path.

Workflow
--------
Commit your changes to a branch as you normally do. Note that every commit will end up becoming a pull request.
```shell
> touch feature_1
> git add feature_1
> git commit -m "Feature 1"
> touch feature_2
> git add feature_2
> git commit -m "Feature 2"
> touch feature_3
> git add feature_3
> git commit -m "Feature 3"
```

The subject of the commit message will be the title of the pull request, and the body of the message will be the body of the pull request.
If you have a work in progress change that you want to commit, but don't want to create a pull request yet, start the commit message with all caps **WIP**. The spr script will not create a pull request for any commit which starts with WIP, when you are ready to create a pull request remove the WIP.
There is no need to create new branches for every change, and you don't have to call git push to get your code to github. Instead just call **git spr update**.

Managing Pull Requests
----------------------
Run **git spr update** to sync your whole commit stack to github and create pull requests for each new commit in the stack. If a commit was amended the pull request will be updated automatically. The command outputs a list of your open pull requests and their status. **git spr update** pushes your commits to github and creates pull requests for you, so you don't need to call git push or open pull requests manually in the UI.

```shell
> git spr update
[·✗✔✗] 60: Feature 3
[✔✔✔✔] 59: Feature 2
[✔✔✔✔] 58: Feature 1
```

Amending Commits
----------------
When you need to update a commit, either to fix tests, update code based on review comments, or just need to change something because you feel like it. You should amend the commit. 
Use **git amend** to easily amend your changes anywhere in the stack. Stage the files you want to amend, and instead of calling git commit, use **git amend** and choose the commit you want to amend when prompted.  
```shell
> touch feature_2
> git add feature_2
> git amend
 3 : 5cba235d : Feature 3
 2 : 4dc2c5b2 : Feature 2
 1 : 9d1b8193 : Feature 1
Commit to amend [1-3]: 2
```

Merge Status Bits
-----------------
Each pull request has four merge status bits signifying the request's ability to be merged. For a request to be merged, all required status bits need to show **✔**. Each status bit has the following meaning:
1. github checks run and pass 
  - · : pending 
  - ✗ : some check failed 
  - ✔ : all checks pass 
  - \- : checks are not required to merge (can be configured in yml config)
2. pull request approval
  - ✗ : pull request hasn't been approved
  - ✔ : pull request is approved
  - \- : approval is not required to merge (can be configured in yml config)
3. merge conflicts
  - ✗ : commit has conflicts that need to be resolved
  - ✔ : commit has no conflicts 
4. stack status
  - ✗ : commit has other pull requests below it that can't merge
  - ✔ : all commits below this one are clear to merge

Pull request approval and checks requirement can be disabled in the config file, see configuration section below for more details.

Merging Pull Requests
---------------------
Your pull requests are stacked. Don't use the UI to merge pull requests, if you do it in the wrong order, you'll end up pushing one pull request into another, which is probably not what you want. Instead just use **git spr merge** and you can merge all the pull requests that are mergeable in one shot. Status for the remaining pull requests will be printed after the merged requests.

```shell
> git spr merge
MERGED #58 Feature 1
MERGED #59 Feature 2
[·✗✔✗] 60: Feature 3
```

Show Current Pull Requests
--------------------------
```shell
> git spr status
[·✗✔✗] 60: Feature 3
```

Configuration
-------------
When the script is run for the first time two config files are created.
Repository configuration is saved to .spr.yml in the repository base directory. 
User specific configuration is saved to .spr.yml in the user home directory.

| Repository Config   | Type | Default    | Description                                                    |
| ------------------- | ---- | ---------- | -------------------------------------------------------------- |
| requireChecks       | bool | true       | require checks to pass in order to merge                       |
| requireApproval     | bool | true       | require pull request approval in order to merge                |
| githubRepoOwner     | str  |            | name of the github owner (fetched from git remote config)      |
| githubRepoName      | str  |            | name of the github repository (fetched from git remote config) |
| githubRemote        | str  | origin     | github remote name to use                                      |
| githubBranch        | str  | master     | github branch for pull request target                          |
| githubHost          | str  | github.com | github host, can be updated for github enterprise usecase      |

| User Config         | Type | Default | Description                                                       |
| ------------------- | ---- | ------- | ----------------------------------------------------------------- |
| showPRLink          | bool | true    | show full pull request http link                                  |
| logGitCommands      | bool | false   | logs all git commands to stdout                                   |
| logGitHubCalls      | bool | false   | logs all github api calls to stdout                               |
| statusBitsHeader    | bool | true    | show status bits type headers                                     |

Happy Coding!
-------------
If you find a bug, feel free to open an issue. Pull requests are welcome.

If you find this script as useful as I do, add a **star** and tell your fellow githubers.

License
-------

- [MIT License](LICENSE)
