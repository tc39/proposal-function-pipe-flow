# Function.pipe and flow for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2021.

* Formal specification: Not yet
* Babel plugin: Not yet
* [Proposal history][HISTORY.md]

[HISTORY.md]: https://github.com/js-choi/proposal-function-pipe-flow/blob/main/HISTORY.md

Serial function application and function composition are useful and common in
JavaScript. Developers often divide code into many smaller **unary functions**,
which are then sequentially called with some initial input—or which are composed
into larger callbacks that will sequentially call those functions later.

To do this, they tend to use one or two functions:
**pipe** (serial function application) and **flow** (function composition).
It would be useful to standardize these functions.

This is true even in spite of the [Hack pipe operator][pipe].
The pipe operator is useful for generically flattening deeply nested expressions
into unidirectional, linear chains.
These expressions include `await` expressions, property access,
function calls, and arithmetic expressions.
However, many developers frequently serially apply or compose **unary
functions** – and, when doing so with the Hack pipe operator, they must repeat
`(%)` at each step:

From StoplightIO Prism v4.5.0 packages/http/src/validator/validators/body.ts.

```js
return pipe(
  specs,
  A.findFirst(spec => !!typeIs(mediaType, [spec.mediaType])),
  O.alt(() => A.head(specs)),
  O.map(content => ({ mediaType, content }))
);
```

From strapi@3.6.8
packages/strapi-admin/services/permission/permissions-manager/query-builers.js:
```js
const transform = flow(flattenDeep, cleanupUnwantedProperties);
```

From semantic-ui-react@v2.0.4/docs/static/utils/getInfoForSeeTags.js:
```js
const getInfoForSeeTags = _.flow(
  _.get('docblock.tags'),
  _.filter((tag) => tag.title === 'see'),
  _.map((tag) => { /* … */ }),
)
```

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
// From @gripeless/pico@1.0.1/source/inline.ts:
return pipe(
  download(absoluteURL),
  mapRej(downloadErrorToDetailedError),
  chainFluture(responseToBlob),
  chainFluture(blobToDataURL),
  mapFluture(dataURL => `url(${dataURL})`)
)

// From StoplightIO Prism v4.5.0 packages/http/src/validator/validators/body.ts:
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
// From gatsby@3.14.3/packages/gatsby-plugin-sharp/src/plugin-options.js:
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
// packages/strapi-admin/services/permission/permissions-manager/query-builers.js:
const transform = flow(flattenDeep, cleanupUnwantedProperties);

// From semantic-ui-react@v2.0.4/docs/static/utils/getInfoForSeeTags.js:
const getInfoForSeeTags = flow(
  _.get('docblock.tags'),
  _.filter((tag) => tag.title === 'see'),
  _.map((tag) => { /* … */ }),
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

// async (...args) => await f2(await f1(await f0(...args))).
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

[lodash]: https://lodash.com/docs/4.17.15
[stdlib]: https://github.com/stdlib-js/stdlib
[RxJS]: https://rxjs.dev
[fp-ts]: https://gcanti.github.io/fp-ts/
[Ramda]: https://ramdajs.com/

[pipe]: https://github.com/tc39/proposal-pipeline-operator
[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
