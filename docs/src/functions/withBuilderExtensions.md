[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / withBuilderExtensions

# Function: withBuilderExtensions()

> **withBuilderExtensions**\<`CustomCliArguments`, `CustomExecutionContext`\>(`customBuilder`?, `__namedParameters`?): [`WithBuilderExtensionsReturnType`](../type-aliases/WithBuilderExtensionsReturnType.md)\<`CustomCliArguments`, `CustomExecutionContext`\>

Defined in: [src/index.ts:721](https://github.com/Xunnamius/black-flag-extensions/blob/f26d26e5a4eef6b4a0f448bac9017f85ea6d5319/src/index.ts#L721)

This function enables several additional options-related units of
functionality via analysis of the returned options configuration object and
the parsed command line arguments (argv).

## Type Parameters

• **CustomCliArguments** *extends* `Record`\<`string`, `unknown`\>

• **CustomExecutionContext** *extends* `ExecutionContext`

## Parameters

### customBuilder?

[`BfeBuilderObject`](../type-aliases/BfeBuilderObject.md)\<`CustomCliArguments`, `CustomExecutionContext`\> | (...`args`) => `void` \| [`BfeBuilderObject`](../type-aliases/BfeBuilderObject.md)\<`CustomCliArguments`, `CustomExecutionContext`\>

### \_\_namedParameters?

[`WithBuilderExtensionsConfig`](../type-aliases/WithBuilderExtensionsConfig.md)\<`CustomCliArguments`\> = `{}`

## Returns

[`WithBuilderExtensionsReturnType`](../type-aliases/WithBuilderExtensionsReturnType.md)\<`CustomCliArguments`, `CustomExecutionContext`\>

## See

[WithBuilderExtensionsReturnType](../type-aliases/WithBuilderExtensionsReturnType.md)
