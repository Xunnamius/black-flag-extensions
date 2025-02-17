[**@black-flag/extensions**](../../../README.md)

***

[@black-flag/extensions](../../../README.md) / [test/util](../README.md) / withMockedOutput

# Function: withMockedOutput()

> **withMockedOutput**(`test`, `__namedParameters`?): `Promise`\<`void`\>

Defined in: node\_modules/@-xun/test-mock-output/dist/packages/test-mock-output/src/index.d.ts:30

Mock terminal output functions within the scope of `test`. Guaranteed to
return terminal output functions to their original state no matter how `test`
terminates.

It is not safe to run this function concurrently (e.g. with `Promise.all`).

## Parameters

### test

(`spies`) => `unknown`

### \_\_namedParameters?

[`MockedOutputOptions`](../type-aliases/MockedOutputOptions.md)

## Returns

`Promise`\<`void`\>
