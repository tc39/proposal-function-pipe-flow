# Brief history of JavaScript function helpers
Function operations in JavaScript has a long and twisty history.
That history can give essential context behind this proposal.

For information on what “Stage 1” and “Stage 2” mean,
read about the [TC39 Process][].

More information about contributing is also available in [CONTRIBUTING.md][].

## 2015–2021
The pipe champion group presents F# pipes (a tacit-unary-function-application operator)
for Stage 2 twice to TC39, being unsuccessful both times
due to pushback from multiple other TC39 representatives’
memory performance concerns, syntax concerns about await,
and concerns about encouraging ecosystem bifurcation/forking.

For more information, see the [pipe proposal’s HISTORY.md][pipe history].

## 2021-09
Inspired by [pipe issue #233][], [@js-choi][] creates a new proposal
that would add several Function helper methods.

[TC39 process]: https://tc39.es/process-document/
[CONTRIBUTING.md]: https://github.com/tc39/proposal-pipeline-operator/blob/main/CONTRIBUTING.md

[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
[pipe issue #233]: https://github.com/tc39/proposal-pipeline-operator/issues/233

[@js-choi]: https://github.com/js-choi
