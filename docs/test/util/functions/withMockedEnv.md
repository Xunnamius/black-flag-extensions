[**@black-flag/extensions**](../../../README.md)

***

[@black-flag/extensions](../../../README.md) / [test/util](../README.md) / withMockedEnv

# Function: withMockedEnv()

> **withMockedEnv**(`test`, `simulatedEnv`, `__namedParameters`?): `Promise`\<`void`\>

Defined in: node\_modules/@-xun/test-mock-env/dist/packages/test-mock-env/src/index.d.ts:39

Mock `process.env` within the scope of `test`. Guaranteed to return
`process.env` to its original state no matter how `test` terminates.

It is not safe to run this function concurrently (e.g. with `Promise.all`).

## Parameters

### test

() => `Promisable`\<`void`\>

### simulatedEnv

`Record`\<`string`, `string`\>

### \_\_namedParameters?

[`MockedEnvOptions`](../type-aliases/MockedEnvOptions.md)

## Returns

`Promise`\<`void`\>
