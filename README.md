
# `slp-realtime`

[![npm version](https://img.shields.io/npm/v/@vinceau/slp-realtime.svg?style=flat)](https://npmjs.org/package/@vinceau/slp-realtime "View this project on npm")
[![Build Status](https://github.com/vinceau/slp-realtime/workflows/build/badge.svg)](https://github.com/vinceau/slp-realtime/actions?workflow=build)
[![Coverage Status](https://coveralls.io/repos/github/vinceau/slp-realtime/badge.svg?branch=feat/coverage)](https://coveralls.io/github/vinceau/slp-realtime?branch=feat/coverage)
[![License](https://img.shields.io/npm/l/@vinceau/slp-realtime)](https://github.com/vinceau/slp-realtime/blob/master/LICENSE)

> A real-time Slippi parsing library.

This library provides an easy way to subscribe to real-time [Slippi](https://github.com/project-slippi/project-slippi) game events as they happen.


## Highlights

* Go file-less. Read directly from the relay or console.
* Custom combos. Easily add combo parameters and output Dolphin-compatible JSON files.
* Events, Promise, and Stream API.

## Installation

**With NPM**

```bash
npm install @vinceau/slp-realtime
```

**With Yarn**

```bash
yarn add @vinceau/slp-realtime
```

## Usage

See a [working example](examples) or [check out the docs](https://vince.id.au/slp-realtime/).

### Subscribing to In-Game Events

We can use this library to subscribe to in game events.

First instantiate an instance of `SlpLiveStream` and connect to a Wii or Slippi relay.

```javascript
const { SlpLiveStream } = require("@vinceau/slp-realtime");

const livestream = new SlpLiveStream();
livestream.start(address, slpPort)
  .then(() => {
    console.log("Successfully connected!");
  })
  .catch(console.error);
```

Then instantiate an instance of `SlpRealTime` and pass the `SlpLiveStream` to it.
We will use it to subscribe to desired events. For example:

```javascript
const { SlpRealTime } = require("@vinceau/slp-realtime");

const realtime = new SlpRealTime();
// Read from the SlpLiveStream object from before
realtime.setStream(livestream);

realtime.on("gameStart", () => {
    console.log("game started");
});

realtime.on("percentChange", (index, percent) => {
    console.log(`player ${index}'s new percent: ${percent}`);
});

realtime.on("spawn", (index, stock) => {
    console.log(`player ${index} spawned with ${stock.count} stocks remaining`);
});

realtime.on("death", (i) => {
    console.log(`player ${i} died`);
});

realtime.on("comboEnd", () => {
    console.log("a combo just happened");
});
```

### Detecting Custom Combos

We can already subscribe to the `comboEnd` event but that gets emitted for *every* combo. We need a way to filter out for specific combos.

First, instantiate a `ComboFilter`. For all the possible filtering options, see [ComboFilterSettings](api/README.md#combofiltersettings).

```javascript
const { ComboFilter } = require("@vinceau/slp-realtime");

const comboFilter = new ComboFilter();
comboFilter.updateSettings({
    excludeCPUs: false,    // combos on CPUs are okay
    comboMustKill: false,  // combos don't have to kill
    minComboPercent: 40,   // combos have to do at least 40% damage
});
```

`ComboFilter` exposes a handy `isCombo()` method which returns `true` if a given combo matches the specified criteria. We can hook it up to our live stream with the following:

```javascript
realtime.on("comboEnd", (combo, settings) => {
    if (!comboFilter.isCombo(combo, settings)) {
        console.log("Combo did not match criteria!");
        return;
    }

    console.log("Combo matched!");
});
```

## Development

To build the library from source:

```bash
yarn run build
```

To start the development server:

```bash
yarn run watch
```

To run the tests:

```bash
yarn run test
```

## Acknowledgements

This project was made possible by:

* [Jas Laferriere](https://github.com/JLaferri) and the rest of the [Project Slippi](https://github.com/project-slippi) team

* [NikhilNarayana](https://github.com/NikhilNarayana) and his [Get Slippi Combos](https://gist.github.com/NikhilNarayana/d45e328e9ea47127634f2faf575e8dcf) script


## License

This software is released under the terms of [MIT license](LICENSE).
