# Function helpers for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2021.

* Formal specification: Not yet
* Babel plugin: Not yet
* [Proposal history][HISTORY.md]

[HISTORY.md]: https://github.com/js-choi/proposal-function-helpers/blob/main/HISTORY.md

Several useful, common helper functions are defined, downloaded, and used a lot.
We should standardize at least some of them.
This proposal is seeking Committee consensus for Stage 1:
that standardizing at least some Function helpers is “worth investigating”.
It is not seeking to standardize every imaginable helper function:
just a selected few frequently used functions.
Choosing which functions to standardize would be bikeshedding for Stage 2.
Alternatively, the Committee could request that this proposal
be split up into multiple proposals.

These convenience functions are simple,
and they can be reimplemented easily in userspace.
So why standardize them? Because:

1. These helper functions are commonly used and universally useful.
   Each function is frequently downloaded from NPM,
   despite their being easily reimplementable.
   This is to be expected:
   after all, every JavaScript developer needs to manipulate callbacks,
   but they often do not wish to write the utilities themselves.
2. Standardization would improve developer ergonomics.
   If we find ourselves needing these functions in a REPL or script,
   instead of having to download an external package
   or pasting in a definition into our own code,
   we can simply destructure the `Function` object.
3. Standardization would improve code clarity.
   There would be one standard name for each of these functions,
   rather than various names from various libraries
   that refer to the same thing.

Unlike new syntax, standardized helper functions are relatively lightweight ways
to improve the experience of all developers.
These helper functions are well-trodden cowpaths,
each of which deserves consideration for standardization.

The following functions are only possibilities.
Choosing which functions to standardize would be bikeshedding for Stage 2.

## Function.flow
The `Function.flow` static method creates a new function by combining several callbacks.

```js
Function.flow(...fns);

const { flow } = Function;

const f = flow(f0, f1, f2);
f(5, 7); // f2(f1(f0(5, 7))).

const g = flow(g0);
g(5, 7); // g0(5, 7).

const h = flow();
h(5, 7); // 5.
```

The following real-world examples originally used [lodash.flow][].

```js
// From gatsby@3.14.3/packages/gatsby-plugin-sharp/src/plugin-options.js
flow(
  mapUserLinkHeaders(pluginData),
  applySecurityHeaders(pluginOptions),
  applyCachingHeaders(pluginData, pluginOptions),
  mapUserLinkAllPageHeaders(pluginData, pluginOptions),
  applyLinkHeaders(pluginData, pluginOptions),
  applyTransfromHeaders(pluginOptions),
  saveHeaders(pluginData)
)

// From strapi@3.6.8
// packages/strapi-admin/services/permission/permissions-manager/query-builers.js
const transform = flow(flattenDeep, cleanupUnwantedProperties);

// From semantic-ui-react@v2.0.4/docs/static/utils/getInfoForSeeTags.js
const getInfoForSeeTags = flow(
  _.get('docblock.tags'),
  _.filter((tag) => tag.title === 'see'),
  _.map((tag) => {
  }),
)
```

Any function created by `Function.flow`
applies its own arguments to its leftmost callback.
Then that result is applied to its next callback.
In other words, function composition occurs from left to right.

The leftmost callback may have any arity,
but any subsequent callbacks are expected to be unary.

If `Function.flow` receives no arguments, then, by default,
it will return `Function.identity` (which is defined later in this proposal).

Precedents include:
* [lodash][]: [lodash.flow][] is individually downloaded from NPM
  about [600,000 times weekly][lodash.flow]
* [fp-ts][]: `import { flow } from 'fp-ts/function';`
* [Ramda][]: `import { pipe } from 'ramda/src/pipe';`
* [RxJS][]: `import { pipe } from 'rxjs';`

[lodash.flow]: https://www.npmjs.com/package/lodash.flow

## Function.flowAsync
The `Function.flowAsync` static method creates a new function
by combining several potentially async callbacks;
the created function will always return a promise.

