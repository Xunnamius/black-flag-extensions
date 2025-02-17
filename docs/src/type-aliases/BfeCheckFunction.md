[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / BfeCheckFunction

# Type Alias: BfeCheckFunction()\<CustomCliArguments, CustomExecutionContext\>

> **BfeCheckFunction**\<`CustomCliArguments`, `CustomExecutionContext`\>: (`currentArgumentValue`, `argv`) => `Promisable`\<`unknown`\>

Defined in: [src/index.ts:559](https://github.com/Xunnamius/black-flag-extensions/blob/f26d26e5a4eef6b4a0f448bac9017f85ea6d5319/src/index.ts#L559)

This function is used to validate an argument passed to Black Flag.

## Type Parameters

• **CustomCliArguments** *extends* `Record`\<`string`, `unknown`\>

• **CustomExecutionContext** *extends* `ExecutionContext`

## Parameters

### currentArgumentValue

`any`

### argv

`Arguments`\<`CustomCliArguments`, `CustomExecutionContext`\>

## Returns

`Promisable`\<`unknown`\>

## See

[BfeBuilderObjectValueExtensions.check](BfeBuilderObjectValueExtensions.md#check)
