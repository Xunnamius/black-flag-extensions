[**@black-flag/extensions**](../../../README.md)

***

[@black-flag/extensions](../../../README.md) / [test/util](../README.md) / withMockedExit

# Function: withMockedExit()

> **withMockedExit**(`test`): `Promise`\<`void`\>

Defined in: node\_modules/@-xun/test-mock-exit/dist/packages/test-mock-exit/src/index.d.ts:9

Mock `process.exit` within the scope of `test`. Guaranteed to return
`process.env` to its original state no matter how `test` terminates.

It is not safe to run this function concurrently (e.g. with `Promise.all`).

## Parameters

### test

(`spies`) => `Promisable`\<`void`\>

## Returns

`Promise`\<`void`\>