```js
Function.flowAsync(...fns);

const { flowAsync } = Function;

// (...args) => Promise.resolve(x).then(f0).then(f1).then(f2).
flowAsync(f0, f1, f2);

const f = flowAsync(f0, f1, f2);
await f(5, 7); // await f2(await f1(await f0(5, 7))).

const g = flowAsync(g0);
await g(5, 7); // await g0(5, 7).

const h = flowAsync();
await h(5, 7); // await 5.
```

Any function created by `Function.flowAsync`
applies its own arguments to its leftmost callback.
Then that result is `await`ed before being applied to its next callback.
In other words, async function composition occurs from left to right.

The leftmost callback may have any arity,
but any subsequent callbacks are expected to be unary.

If `Function.flowAsync` receives no arguments, then, by default,
it will return `Promise.resolve`.

The name “flow” comes from lodash.flow.
(The name compose would be confusing with other languages’ RTL function composition.)

## Function.pipe
The `Function.pipe` static method applies a sequence
of callbacks to a given input value, returning the final callback’s result.

```js
Function.pipe(input, ...fns);

const { pipe } = Function;

// f2(f1(f0(5))).
pipe(5, f0, f1, f2);

// 5.
pipe(5);

// undefined.
pipe();
```

The following real-world examples originally used [fp-ts][]’s `pipe` function.

```js
// From @gripeless/pico@1.0.1/source/inline.ts
return pipe(
  download(absoluteURL),
  mapRej(downloadErrorToDetailedError),
  chainFluture(responseToBlob),
  chainFluture(blobToDataURL),
  mapFluture(dataURL => `url(${dataURL})`)
)

// From StoplightIO Prism v4.5.0 packages/http/src/validator/validators/body.ts
return pipe(
  specs,
  A.findFirst(spec => !!typeIs(mediaType, [spec.mediaType])),
  O.alt(() => A.head(specs)),
  O.map(content => ({ mediaType, content }))
);
```

The first callback is applied to `input`,
then the second callback is applied to the first callback’s result,
and so forth.
In other words, function piping occurs from left to right.

Each callback is expected to be a unary function.

If `Function.pipe` receives only one argument, then it will return `input` by default.\
If `Function.pipe` receives no arguments, then it will return `undefined`.

Precedents include:
* [fp-ts][]: `import { pipe } from 'fp-ts/function';`

***

<details>
<summary>What happened to the F# pipe operator?</summary>

***

F#, Haskell, and other languages that are based on auto-curried unary functions
have a tacit-unary-function-application operator.
The pipe champion group has presented F# pipes for Stage 2 twice to TC39,
being unsuccessful both times
due to pushback from multiple other TC39 representatives’
memory performance concerns, syntax concerns about await,
and concerns about encouraging ecosystem bifurcation/forking.
(For more information, see the [pipe proposal’s HISTORY.md][pipe history].)

Given this reality, TC39 is much more likely to pass
a `Function.pipe` helper function than a similar syntactic operator.

Standardizing a helper function does not preclude
standardizing an equivalent operator later.
For example, TC39 standardized binary `**` even when `Math.pow` existed.

In the future, we might try to propose a F# pipe operator,
but we would like to try proposing `Function.pipe` first,
in an effort to bring its benefits to the wider JavaScript community
as soon as possible.

</details>

## Function.pipeAsync
The `Function.pipeAsync` static method applies a sequence
of potentially async callbacks to a given input value, returning a promise.
The promise will resolve to the final callback’s result.

```js
Function.pipeAsync(input, ...fns);

const { pipeAsync } = Function;

// Promise.resolve(5).then(f0).then(f1).then(f2).
pipeAsync(5, f0, f1, f2);

// Promise.resolve(5).
pipeAsync(5);

// Promise.resolve(undefined).
pipeAsync();
```

The input is first `await`ed.
Then the first callback is applied to `input` and then `await`ed,
then the second callback is applied to the first callback’s result then `await`ed,
and so forth.
In other words, function piping occurs from left to right.

Each callback is expected to be a unary function.

If any callback returns a promise that then rejects with an error,
then the promise returned by `Function.pipeAsync` will reject with the same error.

If `Function.pipeAsync` receives only one argument,
then it will return `Promise.resolve(input)` by default.\
If `Function.pipeAsync` receives no arguments,
then it will return `Promise.resolve(undefined)`.

