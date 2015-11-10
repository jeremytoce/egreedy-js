# egreedy

[![Build Status](https://travis-ci.org/kurttheviking/egreedy.svg)](https://travis-ci.org/kurttheviking/egreedy)

**A Promises/A+, [multi-armed bandit optimization](http://en.wikipedia.org/wiki/Multi-armed_bandit) implemented with an epsilon-greedy algorithm.**

This implemention is based on [<em>Bandit Algorithms for Website Optimization</em>](http://shop.oreilly.com/product/0636920027393.do) and related empirical research in ["Algorithms for the multi-armed bandit problem"](https://d2w9gswcdc2jtf.cloudfront.net/research/Algorithms+for+the+multi-armed+bandit+problem.pdf).


## Quick start

First, install this module in your project:

```sh
npm install egreedy --save
```

Then, use the algorithm:

1. Create a bandit with 3 arms and epsilon 0.25:

    ```js
    var Bandit = require('egreedy');

    var bandit = new Bandit({
      arms: 3,
      epsilon: 0.25
    });
    ```

2. Select an arm (for exploration or exploitation, according to the algorithm):

    ```js
    bandit.select().then(function (arm) {
      console.log('pulled arm=' + arm);
    });
    ```

3. Report the reward earned from a chosen arm:

    ```js
    bandit.reward(armId, value).then(function (n) {
      console.log('total observations=' + n);
    });
    ```


## API

This module conforms to the [Bandit Lab 1.0 specification](https://github.com/banditlab/spec-js/blob/master/README.md).

#### `Bandit([config])`

This algorithm defaults to 2 arms and epsilon 0.5:

```js
var bandit = new Bandit();
assert.equal(bandit.arms, 3);
assert.equal(bandit.epsilon, 0.5);
```

The constructor also accepts a `config` object that supports two parameters:

- `arms`: integer, the number of arms over which the optimization will operate
- `epsilon`: float, from 0 (never explore/always exploit) to 1 (always explore/never exploit)

```js
var bandit = new Bandit({
  arms: 4,
  epsilon: 0.75
});
assert.equal(bandit.arms, 4);
assert.equal(bandit.epsilon, 0.75);
```

#### `Bandit#select()`

Choose an arm to play, according to the specified bandit algorithm.

**Arguments**

_None_

**Returns**

A promise that resolves to a Number corresponding to the associated arm index.

**Example**

```js
> var Bandit = require('egreedy');
> var bandit = new Bandit();
> bandit.select().then(function (arm) { console.log(arm); });

0
```

#### `Bandit#reward(arm, reward)`

Inform the algorithm about the payoff from a given arm.

**Arguments**

- `arm` (Integer): the arm index (provided from `bandit.select()`)
- `reward` (Number): the observed reward value (which can be 0, to indicate no reward)

**Returns**

A promise that resolves to a Number representing the count of observed rounds.

**Example**

```js
> var Bandit = require('egreedy');
> var bandit = new Bandit();
> bandit.reward(0, 1).then(function (n) { console.log(n); });

1
```

#### `bandit.serialize()`

Obtain a plain object representing the internal state of the algorithm.

**Arguments**

_None_

**Returns**

A promise that resolves to an Object representing parameters required to reconstruct algorithm state.

**Example**

```js
> var Bandit = require('egreedy');
> var bandit = new Bandit();
> bandit.serialize().then(function (state) { console.log(state); });

{
  arms: 2,
  epsilon: 0.5,
  counts: [0, 0],
  values: [0, 0]
}
```

#### `bandit.load(state)`

Restore an instance of a bandit to a previously serialized algorithm state. This method overrides any options parameters passed at instantiation.

**Arguments**

- `state` (Object): a serialized algorithm state (provided from `bandit.serialize()`)

**Returns**

A promise that resolves to a Number representing the count of observed rounds.

**Example**

```js
> var state = {arms: 2, epsilon: 0.5, counts: [1, 2], values: [1, 0.5]};
> var Bandit = require('egreedy');
> var bandit = new Bandit();
> bandit.load(state).then(function (n) { console.log(n); });

3
```


## Tests

To run the full unit test suite

```
npm test
```

Tests against stochastic methods (e.g. `bandit.select()`) are inherently tricky to test with deterministic assertions. The approach here is to iterate across a semi-random set of conditions to verify that each run produces valid output. So, strictly speaking, each call to `npm test` is executing a slightly different test suite. At some point, the test suite may be expanded to include a more robust test of the distribution's properties &ndash; though because of the number of runs required, would be triggered with an optional flag.


## Contribute

PRs are welcome! For bugs, please include a failing test which passes when your PR is applied.


## Caveat emptor

Currently, this implementation relies on the [native Math.random()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random) which uses a seeded "random" number generator. In addition, the underlying calculations often encounter extended floating point numbers. Arm selection is therefore subject to JavaScript's floating point precision limitations. For general information about floating point issues see the [floating point guide](http://floating-point-gui.de/).

While these factors generally do not impede commercial application, I would consider the implementation suspect in any academic setting.
