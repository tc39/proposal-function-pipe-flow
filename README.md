# Function helpers for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2021.

* Formal specification: Not yet
* Babel plugin: Not yet
* [Proposal history][HISTORY.md]

[HISTORY.md]: https://github.com/js-choi/proposal-function-helpers/blob/main/HISTORY.md

This proposal would standardize some useful,
common helper functions that get downloaded from NPM a lot.

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

## Function.flow
The `Function.flow` static method creates a new function from several sub-functions.

```js
Function.flow(...fns);

const { flow } = Function;

const f = flow(f0, f1, f2);
f(5, 7); // f2(f1(f0(5, 7))).

const g = flow(g0);
g(5, 7); // g0(5, 7).

const h = flow();
h(5, 7); // 5.

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

// From strapi@3.6.8/packages/strapi-admin/services/permission/permissions-manager/query-builers.js
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
applies its own arguments to its leftmost sub-function.
Then that result is applied to its next sub-function.
In other words, function composition occurs from left to right.

The leftmost sub-function may have any arity,
but any subsequent sub-functions are expected to be unary.

If `Function.flow` receives no arguments, then, by default,
it will return `Function.identity` (which is defined later in this proposal).

Precedents include:
* [lodash][]: [lodash.flow][] is individually downloaded from NPM
  about [600,000 times weekly][lodash.flow]
* [fp-ts][]: `import { flow } from 'fp-ts/function';`
* [Ramda][]: `import { pipe } from 'ramda/src/pipe';`
* [RxJS][]: `import { pipe } from 'rxjs';`

[lodash.flow]: https://www.npmjs.com/package/lodash.flow

## Function.pipe
The `Function.pipe` static method applies a sequence
of sub-functions to a given input value, returning the final sub-function’s result.

```js
Function.pipe(input, ...fns);

const { pipe } = Function;

// f2(f1(f0(5))).
pipe(5, f0, f1, f2);

// 5.
pipe(5);

// undefined.
pipe();

// From pico@1.0.1/source/inline.ts
return pipe(
  download(absoluteURL),
  mapRej(downloadErrorToDetailedError),
  chainFluture(responseToBlob),
  chainFluture(blobToDataURL),
  mapFluture(dataURL => `url(${dataURL})`)
)
```

The first sub-function is applied to `input`,
then the second sub-function is applied to the first sub-function’s result,
and so forth.
In other words, function piping occurs from left to right.

Each sub-function is expected to be a unary function.

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
of potentially async sub-functions to a given input value, returning a promise.
The promise will resolve to the final sub-function’s result.

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
Then the first sub-function is applied to `input` and then `await`ed,
then the second sub-function is applied to the first sub-function’s result then `await`ed,
and so forth.
In other words, function piping occurs from left to right.

Each sub-function is expected to be a unary function.

If any sub-function returns a promise that then rejects with an error,
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

// From cypress@8.6.0/packages/net-stubbing/lib/server/util.ts
setDefaultHeader('access-control-expose-headers', constant('*'))

// From cypress@8.6.0/packages/driver/src/cypress/utils.ts
return [fn, constant(type)]

// From <https://github.com/odoo/odoo/blob/15.0/addons/pad/static/src/js/pad.js>
url.toJSON = constant(this.url);

// From ng-table@3.0.1/test/specs/settings.spec.ts
const newSettings: = {
  filterOptions: _.mapValues(allSettings.filterOptions, constant(undefined)),
  dataOptions: _.mapValues(allSettings.dataOptions, constant(undefined)),
  groupOptions: _.mapValues(allSettings.groupOptions, constant(undefined))
};

// From <https://github.com/elastic/kibana/blob/v7.15.1>
{
  name: 0,
  size: 378611,
  aggConfig: {
    type: 'histogram',
    schema: 'segment',
    fieldFormatter: constant(String),
    params: {
      interval: 1000,
      extended_bounds: {},
    },
  },
},
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

// From cypress@8.6.0/packages/driver/src/cypress/runner.ts
// “iterates over a suite's tests (including nested suites)
// and will return as soon as the callback is true”
const findTestInSuite = (suite, fn = identity) => {
  for (const test of suite.tests) {
    if (fn(test)) {
      return test
    }
  }
}

// From gatsby@3.14.3/packages/gatsby-plugin-sharp/src/plugin-options.js
// “get all non falsey values”
return _.pickBy(options, identity)

// From gatsby@3.14.3/packages/gatsby-plugin-gatsby-cloud/src/constants.js
export const DEFAULT_OPTIONS = {
  // “optional transform for manipulating headers for sorting, etc”
  transformHeaders: identity,
}

// From ghost@4.19.0/core/frontend/helpers/img_url.js
// “CASE: only make paths relative if we didn't get a request for an absolute url”
const maybeEnsureRelativePath = !absoluteUrlRequested ? ensureRelativePath : _.identity;
```

Precedents include:
* [lodash][]: [lodash.identity][] is individually downloaded from NPM
  about [700,000 times weekly][lodash.identity]
* [stdlib][]: `import identity from '@stdlib/utils-identity-function';`
* [fp-ts][]: `import { identity } from 'fp-ts/function';`
* [Ramda][]: `import { identity } from 'ramda/src/identity';`

[lodash.identity]: https://www.npmjs.com/package/lodash.identity

## Function.tap
The `Function.tap` static method creates a new unary function
that applies some callback to its argument before returning the original argument.

```js
Function.tap(callback)(input);

const { tap } = Function;

tap(console.log)(5); // Prints 5 before returning 5.

arr.map(tap(console.log)).map(f); // Prints each item from `arr` before passing them to `f`.

const data = await Promise.resolve('intro.txt')
  .then(Deno.open)
  .then(Deno.readAll)
  .then(tap(console.log))
  .then(data => new TextDecoder('utf-8').decode(data));

// From testdouble@3.16.2/src/constructor.js
var fakeConstructorFromNames = (funcNames) => {
  return tap(tdFunction('(unnamed constructor)'), (fakeConstructor) => {
    _.each(funcNames, (funcName) => {
      fakeConstructor.prototype[funcName] = tdFunction(`#${String(funcName)}`)
    })
  })
}

// From <https://github.com/rendrjs/rendr/blob/1.1.4/test/shared/fetcher.test.js>
function getModelResponse(version, id, addJsonKey) {
  if (addJsonKey) {
    return tap({}, function(obj) {
      obj.listing = resp;
    });
  }
}
```

Precedents include:
* [lodash][]: `_.tap`
* [Ramda][]: `import { tap } from 'ramda/src/tap';`

[lodash]: https://lodash.com/docs/4.17.15
[stdlib]: https://github.com/stdlib-js/stdlib
[RxJS]: https://rxjs.dev
[fp-ts]: https://gcanti.github.io/fp-ts/
[Ramda]: https://ramdajs.com/

[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
