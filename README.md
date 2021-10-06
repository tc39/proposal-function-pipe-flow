# Function helpers for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2021.

* Formal specification: Not yet
* Babel plugin: Not yet
* [Proposal history][HISTORY.md]

[HISTORY.md]: https://github.com/js-choi/proposal-function-helpers/blob/main/HISTORY.md

There are several higher-order functions that are often used in functional programming.
This proposal would add them to the `Function` global object as static methods.

## Function.flow
The `Function.flow` static method creates a new function from several sub-functions.

```js
Function.flow(...fns);

const { flow } = Function;

const f = flow((x, y) => x + y, x => x * 2, x => x + 1);
f(5, 7); // Evalutes to ((5 + 7) * 2) + 1.

const g = flow(x => x + 1);
g(5); // Evaluates to 6.

const h = flow();
h(5); // Evalutes to 5.
```

Any function created by `Function.flow`
applies its own arguments to its leftmost sub-function.
Then that result is applied to its next sub-function.
In other words, function composition occurs from left to right.

The leftmost sub-function may have any arity,
but any subsequent sub-functions are expected to be unary.

If `Function.flow` receives no arguments, then, by default,
it will return `Function.identity` (which is defined later in this proposal).

| Precedent | Example
| --------- | ---------
|[lodash][] |`_.flow`
|[fp-ts][] |`import { flow } from 'fp-ts/function';`
|[Ramda][] |`R.pipe`

## Function.constant
The `Function.constant` static method creates a new function from a constant value.
The new function will always return that value, no matter what arguments it is given.

```js
Function.constant(value);

const { constant } = Function;

const f = constant(5);
f(11, 0, 3); // Evaluates to 5.

const g = constant();
g(11, 0, 3); // Evaluates to undefined.
```

| Precedent | Example
| --------- | ---------
|[lodash][] |`_.constant`
|[stdlib][] |`import constantFunction from '@stdlib/utils-constant-function';`
|[fp-ts][] |`import { constant } from 'fp-ts/function';`
|[Ramda][] |`R.always`

## Function.identity
The `Function.identity` static method always returns its first argument.

```js
Function.identity(value);

const { identity } = Function;

identity(5); // Evaluates to 5.
identity(); // Evaluates to undefined.
```

| Precedent | Example
| --------- | ---------
|[lodash][] |`_.identity`
|[stdlib][] |`import identity from '@stdlib/utils-identity-function';`
|[fp-ts][] |`import { identity } from 'fp-ts/function';`
|[Ramda][] |`R.identity`

## Function.pipe
The `Function.pipe` static method applies a sequence
of sub-functions to a given input value, returning the final sub-function’s result.

```js
Function.pipe(input, ...fns);

const { pipe } = Function;

// Evalutes to (5 + 7) * 2.
pipe(5,
  x => x + 1,
  x => x * 2,
);

// Evaluates to 5.
pipe(5);

// Evaluates to undefined.
pipe();

// Throws a SyntaxError.
pipe('x', JSON.parse);
```

The first sub-function is applied to `input`,
then the second sub-function is applied to the first sub-function’s result,
and so forth.
In other words, function piping occurs from left to right.

Each sub-function is expected to be a unary function.

If `Function.pipe` receives only one argument, then it will return `input` by default.\
If `Function.pipe` receives no arguments, then it will return `undefined`.

| Precedent | Example
| --------- | ---------
|[RxJS][]   |`rx.pipe`
|[fp-ts][] |`import { pipe } from 'fp-ts/function';`

### What happened to the F# pipe operator?
F#, Haskell, and other languages that are based on auto-curried unary functions
have a tacit-unary-function-application operator.
The pipe champion group has presented F# pipes for Stage 2 twice to TC39,
being unsuccessful both times
due to pushback from multiple other TC39 representatives’
memory performance concerns, syntax concerns about await,
and concerns about encouraging ecosystem bifurcation/forking.
(For more information, see the [pipe proposal’s HISTORY.md][pipe history].)

Given this reality, TC39 is immensely more likely to pass
a `Function.pipe` helper function than a similar syntactic operator.

Standardizing a helper function does not preclude
standardizing an equivalent operator later.
For example, TC39 standardized binary `**` even when `Math.pow` existed.

In the future, I might try to propose a F# pipe operator,
but I would like to try proposing `Function.pipe` first,
in an effort to bring its benefits to the wider JavaScript community
as soon as possible.

## Function.pipeAsync
The `Function.pipeAsync` static method applies a sequence
of potentially async sub-functions to a given input value, returning a promise.
The promise will resolve to the final sub-function’s result.

```js
Function.pipeAsync(input, ...fns);

const { pipeAsync } = Function;

// A promise that will resolve to ((await (await fetch(await url, options)).json()) + 1) * 2.
pipeAsync(url,
  x => fetch(x, options),
  x => x.json(),
  x => x + 1,
  x => x * 2,
);

// A promise that will resolve to 5.
pipeAsync(5);

// A promise that will resolve to undefined.
pipeAsync();

// A promise that will reject with a SyntaxError.
pipeAsync('x', JSON.parse);
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

[lodash]: https://lodash.com/docs/4.17.15
[stdlib]: https://github.com/stdlib-js/stdlib
[RxJS]: https://rxjs.dev
[fp-ts]: https://gcanti.github.io/fp-ts/
[Ramda]: https://ramdajs.com/

[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
