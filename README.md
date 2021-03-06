# ipfsx

[![Build Status](https://travis-ci.org/ipfs-shipyard/ipfsx.svg?branch=master)](https://travis-ci.org/ipfs-shipyard/ipfsx) [![dependencies Status](https://david-dm.org/ipfs-shipyard/ipfsx/status.svg)](https://david-dm.org/ipfs-shipyard/ipfsx)

> Experimental IPFS API

## Table of Contents

* [Background](#background)
* [Install](#install)
* [Usage](#usage)
* [API](#api)
* [Contribute](#contribute)
* [License](#license)

## Background

JS IPFS supports two types of stream at the API level, but uses pull streams for internals. If I was working on js-ipfs at the time I'd have made the same decision. Since then, async/await became part of the JS language and the majority of JavaScript runtimes now support async/await, async iterators and for/await/of (i.e. no need to transpile). These tools give us the power to stream data without needing to rely on a library.

Just because there are new language features available doesn't mean we should switch to using them. It's a significant upheaval to change the core interface spec and it's implementations (js-ipfs, js-ipfs-api etc.) without good reason. That is why this repository exists: it provides a playground where we can test out new API ideas without having to set them in stone by writing them in the spec.

The big changes are to switch to async/await syntax and to make use of async iterators in place of Node.js/pull streams. I want JS IPFS to feel modern, up to date and cutting edge and I'm willing to bet that this will aid community contributions and adoption.

Part of the reason I'm pro switching to async iterators is because I see parallels between them and pull streams, and I'm super pro pull streams for their simplicity and power:

* Clear story for error propagation and handling
* Backpressure is built in and easy to implement
* No complicated internal state that is difficult to understand

There's actually a bunch of other good reasons to switch to async/await and async iterators:

* Reduction in bundle size - no need to bundle two different stream implementations, and their eco-system helper modules, no need for the `async` module.
* Reduce `npm install` time - fewer dependencies to install.
* Allows us to remove a bunch of plumbing code that converts Node.js streams to pull streams and vice versa.
* Simplifies the API, no `addPullStream`, `addReadableStream`.
* Building an `interface-ipfs-core` compatible interface becomes a whole lot easier, no dual promise/callback API and no multiple stream implementation variations of the same function. It would also reduce the number of tests in the `interface-ipfs-core` test suite for the same reasons.
* [Node.js readable streams are now async iterators](http://2ality.com/2018/04/async-iter-nodejs.html) thanks to [#17755](https://github.com/nodejs/node/pull/17755)!
* Of note, it is trivial to convert from [pull stream to (async) iterator](https://github.com/ipfs-shipyard/pull-stream-to-async-iterator) and [vice versa](https://github.com/ipfs-shipyard/async-iterator-to-pull-stream).
* Unhandled throws that cannot be caught will no longer be a problem
* Better stack traces, stacks no longer clipped at async boundaries, [`await` stack traces better than promise stack traces](https://mathiasbynens.be/notes/async-stack-traces)

Something for your consideration - async/await is inevitable for js-ipfs and js-ipfs-api, the CLI tests are already all promise based, when we inevitably upgrade to Hapi 17 the HTTP API will have to become promise based. The whole of the core interface is dual callback/promise based through `promisify`. Maybe it's time to double down on promises?

Specific rationale for deviations from the `interface-ipfs-core` API is documented in [RATIONALE.md](https://github.com/ipfs-shipyard/ipfsx/blob/master/RATIONALE.md).

## Install

```sh
npm install ipfsx
```

## Usage

```js
import ipfsx from 'ipfsx'
import IPFS from 'ipfs' // N.B. also works with ipfs-api!

const node = await ipfsx(new IPFS)

// IPFS node now ready to use!

// Add something to IPFS
const { cid } = await node.add('hello world').first()

// Stream content from IPFS using async iterators
let data = Buffer.alloc(0)
for await (const chunk of node.cat(cid)) {
  data = Buffer.concat([data, chunk])
}

// for more, see API below
```

## API

* [Getting started](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#getting-started)
* [`add`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#add)
* [`block.get`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#blockget)
* [`block.put`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#blockput)
* [`block.stat`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#blockstat)
* [`cat`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#cat)
* [`cp`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#cp) <sup>(MFS)</sup>
* [`dag.get`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#dagget)
* [`dag.put`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#dagput)
* [`dag.resolve`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#dagresolve)
* [`get`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#get)
* [`id`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#id)
* [`ls`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#ls) <sup>(MFS)</sup>
* [`mkdir`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#mkdir) <sup>(MFS)</sup>
* [`mv`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#mv) <sup>(MFS)</sup>
* [`read`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#read) <sup>(MFS)</sup>
* [`rm`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#rm) <sup>(MFS)</sup>
* [`start`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#start)
* [`stat`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#stat) <sup>(MFS)</sup>
* [`stop`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#stop)
* [`version`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#version)
* [`write`](https://github.com/ipfs-shipyard/ipfsx/blob/master/API.md#write) <sup>(MFS)</sup>
* TODO: more to come in upcoming releases!

## Contribute

Feel free to dive in! [Open an issue](https://github.com/ipfs-shipyard/ipfsx/issues/new) or submit PRs.

## License

[MIT](LICENSE) © Protocol Labs
