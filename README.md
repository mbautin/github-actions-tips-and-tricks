# github-actions-tips-and-tricks
Various notes related to using GitHub Actions

## Creating a matrix with multiple OSes

From https://github.community/t5/GitHub-Actions/Create-matrix-with-multiple-OS-and-env-for-each-one/td-p/38339

Example:
```
name: test matrix 
on: push
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [macos-latest, ubuntu-latest, ubuntu-latest]
        include: 
        - os: macos-latest
          TARGET: x86_64-apple-darwin
          COMPILER: clang
          LINKER: clang       

        - os: ubuntu-latest
          TARGET: armv7-unknown-linux-musleabihf
          COMPILER: arm-linux-gnueabihf-gcc-5
          LINKER: gcc-5-arm-linux-gnueabihf       

        - os: ubuntu-latest
          TARGET: x86_64-unknown-linux-musl
          COMPILER: gcc
          LINKER: gcc
    steps:
     - uses: actions/checkout@v1
     - run: echo ${{matrix.TARGET}}
```

More on creating a build matrix (from https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions):

> `jobs.<job_id>.strategy.matrix`
> You can define a matrix of different job configurations. A matrix allows you to create multiple jobs by performing variable substitution in a single job definition. For example, you can use a matrix to create jobs for more than one supported version of a programming language, operating system, or tool. A matrix reuses the job's configuration and creates a job for each matrix you configure.
>
> job matrix can generate a maximum of 256 jobs per workflow run. This limit also applies to self-hosted runners.
>
> Each option you define in the matrix has a key and value. The keys you define become properties in the matrix context and you can reference the property in other areas of your workflow file. For example, if you define the key os that contains an array of operating systems, you can use the `matrix.os` property as the value of the runs-on keyword to create a job for each operating system. For more information, see [Context and expression syntax for GitHub Actions](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions).
>
> The order that you define a matrix matters. The first option you define will be the first job that runs in your workflow.

In the example above, the `os` array ends up being the set of names of jobs, which is inconvenient (two jobs have exactly the same name). Instead of that, I found that this setup works:

```
jobs:
  build:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        name:
          - macOS
          - "CentOS 7"
          - "Ubuntu 18.04"
        include:
          - name: "macOS"
            os: macos-latest
            docker_image: none

          - name: "CentOS 7"
            os: ubuntu-latest
            docker_image: yugabyteci/yb_build_infra_centos7:v2020-03-24T08_20_00

          - name: "Ubuntu 18.04"
            os: ubuntu-latest
            docker_image: yugabyteci/yb_build_infra_ubuntu1804:v2020-03-24T08_20_00
```
(used in https://github.com/yugabyte/yugabyte-bash-common/blob/master/.github/workflows/build.yml). Here, we are giving the parallel jobs human-readable names `macOS`, `CentOS 7` and `Ubuntu 18.04`. We have to reference these job names in the items of the `include` list, otherwise something strange happens: the same element of the `include` list could be used in multiple jobs, even with different names. In this example, we are running one job using macOS and two jobs using the default Ubuntu image as the host OS and specify different Docker images inside of those jobs. I suspect we could have used a different key instead of `name` and it might work just as well.

The end result looks like this:

![image](https://user-images.githubusercontent.com/552936/77607025-5b2db400-6ed6-11ea-8905-affe9299b496.png)

## Running tests on pull requests

Relevant discussion: https://github.community/t5/GitHub-Actions/Run-a-GitHub-action-on-pull-request-for-PR-opened-from-a-forked/td-p/15426

https://help.github.com/en/actions/reference/events-that-trigger-workflows#pull-request-events-for-forked-repositories

> Pull request events for forked repositories
> Note: Workflows do not run on private base repositories when you open a pull request from a forked repository.
>
> When you create a pull request from a forked repository to the base repository, GitHub sends the pull_request event to the base repository and no pull request events occur on the forked repository.
>
> Workflows don't run on forked repositories by default. You must enable GitHub Actions in the Actions tab of the forked repository.
>
> The permissions for the GITHUB_TOKEN in forked repositories is read-only. For more information, see "Authenticating with the GITHUB_TOKEN."

## Collapsing boring parts of the log

https://github.community/t5/GitHub-Actions/Has-github-action-somthing-like-travis-fold/td-p/37715
