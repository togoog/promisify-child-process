# promisify-child-process

[![Build Status](https://travis-ci.org/jcoreio/promisify-child-process.svg?branch=master)](https://travis-ci.org/jcoreio/promisify-child-process)
[![Coverage Status](https://codecov.io/gh/jcoreio/promisify-child-process/branch/master/graph/badge.svg)](https://codecov.io/gh/jcoreio/promisify-child-process)
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
[![npm version](https://badge.fury.io/js/promisify-child-process.svg)](https://badge.fury.io/js/promisify-child-process)

seriously like the best async child process library

Based upon [`child-process-async`](https://github.com/itsjustcon/node-child-process-async),
but more thorough, because that package doesn't seem very actively maintained.

`promisify-child-process` provides a **drop-in replacement** for the
original `child_process` functions, not just duplicate methods that
return a `Promise`. So when you call `exec(...)` we still return a
`ChildProcess` instance, just with `.then()` and `.catch()` added to
make it promise-friendly.

## Install and Set-up

```sh
npm install --save promisify-child-process
```

If you are using a old version of Node without build-in `Promise`s or
`Object.create`, you will need to use polyfills.

## Upgrading to v3

You must now pass `maxBuffer` or `encoding` to `spawn`/`fork` if you want to
capture `stdout` or `stderr`.

## Warning: capturing output

`exec` and `execFile` capture `stdout` and `stderr` by default. But `spawn` and
`fork` don't capture `stdout` and `stderr` unless you pass an `encoding` or
`maxBuffer` option:

```js
const { spawn } = require('promisify-child-process');

async function() {
  // captures output
  const { stdout, stderr } = await spawn('ls', [ '-al' ], {encoding: 'utf8'});
  const { stdout, stderr } = await spawn('ls', [ '-al' ], {maxBuffer: 200 * 1024});

  // BUG, DOESN'T CAPTURE OUTPUT:
  const { stdout, stderr } = await spawn('ls', [ '-al' ]);
}
```

## Usage

```js
// OLD:
const { exec, spawn, fork, execFile } = require('child_process')
// NEW:
const { exec, spawn, fork, execFile } = require('promisify-child-process')
```

If for any reason you need to wrap a `ChildProcess` you didn't create,
you can use the exported `promisifyChildProcess` function:

```js
const {promisifyChildProcess} = require('promisify-child-process');

async function() {
  const { stdout, stderr } = await promisifyChildProcess(
    some3rdPartyFunctionThatReturnsChildProcess()
  )
}
```

## Examples

### `exec()`

```js
async function() {
  const { stdout, stderr } = await exec('ls -al');
  // OR:
  const child = exec('ls -al', {});
  // do whatever you want with `child` here - it's a ChildProcess instance just
  // with promise-friendly `.then()` & `.catch()` functions added to it!
  child.stdin.write(...);
  child.stdout.pipe(...);
  child.stderr.on('data', (data) => ...);
  const { stdout, stderr } = await child;
}
```

### `spawn()`

```js
async function() {
  const { stdout, stderr, code } = await spawn('ls', [ '-al' ], {encoding: 'utf8'});
  // OR:
  const child = spawn('ls', [ '-al' ], {});
  // do whatever you want with `child` here - it's a ChildProcess instance just
  // with promise-friendly `.then()` & `.catch()` functions added to it!
  child.stdin.write(...);
  child.stdout.pipe(...);
  child.stderr.on('data', (data) => ...);
  const { stdout, stderr, code } = await child;
}
```
