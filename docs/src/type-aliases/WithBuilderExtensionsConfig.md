[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / WithBuilderExtensionsConfig

# Type Alias: WithBuilderExtensionsConfig\<CustomCliArguments\>

> **WithBuilderExtensionsConfig**\<`CustomCliArguments`\>: `object`

Defined in: [src/index.ts:683](https://github.com/Xunnamius/black-flag-extensions/blob/f26d26e5a4eef6b4a0f448bac9017f85ea6d5319/src/index.ts#L683)

A configuration object that further configures the behavior of
[withBuilderExtensions](../functions/withBuilderExtensions.md).

## Type Parameters

• **CustomCliArguments** *extends* `Record`\<`string`, `unknown`\>

## Type declaration

### commonOptions?

> `optional` **commonOptions**: readonly `LiteralUnion`\<keyof `CustomCliArguments` \| `"help"` \| `"version"`, `string`\>[]

An array of zero or more string keys of `CustomCliArguments`, with the
optional addition of `'help'` and `'version'`, that should be grouped under
_"Common Options"_ when [automatic grouping of related
options](https://github.com/Xunnamius/black-flag-extensions?tab=readme-ov-file#automatic-grouping-of-related-options)
is enabled.

This setting is ignored if `disableAutomaticGrouping === true`.

#### Default

```ts
['help']
```

### disableAutomaticGrouping?

> `optional` **disableAutomaticGrouping**: `boolean`

Set to `true` to disable BFE's support for automatic grouping of related
options.

See [the
documentation](https://github.com/Xunnamius/black-flag-extensions?tab=readme-ov-file#automatic-grouping-of-related-options)
for details.

#### Default

```ts
false
```