## Function.constant
The `Function.constant` static method creates a new function from a constant value.
The new function will always return that value, no matter what arguments it is given.

```js
Function.constant(value);

const { constant } = Function;

const f = constant(5);
f(11, 0, 3); // 5.

const g = constant();
g(11, 0, 3); // undefined.
```

The following real-world examples originally used [lodash.constant][].

```js
// From cypress@8.6.0/packages/net-stubbing/lib/server/util.ts
setDefaultHeader('access-control-expose-headers', constant('*'))

// From cypress@8.6.0/packages/driver/src/cypress/utils.ts
return [fn, constant(type)]

// From Odoo v15.0 addons/pad/static/src/js/pad.js
url.toJSON = constant(this.url);

// From ng-table@3.0.1/test/specs/settings.spec.ts
const newSettings: = {
  filterOptions: _.mapValues(allSettings.filterOptions, constant(undefined)),
  dataOptions: _.mapValues(allSettings.dataOptions, constant(undefined)),
  groupOptions: _.mapValues(allSettings.groupOptions, constant(undefined))
};

// From Elastic Kibana v7.15.1
// src/plugins/vis_types/vislib/public/fixtures/mock_data/histogram/_slices.js
{
  name: 0,
  size: 378611,
  aggConfig: {
    type: 'histogram',
    schema: 'segment',
    fieldFormatter: constant(String),
    params: {
      interval: 1000,
      extended_bounds: { /* … */ },
    },
  },
  /* … */
},

// From Yhat Rodeo v2.5.2 src/node/services/files.test.js
fs.lstat.onCall(0).yields(null, {isDirectory: constant(true)});
```

Precedents include:
* [lodash][]: [lodash.constant][] is individually downloaded from NPM
  about [81,000 times weekly][lodash.constant]
* [stdlib][]: `import constantFunction from '@stdlib/utils-constant-function';`
* [fp-ts][]: `import { constant } from 'fp-ts/function';`
* [Ramda][]: `import { always } from 'ramda/src/always';`

[lodash.constant]: https://www.npmjs.com/package/lodash.constant

## Function.identity
The `Function.identity` static method always returns its first argument.

```js
Function.identity(value);

const { identity } = Function;

identity(5); // 5.
identity(); // undefined.
```

The following real-world examples originally used [lodash.identity][].

```js
// From cypress@8.6.0/packages/driver/src/cypress/runner.ts
// “iterates over a suite's tests (including nested suites)
// and will return as soon as the callback is true”
const findTestInSuite = (suite, fn = identity) => {
  for (const test of suite.tests) {
    if (fn(test)) {
      return test
    } /* … */
  }
}

// From gatsby@3.14.3/packages/gatsby-plugin-sharp/src/plugin-options.js
// “get all non falsey values”
return _.pickBy(options, identity)

// From gatsby@3.14.3/packages/gatsby-plugin-gatsby-cloud/src/constants.js
export const DEFAULT_OPTIONS = {
  // “optional transform for manipulating headers for sorting, etc”
  transformHeaders: identity,
  /* … */
}

// From ghost@4.19.0/core/frontend/helpers/img_url.js
// “CASE: only make paths relative if we didn't get a request for an absolute url”
const maybeEnsureRelativePath = !absoluteUrlRequested ? ensureRelativePath : _.identity;

// From Meteor v2.5.0 tools/cordova/builder.js
const boilerplate = new Boilerplate(CORDOVA_ARCH, manifest, {
  urlMapper: identity,
  /* … */
});
```

Precedents include:
* [lodash][]: [lodash.identity][] is individually downloaded from NPM
  about [700,000 times weekly][lodash.identity]
* [stdlib][]: `import identity from '@stdlib/utils-identity-function';`
* [fp-ts][]: `import { identity } from 'fp-ts/function';`
* [Ramda][]: `import { identity } from 'ramda/src/identity';`

[lodash.identity]: https://www.npmjs.com/package/lodash.identity

## Function.noop

The `Function.noop` static method always returns undefined.
`Function.noop` is equivalent to `() => {}`.
It is also equivalent to `constant()`.

