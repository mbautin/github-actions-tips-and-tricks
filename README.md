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
