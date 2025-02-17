[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / BfeBuilderObject

# Type Alias: BfeBuilderObject\<CustomCliArguments, CustomExecutionContext\>

> **BfeBuilderObject**\<`CustomCliArguments`, `CustomExecutionContext`\>: `object`

Defined in: [src/index.ts:203](https://github.com/Xunnamius/black-flag-extensions/blob/58ca41292dc469d27da4ef365acd1d10c30aedca/src/index.ts#L203)

A version of the object type of the `builder` export accepted by Black Flag
that supports BFE's additional functionality.

## Type Parameters

• **CustomCliArguments** *extends* `Record`\<`string`, `unknown`\>

• **CustomExecutionContext** *extends* `ExecutionContext`

## Index Signature

\[`key`: `string`\]: [`BfeBuilderObjectValue`](BfeBuilderObjectValue.md)\<`CustomCliArguments`, `CustomExecutionContext`\>