This function is already available and frequently used from both [jQuery][] and Lodash, generally to fill a required callback argument or to disable a callback property.

```js
const { noop } = Function;
[ 0, 1 ].map(noop)
// [ undefined, undefined ]
```

The following real-world examples originally used
jQuery’s [`$.noop`][] or [lodash.noop][].

```js
// From Wordpress v5.1.11:
{ /* … */
  defaultExpandedArguments: {
    duration: 'fast',
    completeCallback: noop }
  /* … */ }

// From three@0.133.1/test/benchmark/benchmark.js:
SuiteUI.prototype.run = function() {
  this.runButton.click = noop;
  this.runButton.innerText = "Running..."
  this.suite.run({ async: true });
}

// From typeahead.js@0.11.1/src/typeahead/dataset.js
this.cancel = function cancel() {
  canceled = true;
  that.cancel = noop;
  that.async &&
    that.trigger('asyncCanceled', query);
};

// From typeahead.js@0.11.1/src/bloodhound/bloodhound.js
// “if max size is less than 0, provide a noop cache”
sync = sync || noop;
async = async || noop;
sync(this.remote ? local.slice() : local);

// From typeahead.js@0.11.1/src/bloodhound/lru_cache.js
// “if max size is less than 0, provide a noop cache”
if (this.maxSize <= 0) {
    this.set = this.get = $.noop;
}

// From verdaccio@5.1.6/packages/middleware/src/middleware.ts
errorReportingMiddleware(req, res, noop);

// From Odoo v15.0 addons/bus/static/src/js/services/bus_service.js
Promise.resolve(this._audio.play()).catch(noop);

// From ClickHouse v21.10.2.15-stable website/js/docsearch.js
if (this.$hint.length === 0) {
  this.setHint = this.getHint = this.clearHint = this.clearHintIfInvalid = noop;
}
```

Precedents include:
* [jQuery][]: [`$.noop`][]
* [lodash][]: [lodash.noop][] is individually downloaded from NPM
  about [415,600 times weekly][lodash.noop]

[`$.noop`]: https://www.npmjs.com/package/lodash.noop
[lodash.noop]: https://www.npmjs.com/package/lodash.noop

## Function.prototype.once
The `Function.prototype.once` method creates a new function
that calls the original function at most once,
no matter how much the new function is called.
```js
fn.once();

const fn = console.log.once();
fn(5); // Prints 5.
fn(5); // Does not print anything.
fn(5); // Does not print anything.

const initialize = createApplication.once();
initialize();
initialize();
// createApplication is invoked only once.
```

The following real-world example originally used [lodash.once][].

```js
// From Meteor v2.2.1.
// “Are we running Meteor from a git checkout?”
export const inCheckout = (function () {
  try { /* … */ } catch (e) { console.log(e); }
  return false;
}).once();

// From cypress@8.6.0.
cy.on('command:retry', _.after(2, (() => {
  button.remove() /* … */
}).once()))

// From Jitsi Meet v6482.
this._hangup = (() => {
 sendAnalytics(createToolbarEvent('hangup'));
 /* … */
}).once()
```

Precedents include:
* [lodash][]: [lodash.once][] is individually downloaded from NPM
  about [8,900,000 times weekly][lodash.once]
* [jQuery][]: [`.one`][]
* Node: [`eventEmitter.once`][]

[lodash.once]: https://www.npmjs.com/package/lodash.once
[`.one`]: https://api.jquery.com/one/
[`eventEmitter.once`]: https://nodejs.org/api/events.html#handling-events-only-once

## Function.prototype.debounce
The `Function.prototype.debounce` method creates a new function
that calls the original function at most once,
no matter how much the new function is called.
```js
fn.debounce(numOfMilliseconds);
```

Numerous graphical applications use `debounce`.
In this example, logging happens on keyup events from `inputEl`,
but only after the user has stopped typing for at least 250 ms:
```js
inputEl.addEventListener('keyup',
  console.log.debounce(250));
```

This method may come with options that could be bikeshedded in Stage 1.

