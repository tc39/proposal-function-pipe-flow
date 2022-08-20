# Function.pipe and flow for JavaScript
ECMAScript Stage-0 Proposal (**withdrawn**). J. S. Choi, 2021.

**⚠️ This proposal is withdrawn.** [In the plenary on July 21, proposal-function-pipe-flow was formally presented to the Committee, and it was rejected for Stage 1][2022-07 plenary]. The Committee generally found its use cases either easily solved by **userland functions**, such as:
```js
function pipe (input, ...fnArray) {
  return fnArray.reduce((value, fn) => fn(value), input);
}
```
…or also solved by the [pipe operator][]. Its champion subsequently withdrew it from consideration. (Eventually, after the pipe operator gains users, pain points with the pipe operator may be enough motivation to revive this proposal, but that would not occur for a long time.)

[2022-07 plenary]: https://github.com/tc39/notes/blob/main/meetings/2022-07/jul-21.md#functionpipe--flow-for-stage-1
[pipe operator]: https://github.com/tc39/proposal-pipeline-operator

* [Proposal history][HISTORY.md]

[HISTORY.md]: https://github.com/js-choi/proposal-function-pipe-flow/blob/main/HISTORY.md

<details>

<summary>Original proposal</summary>

Serial function application and function composition are useful and common in
JavaScript. Developers often divide code into many smaller **unary callbacks**,
which are then sequentially called with some initial input—or which are composed
into larger callbacks that will sequentially call those functions later.

To do this, they often combine these callbacks with helper functions: **pipe**
(serial function application) and **flow** and/or **compose** (function
composition). It would be useful to standardize these metafunctions.

The problem space here is the **application and composition of serial
callbacks**. Much of the JavaScript community already has been serially
applying or composing callbacks.

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

If this proposal is approved for Stage 1, then we would explore various
directions for serially applying and composing unary callbacks. Additionally,
we would assemble as many real-world use cases as possible and shape our design
to fulfill them.

(This problem space is already being explored to some extent by the [pipe
operator’s proposal][pipe]. However, unlike in this proposal, the pipe operator
can “apply” any kind of expression operation, not only unary function calls.
The tradeoff is that the pipe operator involves new syntax; additionally, the
pipe function is more concise for unary function calls and can handle dynamic
arrays of callbacks. In Stage 1, we would continue to examine cross-cutting
concerns and overlap with the pipe operator and other “dataflow” proposals.)

## Solutions
We could add various combinations of the following static functions:

* `Function.pipe` and `Function.pipeAsync` (for serial application).
* `Function.flow` and Function.flowAsync (for LTR serial composition).
* `Function.compose` and `Function.composeAsync` (for RTL serial composition).

(LTR = left to right. RTL = right to left.)

<details>

<summary>There are eight possible combinations of `pipe`, `flow`, and `compose`.</summary>

* Choice #0: **Status quo**
* Choice #1: LTR **flow**

  ```js
  Function.flow(f, g, h);
  Function.flowAsync(f, g, h);
  ```

* Choice #2: RTL **compose**

  ```js
  Function.compose(h, g, f);
  Function.composeAsync(h, g, f);
  ```

* Choice #3: LTR **flow** & RTL **compose**

  ```js
  Function.flow(f, g, h);
  Function.flowAsync(f, g, h);
  Function.compose(h, g, f);
  Function.composeAsync(h, g, f);
  ```

* Choice #4: **Pipe**

  ```js
  Function.pipe(x, f, g, h);
  Function.pipeAsync(x, f, g, h);
  ```

* Choice #5: **Pipe** & LTR **flow**

  ```js
  Function.pipe(x, f, g, h);
  Function.pipeAsync(x, f, g, h);
  Function.flow(f, g, h);
  Function.flowAsync(f, g, h);
  ```

* Choice #6: **Pipe** & RTL **compose**

  ```js
  Function.pipe(x, f, g, h);
  Function.pipeAsync(x, f, g, h);
  Function.compose(h, g, f);
  Function.composeAsync(h, g, f);
  ```

* Choice #7: **Pipe**, LTR **flow** & RTL **compose**

  ```js
  Function.pipe(x, f, g, h);
  Function.pipeAsync(x, f, g, h);
  Function.flow(f, g, h);
  Function.flowAsync(f, g, h);
  Function.compose(h, g, f);
  Function.composeAsync(h, g, f);
  ```

</details>

***

<details>
<summary>What happened to the F# pipe operator?</summary>

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

## (LTR) Function.flow
The `Function.flow` static method creates a new function by combining
several callbacks in left-to-right order.

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

The leftmost callback may have any arity,
but any subsequent callbacks are expected to be unary.

If `Function.flow` receives no arguments, then, by default,
it will return a unary identity function.

Precedents include:
* [lodash][]: [lodash.flow][] is individually downloaded from NPM
  about [600,000 times weekly][lodash.flow]
* [fp-ts][]: `import { flow } from 'fp-ts/function';`
* [Ramda][]: `import { pipe } from 'ramda/src/pipe';`
* [RxJS][]: `import { pipe } from 'rxjs';`

[lodash.flow]: https://www.npmjs.com/package/lodash.flow

## (LTR) Function.flowAsync
The `Function.flowAsync` static method creates a new function
by combining several potentially async callbacks in left-to-right order;
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

The leftmost callback may have any arity,
but any subsequent callbacks are expected to be unary.

If `Function.flowAsync` receives no arguments, then, by default,
it will return `Promise.resolve`.

## (RTL) Function.compose
The `Function.compose` static method creates a new function by combining
several callbacks in right-to-left order.

```js
Function.compose(...fns);

const { compose } = Function;

const f = compose(f2, f1, f0);
f(5, 7); // f2(f1(f0(5, 7))).

const g = compose(g0);
g(5, 7); // g0(5, 7).

const h = compose();
h(5, 7); // 5.
```

Any function created by `Function.compose`
applies its own arguments to its rightmost callback.
Then that result is applied to its next callback.

The rightmost callback may have any arity,
but any subsequent callbacks are expected to be unary.

If `Function.compose` receives no arguments, then, by default,
it will return a unary identity function.

## (RTL) Function.composeAsync
The `Function.composeAsync` static method creates a new function
by combining several potentially async callbacks in right-to-left order;
the created function will always return a promise.

```js
Function.composeAsync(...fns);

const { composeAsync } = Function;

// async (...args) => await f2(await f1(await f0(...args))).
composeAsync(f2, f1, f0);

const f = composeAsync(f2, f1, f0);
await f(5, 7); // await f2(await f1(await f0(5, 7))).

const g = composeAsync(g0);
await g(5, 7); // await g0(5, 7).

const h = composeAsync();
await h(5, 7); // await 5.
```

Any function created by `Function.composeAsync`
applies its own arguments to its rightmost callback.
Then that result is `await`ed before being applied to its next callback.
In other words, async function composition occurs from left to right.

The rightmost callback may have any arity,
but any subsequent callbacks are expected to be unary.

If `Function.composeAsync` receives no arguments, then, by default,
it will return `Promise.resolve`.

[lodash]: https://lodash.com/docs/4.17.15
[stdlib]: https://github.com/stdlib-js/stdlib
[RxJS]: https://rxjs.dev
[fp-ts]: https://gcanti.github.io/fp-ts/
[Ramda]: https://ramdajs.com/

[pipe]: https://github.com/tc39/proposal-pipeline-operator
[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md

</details>
