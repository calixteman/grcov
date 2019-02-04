# grcov

[![Build Status](https://travis-ci.org/mozilla/grcov.svg?branch=master)](https://travis-ci.org/mozilla/grcov)
[![Build status](https://ci.appveyor.com/api/projects/status/1957u00h26alxey2/branch/master?svg=true)](https://ci.appveyor.com/project/marco-c/grcov)
[![codecov](https://codecov.io/gh/mozilla/grcov/branch/master/graph/badge.svg)](https://codecov.io/gh/mozilla/grcov)

grcov collects and aggregates code coverage information for multiple source files.

This is a project initiated by Mozilla to gather code coverage results on Firefox.

## Usage

1. Download grcov from https://github.com/mozilla/grcov/releases or run ```cargo install grcov```
2. Run grcov:

```
Usage: grcov DIRECTORY_OR_ZIP_FILE[...] [-t OUTPUT_TYPE] [-s SOURCE_ROOT] [-p PREFIX_PATH] [--token COVERALLS_REPO_TOKEN] [--commit-sha COVERALLS_COMMIT_SHA] [--keep-global-includes] [--ignore-not-existing] [--ignore-dir DIRECTORY] [--llvm] [--path-mapping PATH_MAPPING_FILE] [--branch] [--filter] [--add-prefix ADDED_PREFIX_PATH]
You can specify one or more directories, separated by a space.
OUTPUT_TYPE can be one of:
 - (DEFAULT) lcov for the lcov INFO format;
 - coveralls for the Coveralls specific format.
 - coveralls+ for the Coveralls specific format with function information.
 - ade for the ActiveData-ETL specific format;
 - files to only return a list of files.
SOURCE_ROOT is the root directory of the source files.
PREFIX_PATH is a prefix to remove from the paths (e.g. if grcov is run on a different machine than the one that generated the code coverage information).
ADDED_PREFIX_PATH is a prefix to add to the paths.
COVERALLS_REPO_TOKEN is the repository token from Coveralls, required for the 'coveralls' and 'coveralls+' format.
COVERALLS_COMMIT_SHA is the SHA of the commit used to generate the code coverage data.
By default global includes are ignored. Use --keep-global-includes to keep them.
By default source files that can't be found on the disk are not ignored. Use --ignore-not-existing to ignore them.
The --llvm option can be used when the code coverage information is exclusively coming from a llvm build, to speed-up parsing.
The --ignore-dir option can be used to ignore files/directories specified as globs.
The --branch option enables parsing branch coverage information.
The --filter option allows filtering out covered/uncovered files. Use 'covered' to only return covered files, 'uncovered' to only return uncovered files.
```

Let's see a few examples, assuming the source directory is `~/Documents/mozilla-central` and the build directory is `~/Documents/mozilla-central/build`.

### LCOV output

```sh
grcov ~/Documents/mozilla-central/build -t lcov > lcov.info
```

As the LCOV output is compatible with `lcov`, `genhtml` can be used to generate a HTML summary of the code coverage:
```sh
genhtml -o report/ --show-details --highlight --ignore-errors source --legend lcov.info
```

### Coveralls/Codecov output

```sh
grcov ~/Documents/FD/mozilla-central/build -t coveralls -s ~/Documents/FD/mozilla-central --token YOUR_COVERALLS_TOKEN > coveralls.json
```

### GRCOV with Travis

Here is an example of .travis.yml file
```YAML
sudo: false
language: rust

before_install:
  - curl -L https://github.com/mozilla/grcov/releases/download/v0.4.1/grcov-linux-x86_64.tar.bz2 | tar jxf -

matrix:
  include:
    - os: linux
      rust: nightly

script:
    - export CARGO_INCREMENTAL=0
    - export RUSTFLAGS="-Zprofile -Ccodegen-units=1"
    - cargo build --verbose $CARGO_OPTIONS
    - cargo test --verbose $CARGO_OPTIONS
    - |
      zip -0 ccov.zip `find . \( -name "YOUR_PROJECT_NAME*.gc*" \) -print`;
      ./grcov ccov.zip -s . -t lcov --llvm --branch --ignore-not-existing --ignore-dir "/*" > lcov.info;
      bash <(curl -s https://codecov.io/bash) -f lcov.info;
```

## Build & Test

In order to build, either LLVM 7 or LLVM 8 libraries and headers are required. If one of these versions is sucessfully installed, build with:

```
cargo build
```

To run tests:
```
cargo test
```

## Minimum requirements

- GCC 4.9 or higher is required (if parsing coverage artifacts generated by GCC).

## License

Published under the MPL 2.0 license.
