[**@black-flag/extensions**](../../../README.md)

***

[@black-flag/extensions](../../../README.md) / [src/error](../README.md) / ErrorMessage

# Variable: ErrorMessage

> `const` **ErrorMessage**: `object`

Defined in: [src/error.ts:14](https://github.com/Xunnamius/black-flag-extensions/blob/a33a5cac259d02354ae51b73a38791b29225ca19/src/error.ts#L14)

A collection of possible error and warning messages.

## Type declaration

### AppValidationFailure()

> **AppValidationFailure**: () => `string`

#### Returns

`string`

### AuthFailure()

> **AuthFailure**: () => `string`

#### Returns

`string`

### ClientValidationFailure()

> **ClientValidationFailure**: () => `string`

#### Returns

`string`

### GuruMeditation()

> **GuruMeditation**: () => `string`

#### Returns

`string`

### HttpFailure()

> **HttpFailure**: (`error`?) => `string`

#### Parameters

##### error?

`string`

#### Returns

`string`

### HttpSubFailure()

> **HttpSubFailure**: (`error`, `statusCode`) => `string`

#### Parameters

##### error

`null` | `string`

##### statusCode

`number`

#### Returns

`string`

### InvalidAppConfiguration()

> **InvalidAppConfiguration**: (`details`?) => `string`

#### Parameters

##### details?

`string`

#### Returns

`string`

### InvalidAppEnvironment()

> **InvalidAppEnvironment**: (`details`?) => `string`

#### Parameters

##### details?

`string`

#### Returns

`string`

### InvalidClientConfiguration()

> **InvalidClientConfiguration**: (`details`?) => `string`

#### Parameters

##### details?

`string`

#### Returns

`string`

### InvalidItem()

> **InvalidItem**: (`item`, `itemName`) => `string`

#### Parameters

##### item

`unknown`

##### itemName

`string`

#### Returns

`string`

### InvalidSecret()

> **InvalidSecret**: (`secretType`) => `string`

#### Parameters

##### secretType

`string`

#### Returns

`string`

### ItemNotFound()

> **ItemNotFound**: (`item`, `itemName`) => `string`

#### Parameters

##### item

`unknown`

##### itemName

`string`

#### Returns

`string`

### ItemOrItemsNotFound()

> **ItemOrItemsNotFound**: (`itemsName`) => `string`

#### Parameters

##### itemsName

`string`

#### Returns

`string`

### NotAuthenticated()

> **NotAuthenticated**: () => `string`

#### Returns

`string`

### NotAuthorized()

> **NotAuthorized**: () => `string`

#### Returns

`string`

### NotFound()

> **NotFound**: () => `string`

#### Returns

`string`

### NotImplemented()

> **NotImplemented**: () => `string`

#### Returns

`string`

### ValidationFailure()

> **ValidationFailure**: () => `string`

#### Returns

`string`

### AssertionFailureBadConfigurationPath()

#### Parameters

##### path

`unknown`

#### Returns

`string`

### AssertionFailureBadParameterCombination()

#### Returns

`string`

### AssertionFailureCannotExecuteMultipleTimes()

#### Returns

`string`

### AssertionFailureConfigureExecutionEpilogue()

#### Returns

`string`

### AssertionFailureDuplicateCommandName()

#### Parameters

##### parentFullName

`undefined` | `string`

##### name1

`string`

##### type1

`"name"` | `"alias"`

##### name2

`string`

##### type2

`"name"` | `"alias"`

#### Returns

`string`

### AssertionFailureInvalidCommandExport()

#### Parameters

##### name

`string`

#### Returns

`string`

### AssertionFailureInvocationNotAllowed()

#### Parameters

##### name

`string`

#### Returns

`string`

### AssertionFailureNoConfigurationLoaded()

#### Parameters

##### path

`string`

#### Returns

`string`

### AssertionFailureReachedTheUnreachable()

#### Returns

`string`

### AssertionFailureUseParseAsyncInstead()

#### Returns

`string`

### CheckFailed()

#### Parameters

##### currentArgument

`string`

#### Returns

`string`

### CommandHandlerNotAFunction()

#### Returns

`string`

### CommandNotImplemented()

#### Returns

`string`

### ConfigLoadFailure()

#### Parameters

##### path

`string`

#### Returns

`string`

### ConflictsViolation()

#### Parameters

##### conflicter

`string`

##### seenConflictingKeyValues

`ObjectEntries`

#### Returns

`string`

### DemandGenericXorViolation()

#### Parameters

##### demanded

`ObjectEntries`

#### Returns

`string`

### DemandIfViolation()

#### Parameters

##### demanded

`string`

##### demander

`ObjectEntry`

#### Returns

`string`

### DemandOrViolation()

#### Parameters

##### demanded

`ObjectEntries`

#### Returns

`string`

### DemandSpecificXorViolation()

#### Parameters

##### firstArgument

`ObjectEntry`

##### secondArgument

`ObjectEntry`

#### Returns

`string`

### DuplicateOptionName()

#### Parameters

##### name

`string`

#### Returns

`string`

### FalsyCommandExport()

#### Returns

`string`

### FrameworkError()

#### Parameters

##### error

`unknown`

#### Returns

`string`

### Generic()

#### Returns

`string`

### GracefulEarlyExit()

#### Returns

`string`

### IllegalExplicitlyUndefinedDefault()

#### Returns

`string`

### IllegalHandlerInvocation()

#### Returns

`string`

### ImpliesViolation()

#### Parameters

##### implier

`string`

##### seenConflictingKeyValues

`ObjectEntries`

#### Returns

`string`

### InvalidCharacters()

#### Parameters

##### str

`string`

##### violation

`string`

#### Returns

`string`

### InvalidConfigureArgumentsReturnType()

#### Returns

`string`

### InvalidConfigureExecutionContextReturnType()

#### Returns

`string`

### InvalidSubCommandInvocation()

#### Returns

`string`

### MetadataInvariantViolated()

#### Parameters

##### afflictedKey

`string`

#### Returns

`string`

### ReferencedNonExistentOption()

#### Parameters

##### referrerName

`string`

##### doesNotExistName

`string`

#### Returns

`string`

### RequiresViolation()

#### Parameters

##### requirer

`string`

##### missingRequiredKeyValues

`ObjectEntries`

#### Returns

`string`

### UnexpectedlyFalsyDetailedArguments()

#### Returns

`string`

### UnexpectedValueFromInternalYargsMethod()

#### Returns

`string`
