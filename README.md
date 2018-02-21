
# Stack

 Minimalist provisioning tool written in Go.

 ![golang provisioning tool stack](https://dl.dropboxusercontent.com/u/6396913/stack/provision.gif)

```sh
# Node
curl -L# http://nodejs.org/dist/v0.10.30/node-v0.10.30-darwin-x64.tar.gz | tar -zx --strip 1 -C /usr/local

# Node version manager
npm install -g n

# Node releases
n 0.8.28
n 0.10.30
n 0.11.13
```

## Installation

 Via go-get:

```
$ go get github.com/samgaw/stack/cmd/stack
```


## Usage

```
  Usage:
    stack [--list] [--no-color] [--verbose] <file>
    stack -h | --help
    stack --version

  Options:
    -C, --no-color   output with color disabled
    -l, --list       output commit status
    -V, --verbose    output command stdio
    -h, --help       output help information
    -v, --version    output version
```

## About

 There are a lot of great provisioning tools out there, but as far as
 I know most of them are part of much larger systems, use unfamiliar DSLs,
 or rely on the presence of an interpreter for scripting languages such as Ruby or Python.

 I'm not suggesting this tool is better than any existing solution but I really wanted
 something that looked and behaved like a regular shell script. Also since it's written in Go it's very simple to curl the binary on to any existing system.

 The choice of using shell commands makes this tool less declarative than
 some of the alternatives, however I think it's a good fit for the goal
 of being a minimalistic solution.

## How it works

 Stack behaves like shell scripts with `set -e`, as it will exit on failure. Unlike
 shell scripts a commit log is used in order to prevent re-execution of previous commands.

 The log is held at ~/.provision.log (by default), this file keeps
 track of commands which have already completed. Once a command is run
 and successfully exits, it is considered complete, at which time the
 SHA1 of the command is written to this file. Subsequent runs will see
 the SHA and ignore the command.

 The commit log is shared between any number of provision files, this
 means the same command run in a different provisioning script will
 no-op if it has already been successfully run.

 If a command line is modified it will result in a different hash,
 thus it will be re-run.

 This gif illustrates how exiting after the initial "commit" will cause it to be ignored
 the second time around:

 ![stack commits](https://dl.dropboxusercontent.com/u/6396913/stack/provision-commits.gif)

## Syntax

 The syntax has two flavours, the shell-like syntax, and the canonical version which pkg/provisioner consumes. For example here is the shell version of a small node.js provisioning script:

```sh
# Node
curl -L# http://nodejs.org/dist/v0.10.30/node-v0.10.30-darwin-x64.tar.gz | tar -zx --strip 1 -C /usr/local

# Node version manager
npm install -g n

# Node releases
n 0.8.28
n 0.10.30
n 0.11.13
```

 Here's the same script after it's rewritten to the canonical syntax:

```sh

LOG Node
RUN curl -L# http://nodejs.org/dist/v0.10.30/node-v0.10.30-darwin-x64.tar.gz | tar -zx --strip 1 -C /usr/local

LOG Node version manager
RUN npm install -g n

LOG Node releases
RUN n 0.8.28
RUN n 0.10.30
RUN n 0.11.13
```

## Commands

 Currently only a few commands are supported, however more
 may be added in the future to simplify common processes,
 provide concurrency, and so on.

 Open an issue if there's something you'd like to see!

### RUN <command>

  `RUN <command>` executes a command through `/bin/sh`, so shell
  features such as globbing, brace expansion and pipelines will
  work as expected.

  If the `<command>` exits > 0 then commit is a failure and will
  not be written to the log.

  Lines without a command are considered to be `RUN` lines.

### LOG <message>

  `LOG <message>` simply outputs a log message to stdio.

  Aliased as `#`.

### INCLUDE <path>

  `INCLUDE <path>` reads the file at `<path>`, rewrites it
  and injects it into the location of this command in the
  pre-processing step.

  The include `<path>` is relative to the CWD.

  Aliased as `.` and `source`.

## Options

### --verbose

  By default output is suppressed, however `--verbose` will stream std{err,out}:

  ![stack provisioning verbose](https://dl.dropboxusercontent.com/u/6396913/stack/provision-verbose.gif)

## Running tests

 All tests:

```
$ make test
```

 Individual tests:

```
$ cd pkg/rewriter
$ go test
```

# License

 MIT
