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

> job matrix can generate a maximum of 256 jobs per workflow run. This limit also applies to self-hosted runners.

> Each option you define in the matrix has a key and value. The keys you define become properties in the matrix context and you can reference the property in other areas of your workflow file. For example, if you define the key os that contains an array of operating systems, you can use the matrix.os property as the value of the runs-on keyword to create a job for each operating system. For more information, see "Context and expression syntax for GitHub Actions."

> The order that you define a matrix matters. The first option you define will be the first job that runs in your workflow.
