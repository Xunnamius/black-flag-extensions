[**@black-flag/extensions**](../../README.md)

***

[@black-flag/extensions](../../README.md) / [src](../README.md) / withUsageExtensions

# Function: withUsageExtensions()

> **withUsageExtensions**(`altDescription`, `__namedParameters`): `string`

Defined in: [src/index.ts:1309](https://github.com/Xunnamius/black-flag-extensions/blob/58ca41292dc469d27da4ef365acd1d10c30aedca/src/index.ts#L1309)

Generate command usage text consistently yet flexibly.

Defaults to: `Usage: $000\n\n${altDescription}` where `altDescription` is
`$1.`

## Parameters

### altDescription

`string` = `'$1.'`

### \_\_namedParameters

#### appendPeriod?

`boolean` = `true`

Whether a period will be appended to the resultant string or not. A
period is only appended if one is not already appended.

**Default**

```ts
true
```

#### includeOptions?

`boolean` = `prependNewlines`

Whether the string `' [...options]'` will be appended to the first line of usage text

**Default**

```ts
options.prependNewlines
```

#### prependNewlines?

`boolean` = `true`

Whether newlines will be prepended to `altDescription` or not.

**Default**

```ts
true
```

#### trim?

`boolean` = `true`

Whether `altDescription` will be `trim()`'d or not.

**Default**

```ts
true
```

## Returns

`string`
