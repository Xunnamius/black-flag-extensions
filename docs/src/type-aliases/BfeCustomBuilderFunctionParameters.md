[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / BfeCustomBuilderFunctionParameters

# Type Alias: BfeCustomBuilderFunctionParameters\<CustomCliArguments, CustomExecutionContext, P\>

> **BfeCustomBuilderFunctionParameters**\<`CustomCliArguments`, `CustomExecutionContext`, `P`\>: `P` *extends* \[infer R, `...(infer S)`\] ? `S` *extends* \[infer T, `...(infer _U)`\] ? \[`R` & `object`, `T`, [`BfeStrictArguments`](BfeStrictArguments.md)\<`Partial`\<`CustomCliArguments`\>, `CustomExecutionContext`\> \| `undefined`\] : \[`R` & `object`, `...S`\] : `never`

Defined in: [src/index.ts:625](https://github.com/Xunnamius/black-flag-extensions/blob/a33a5cac259d02354ae51b73a38791b29225ca19/src/index.ts#L625)

A version of Black Flag's `builder` function parameters that exclude yargs
methods that are not supported by BFE.

## Type Parameters

• **CustomCliArguments** *extends* `Record`\<`string`, `unknown`\>

• **CustomExecutionContext** *extends* `ExecutionContext`

• **P** = `Parameters`\<[`BfBuilderFunction`](BfBuilderFunction.md)\<`CustomCliArguments`, `CustomExecutionContext`\>\>

## See

[withBuilderExtensions](../functions/withBuilderExtensions.md)