Precedents include:
* [lodash][]: [lodash.debounce][] is individually downloaded from NPM
  about [11,400,000 times weekly][lodash.debounce]

[lodash.debounce]: https://www.npmjs.com/package/lodash.debounce

## Function.prototype.throttle
The `Function.prototype.throttle` method creates a new function that, when called,
calls the original function—but only at most once within a given length of time.
```js
fn.throttle(numOfMilliseconds);
```

Numerous graphical applications use `throttle`.
In this example, logging happens on window scroll, but no more than once every 250 ms:
```js
inputEl.addEventListener('keyup',
  console.log.throttle(250));
```

This method may come with options that could be bikeshedded in Stage 1.

Precedents include:
* [lodash][]: [lodash.throttle][] is individually downloaded from NPM
  about [3,700,000 times weekly][lodash.throttle]

[lodash.throttle]: https://www.npmjs.com/package/lodash.debounce

## Function.prototype.aside
The `Function.prototype.aside` method creates a new unary function
that applies some callback to its argument before returning the original argument.

```js
fn.aside();

const { aside } = Function;

console.log.aside(5); // Prints 5 before returning 5.

arr.map(console.log.aside).map(f);
// Prints each item from `arr` before passing them to `f`.

const data = await Promise.resolve('intro.txt')
  .then(Deno.open)
  .then(Deno.readAll)
  .then(console.log.aside())
  .then(data => new TextDecoder('utf-8').decode(data));
```

The following real-world example originally used [lodash][].aside and lodash/fp’s pipe.

```js
// From IBM/report-toolkit v0.6.1 packages/common/src/config.js
export function filterEnabledRules(config) {
  return pipe(
    config,
    _.getOr({}, 'rules'),
    _.toPairs,
    _.reduce(
      (enabledRules, [ruleName, ruleConfig]) =>
        (_.isObject(ruleConfig) && _.get('enabled', ruleConfig)) ||
        (_.isBoolean(ruleConfig) && ruleConfig)
          ? [ruleName, ...enabledRules]
          : enabledRules,
      []
    ),
    (ruleIds => {
      debug('found %d enabled rule(s)', ruleIds.length);
    }).aside();
}
```

Precedents include:
* [lodash][]: `_.tap`
* [Ramda][]: `import { tap } from 'ramda/src/tap';`

## Function.prototype.unThis

The `Function.prototype.unThis` method creates a new function
that calls the original function, supplying its first argument
as the original function’s `this` receiver,
and supplying the rest of its arguments as the original function’s ordinary arguments.

This is useful for converting functions
that rely on the dynamic this binding into functions
that only use their arguments.

```js
fn.unThis();

const $slice = Array.prototype.slice.unThis();
$slice([ 0, 1, 2 ], 1); // [ 1, 2 ]
```

This is not a substitute for a bind-this syntax,
which allows developers to change the receiver of functions
without creating a wrapper function.

`fn.unThis()` is equivalent to
`Function.prototype.bind.bind(Function.prototype.call)(fn)`.

Therefore, `fn.unThis()(thisArg, ...restArgs)` is equivalent
to `fn.call(thisArg, ...restArgs)`.

The following real-world example originally used [call-bind][]
or a manually created similar function.

```js
// From chrome-devtools-frontend@1.0.934332
// node_modules/array-includes/test/implementation.js
runTests(implementation.unThis(), t);

// From string.prototype.trimstart@1.0.4/index.js
var bound = getPolyfill().unThis();

// andreasgal/dom.js (84b7ab6) src/snapshot.js
const /* … */
  join = A.join || Array.prototype.join.unThis(),
  map = A.map || Array.prototype.map.unThis(),
  push = A.push || Array.prototype.push.unThis(),
  /* … */;
```

Precedents include:
* [call-bind][]: `callBind`

[lodash]: https://lodash.com/docs/4.17.15
[stdlib]: https://github.com/stdlib-js/stdlib
[RxJS]: https://rxjs.dev
[fp-ts]: https://gcanti.github.io/fp-ts/
[Ramda]: https://ramdajs.com/
[jQuery]: https://jquery.com/
[call-bind]: https://www.npmjs.com/package/call-bind

[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
