[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / AsStrictExecutionContext

# Type Alias: AsStrictExecutionContext\<CustomExecutionContext\>

> **AsStrictExecutionContext**\<`CustomExecutionContext`\>: `OmitIndexSignature`\<`Exclude`\<`CustomExecutionContext`, `"state"`\>\> & `OmitIndexSignature`\<`CustomExecutionContext`\[`"state"`\]\>

Defined in: [src/index.ts:615](https://github.com/Xunnamius/black-flag-extensions/blob/58ca41292dc469d27da4ef365acd1d10c30aedca/src/index.ts#L615)

Maps an ExecutionContext into an identical type that explicitly omits
its fallback indexers for unrecognized properties. Even though it is the
runtime equivalent of ExecutionContext, using this type allows
intellisense to report bad/misspelled/missing arguments from `context` in
various places where it otherwise couldn't.

**This type is intended for intellisense purposes only.**

## Type Parameters

• **CustomExecutionContext** *extends* `ExecutionContext`
