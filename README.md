<!-- badges-start -->

[![Black Lives Matter!][x-badge-blm-image]][x-badge-blm-link]
[![Last commit timestamp][x-badge-lastcommit-image]][x-badge-repo-link]
[![Codecov][x-badge-codecov-image]][x-badge-codecov-link]
[![Source license][x-badge-license-image]][x-badge-license-link]
[![Monthly Downloads][x-badge-downloads-image]][x-badge-npm-link]
[![NPM version][x-badge-npm-image]][x-badge-npm-link]
[![Uses Semantic Release!][x-badge-semanticrelease-image]][x-badge-semanticrelease-link]

<!-- badges-end -->

# @black-flag/extensions

Black Flag Extensions (AKA: @black-flag/extensions, BFE) is a collection of
high-order functions that wrap your commands' exports to provide a bevy of new
declarative features, some of which are heavily inspired by [yargs's GitHub
Issues reports][1].

If you're willing to stray a bit from the vanilla yargs API, you can use BFE to
greatly increase Black Flag's declarative powers.

> [Why are @black-flag/extensions and @black-flag/core separate packages?][2]

---

<!-- remark-ignore-start -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Install](#install)
- [Usage](#usage)
  - [`withBuilderExtensions`](#withbuilderextensions)
  - [`withUsageExtensions`](#withusageextensions)
- [Examples](#examples)
  - [Using `demandThisOption`, `demandThisOptionIf`, `conflicts`, `requires`, and `default`](#using-demandthisoption-demandthisoptionif-conflicts-requires-and-default)
- [Appendix](#appendix)
  - [Differences between Black Flag Extensions and Yargs](#differences-between-black-flag-extensions-and-yargs)
  - [Black Flag versus Black Flag Extensions](#black-flag-versus-black-flag-extensions)
  - [Published Package Details](#published-package-details)
  - [License](#license)
- [Contributing and Support](#contributing-and-support)
  - [Contributors](#contributors)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- remark-ignore-end -->

## Install

```shell
npm install @black-flag/extensions
```

## Usage

> See also: [differences between BFE and Yargs][3].

### `withBuilderExtensions`

> ⪢ API reference: [`withBuilderExtensions`][4]

This function enables several additional options-related units of functionality
via analysis of the returned options configuration object and the parsed command
line arguments (i.e. `argv`).

Note that options passed to configuration keys, e.g.
`{ demandThisOptionXor: ['my‑argument', 'my‑argument‑2'] }`, are represented by
their exact names as defined (e.g. `'my‑argument'`) and not their aliases
(`'arg1'`) or camelCase forms (`'myArgument'`).

For example:

```javascript
import { withBuilderExtensions } from '@black-flag/extensions';

export default function command({ state }) {
  const [builder, withHandlerExtensions] = withBuilderExtensions(
    (blackFlag) => {
      blackFlag.strict(false);

      // ▼ The "returned options configuration object"
      return {
        'my-argument': {
          alias: ['arg1'],
          demandThisOptionXor: ['arg2'],
          string: true
        },
        arg2: {
          boolean: true,
          demandThisOptionXor: ['my-argument']
        }
      };
    },
    { disableAutomaticGrouping: true }
  );

  return {
    name: 'my-command',
    builder,
    handler: withHandlerExtensions(({ myArgument, arg2 }) => {
      state.outputManager.log(
        'Executing command with arguments: arg1=${myArgument} arg2=${arg2}'
      );
    })
  };
}
```

#### New Option Configuration Keys

The following new configuration keys enable additional options-related units of
functionality beyond that offered by vanilla yargs and Black Flag:

> In the below definitions, `P`, `Q`, and `R` are arguments (or argument-value
> pairs) configured via [`blackFlag.options({ P: { [key]: [Q, R] }})`][5] and
> truth values represent the existence of said arguments.

| Key                         | Definition                      |
| :-------------------------- | :------------------------------ |
| [`requires`][6]             | `P ⟹ Q ∧ R` or `¬P ∨ Q ∧ R`     |
| [`conflicts`][7]            | `P ⟹ ¬Q ∧ ¬R` or `¬P ∨ ¬Q ∧ ¬R` |
| [`demandThisOptionIf`][8]   | `Q ∨ R ⟹ P` or `P ∨ ¬Q ∧ ¬R`    |
| [`demandThisOption`][9]     | `P`                             |
| [`demandThisOptionOr`][10]  | `P ∨ Q ∨ R`                     |
| [`demandThisOptionXor`][11] | `P ⊕ Q ⊕ R`                     |
| [`check`][12]               | N/A                             |
| [`subOptionOf`][13]         | N/A                             |

##### `requires`

> `requires` is a superset of and replacement for vanilla yargs's
> [`implies`][14]. However, `implies` is not disallowed by intellisense. If both
> are specified, they will both be considered by Black Flag (and yargs). It is
> recommended to avoid `implies` entirely [due to its ambiguity][15].

> `{ P: { requires: [Q, R] }}` can be read as `P ⟹ Q ∧ R` or `¬P ∨ Q ∧ R`, with
> truth values denoting existence.

`requires` enables checks to ensure the specified arguments, or argument-value
pairs, are given conditioned on the existence of another argument. For example:

```jsonc
{
  "x": { "requires": "y" }, // ◄ Disallows x without y
  "y": {}
}
```

This configuration will trigger a check to ensure that `‑y` is given whenever
`‑x` is given.

`requires` also supports checks against the parsed _values_ of arguments in
addition to the mere argument existence checks demonstrated above. For example:

```jsonc
{
  // ▼ Disallows x unless y == 'one' and z is given
  "x": { "requires": [{ "y": "one" }, "z"] },
  "y": {},
  "z": { "requires": "y" } // ◄ Disallows z unless y is given
}
```

This configuration allows the following arguments: no arguments (`∅`), `‑y=...`,
`‑y=... ‑z`, `‑xz ‑y=one`; and disallows: `‑x`, `‑z`, `‑x ‑y=...`, `‑xz ‑y=...`,
`‑xz`.

##### `conflicts`

> `conflicts` is a superset of vanilla yargs's [`conflicts`][16].

> `{ P: { conflicts: [Q, R] }}` can be read as `P ⟹ ¬Q ∧ ¬R` or `¬P ∨ ¬Q ∧ ¬R`,
> with truth values denoting existence.

`conflicts` enables checks to ensure the specified arguments, or argument-value
pairs, are given conditioned on the existence of another argument. For example:

```jsonc
{
  "x": { "conflicts": "y" }, // ◄ Disallows y if x is given
  "y": {}
}
```

This configuration will trigger a check to ensure that `‑y` is never given
whenever `‑x` is given.

`conflicts` also supports checks against the parsed _values_ of arguments in
addition to the mere argument existence checks demonstrated above. For example:

```jsonc
{
  // ▼ Disallows y == 'one' or z if x is given
  "x": { "conflicts": [{ "y": "one" }, "z"] },
  "y": {},
  "z": { "conflicts": "y" } // ◄ Disallows y if z is given
}
```

This configuration allows the following arguments: no arguments (`∅`), `‑y=...`,
`‑x`, `‑z`, `‑x ‑y=...`; and disallows: `‑y=... ‑z`, `‑x ‑y=one`, `‑xz ‑y=one`,
`‑xz`.

##### `demandThisOptionIf`

> `demandThisOptionIf` is a superset of vanilla yargs's [`demandOption`][17].

> `{ P: { demandThisOptionIf: [Q, R] }}` can be read as `Q ∨ R ⟹ P` or
> `P ∨ ¬Q ∧ ¬R`, with truth values denoting existence.

`demandThisOptionIf` enables checks to ensure an argument is given when at least
one of the specified group of arguments, or argument-value pairs, is also given.

For example:

```jsonc
{
  "x": {},
  "y": { "demandThisOptionIf": "x" }, // ◄ Demands y if x is given
  "z": { "demandThisOptionIf": "x" } // ◄ Demands z if x is given
}
```

This configuration allows the following arguments: no arguments (`∅`), `‑y`,
`‑z`, `‑yz`, `‑xyz`; and disallows: `‑x`, `‑xy`, `‑xz`.

`demandThisOptionIf` also supports checks against the parsed _values_ of
arguments in addition to the mere argument existence checks demonstrated above.
For example:

```jsonc
{
  // ▼ Demands x if y == 'one' or z is given
  "x": { "demandThisOptionIf": [{ "y": "one" }, "z"] },
  "y": {},
  "z": {}
}
```

This configuration allows the following arguments: no arguments (`∅`), `‑x`,
`‑y=...`, `‑x ‑y=...`, `‑xz`, `‑y=... ‑z`, `-xz y=...`; and disallows: `‑z`,
`‑y=one`, `‑y=one ‑z`.

##### `demandThisOption`

> `demandThisOption` is an alias of vanilla yargs's [`demandOption`][17].
> However, `demandOption` is not disallowed by intellisense. If both are
> specified, the configuration defined last will win.

> `{ P: { demandThisOption: true }}` can be read as `P`, with truth values
> denoting existence.

`demandThisOption` enables checks to ensure an argument is always given. This is
equivalent to `demandOption` from vanilla yargs. For example:

```jsonc
{
  "x": { "demandThisOption": true }, // ◄ Disallows ∅, y
  "y": { "demandThisOption": false }
}
```

This configuration will trigger a check to ensure that `‑x` is given.

##### `demandThisOptionOr`

> `demandThisOptionOr` is a superset of vanilla yargs's [`demandOption`][17].

> `{ P: { demandThisOptionOr: [Q, R] }}` can be read as `P ∨ Q ∨ R`, with truth
> values denoting existence.

`demandThisOptionOr` enables non-optional inclusive disjunction checks per
group. Put another way, `demandThisOptionOr` enforces a "logical or" relation
within groups of required options. For example:

```jsonc
{
  "x": { "demandThisOptionOr": ["y", "z"] }, // ◄ Demands x or y or z
  "y": { "demandThisOptionOr": ["x", "z"] }, // ◄ Mirrors the above (discarded)
  "z": { "demandThisOptionOr": ["x", "y"] } // ◄ Mirrors the above (discarded)
}
```

This configuration will trigger a check to ensure _at least one_ of `x`, `y`, or
`z` is given. In other words, this configuration allows the following arguments:
`‑x`, `‑y`, `‑z`, `‑xy`, `‑xz`, `‑yz`, `‑xyz`; and disallows: no arguments
(`∅`).

In the interest of readability, consider mirroring the appropriate
`demandThisOptionOr` configuration to the other relevant options, though this is
not required (redundant groups are discarded). The previous example demonstrates
proper mirroring.

`demandThisOptionOr` also supports checks against the parsed _values_ of
arguments in addition to the mere argument existence checks demonstrated above.
For example:

```jsonc
{
  // ▼ Demands x or y == 'one' or z
  "x": { "demandThisOptionOr": [{ "y": "one" }, "z"] },
  "y": {},
  "z": {}
}
```

This configuration allows the following arguments: `‑x`, `‑y=one`, `‑z`,
`‑x ‑y=...`, `‑xz`, `‑y=... ‑z`, `‑xz ‑y=...`; and disallows: no arguments
(`∅`), `‑y=...`.

##### `demandThisOptionXor`

> `demandThisOptionXor` is a superset of vanilla yargs's [`demandOption`][17] +
> [`conflicts`][16].

> `{ P: { demandThisOptionXor: [Q, R] }}` can be read as `P ⊕ Q ⊕ R`, with truth
> values denoting existence.

`demandThisOptionXor` enables non-optional exclusive disjunction checks per
exclusivity group. Put another way, `demandThisOptionXor` enforces mutual
exclusivity within groups of required options. For example:

```jsonc
{
  "x": { "demandThisOptionXor": ["y"] }, // ◄ Disallows ∅, z, w, xy, xyw, xyz, xyzw
  "y": { "demandThisOptionXor": ["x"] }, // ◄ Mirrors the above (discarded)
  "z": { "demandThisOptionXor": ["w"] }, // ◄ Disallows ∅, x, y, zw, xzw, yzw, xyzw
  "w": { "demandThisOptionXor": ["z"] } // ◄ Mirrors the above (discarded)
}
```

This configuration will trigger a check to ensure _exactly one_ of `‑x` or `‑y`
is given, and _exactly one_ of `‑z` or `‑w` is given. In other words, this
configuration allows the following arguments: `‑xz`, `‑xw`, `‑yz`, `‑yw`; and
disallows: no arguments (`∅`), `‑x`, `‑y`, `‑z`, `‑w`, `‑xy`, `‑zw`, `‑xyz`,
`‑xyw`, `‑xzw`, `‑yzw`, `‑xyzw`.

In the interest of readability, consider mirroring the appropriate
`demandThisOptionXor` configuration to the other relevant options, though this
is not required (redundant groups are discarded). The previous example
demonstrates proper mirroring.

`demandThisOptionXor` also supports checks against the parsed _values_ of
arguments in addition to the mere argument existence checks demonstrated above.
For example:

```jsonc
{
  // ▼ Demands x xor y == 'one' xor z
  "x": { "demandThisOptionXor": [{ "y": "one" }, "z"] },
  "y": {},
  "z": {}
}
```

This configuration allows the following arguments: `‑x`, `‑y=one`, `‑z`,
`‑x ‑y=...`, `‑y=... ‑z`; and disallows: no arguments (`∅`), `‑y=...`,
`‑x ‑y=one`, `‑xz`, `‑y=one ‑z`, `‑xz ‑y=...`.

##### `check`

`check` is declarative sugar around [`yargs::check()`][18] that is applied
specifically to the option being configured. These option-specific custom check
functions are run on Black Flag's [second parsing pass][19].

When a check fails, execution of its command's handler function will cease and
[`configureErrorHandlingEpilogue`][20] will be invoked (unless you threw a
[`GracefulEarlyExitError`][21]).

> Note that there is no concept of a "global" check in this context. If you want
> that, you'll have to call `blackFlag.check(...)` imperatively or implement the
> appropriate configuration hooks (see [the bullet point on `yargs::check`][22]
> in the Black Flag docs).

For example:

```javascript
export const builder = withBuilderExtensions({
  x: {
    number: true,
    check: function (currentXArgValue, fullArgv) {
      if (currentXArgValue < 0 || currentXArgValue > 10) {
        throw new Error(
          `"x" must be between 0 and 10 (inclusive), saw: ${currentXArgValue}`
        );
      }
    }
  },
  y: {
    boolean: true,
    default: false,
    requires: 'x',
    check: function (currentYArgValue, fullArgv) {
      if (currentYArgValue && fullArgv.x <= 5) {
        throw new Error(
          `"x" must be greater than 5 to use 'y', saw: ${fullArgv.x}`
        );
      }
    }
  }
});
```

See the yargs documentation on [`yargs::check()`][18] for more information.

##### `subOptionOf`

One of Black Flag's killer features is [native support for dynamic options][23].
However, taking advantage of this feature in your commands' [`builder`][24]
exports requires a strictly imperative approach.

Take, for example, [the `init` command from @black-flag/demo][25]:

```javascript
// Taken at 06/04/2024
// @ts-check

/**
 * @type {import('@black-flag/core').Configuration['builder']}
 */
export const builder = function (yargs, _, argv) {
  yargs.parserConfiguration({ 'parse-numbers': false });

  if (argv && argv.lang) {
    // This code block implements our dynamic options (depending on --lang)
    return argv.lang === 'node'
      ? {
          lang: { choices: ['node'], demandOption: true },
          version: { choices: ['19.8', '20.9', '21.1'], default: '21.1' }
        }
      : {
          lang: { choices: ['python'], demandOption: true },
          version: {
            choices: ['3.10', '3.11', '3.12'],
            default: '3.12'
          }
        };
  } else {
    // This code block represents the fallback
    return {
      lang: { choices: ['node', 'python'], demandOption: true },
      version: { string: true, default: 'latest' }
    };
  }
};

/**
 * @type {import('@black-flag/core').Configuration<{ lang: string, version: string }>['handler']}
 */
export const handler = function ({ lang, version }) {
  console.log(`> Initializing new ${lang}@${version} project...`);
};
```

Among other freebies, taking advantage of dynamic options support gifts your CLI
with help text more gorgeous and meaningful than anything you could accomplish
with vanilla yargs:

```text
myctl init --lang 'node' --version=21.1
> initializing new node@21.1 project...
```

```text
myctl init --lang 'python' --version=21.1
Usage: myctl init

Options:
  --help     Show help text                                            [boolean]
  --lang                                                     [choices: "python"]
  --version                                    [choices: "3.10", "3.11", "3.12"]

Invalid values:
  Argument: version, Given: "21.1", Choices: "3.10", "3.11", "3.12"
```

```text
myctl init --lang fake
Usage: myctl init

Options:
  --help     Show help text                                            [boolean]
  --lang                                             [choices: "node", "python"]
  --version                                                             [string]

Invalid values:
  Argument: lang, Given: "fake", Choices: "node", "python"
```

```text
myctl init --help
Usage: myctl init

Options:
  --help     Show help text                                            [boolean]
  --lang                                             [choices: "node", "python"]
  --version                                                             [string]
```

Ideally, Black Flag would allow us to describe the relationship between `--lang`
and `--version` _declaratively_, without having to drop down to imperative
interactions with the yargs API like we did above.

This is the goal of the `subOptionOf` configuration key. Using `subOptionOf`,
developers can take advantage of dynamic options without sweating the
implementation details.

> Note that `subOptionOf` updates are run and applied during Black Flag's
> [second parsing pass][19].

For example:

```javascript
/**
 * @type {import('@black-flag/core').Configuration['builder']}
 */
export const builder = withBuilderExtensions({
  x: {
    choices: ['a', 'b', 'c'],
    demandThisOption: true,
    description: 'A choice'
  },
  y: {
    number: true,
    description: 'A number'
  },
  z: {
    // ▼ These configurations are applied as the baseline or "fallback" during
    //   Black Flag's first parsing pass. The updates within subOptionOf are
    //   evaluated and applied during Black Flag's second parsing pass.
    boolean: true,
    description: 'A useful context-sensitive flag',
    subOptionOf: {
      // ▼ Ignored if x is not given
      x: [
        {
          when: (currentXArgValue, fullArgv) => currentXArgValue === 'a',
          update:
            // ▼ We can pass an updater function that returns an opt object.
            //   This object will *replace* the argument's old configuration!
            (oldXArgumentConfig, fullArgv) => {
              return {
                // ▼ We don't want to lose the old config, so we spread it
                ...oldXArgumentConfig,
                description: 'This is a switch specifically for the "a" choice'
              };
            }
        },
        {
          when: (currentXArgValue, fullArgv) => currentXArgValue !== 'a',
          update:
            // ▼ Or we can just pass the replacement configuration object. Note
            //   that, upon multiple `when` matches, the last update in the
            //   chain will win. If you want merge behavior instead of overwrite,
            //   spread the old config in the object you return.
            {
              string: true,
              description: 'This former-flag now accepts a string instead'
            }
        }
      ],
      // ▼ Ignored if y is not given. If x and y ARE given, since this occurs
      //   after the x config, this update will overwrite any others. Use the
      //   functional form + object spread to preserve the old configuration
      y: {
        when: (currentYArgValue, fullArgv) =>
          fullArgv.x === 'a' && currentYArgValue > 5,
        update: (oldConfig, fullArgv) => {
          return {
            array: true,
            demandThisOption: true,
            description:
              'This former-flag now accepts an array of two or more strings',
            check: function (currentZArgValue, fullArgv) {
              if (currentZArgValue.length < 2) {
                throw new Error(
                  `"z" must be an array of two or more strings', only saw: ${currentZArgValue.length}`
                );
              }
            }
          };
        }
      },
      // ▼ Since "does-not-exist" is not an option defined anywhere, this will
      //   always be ignored
      'does-not-exist': []
    }
  }
});
```

> Note that you cannot nest `subOptionOf` keys within each other and will
> trigger a framework error. `subOptionOf` is only valid during Black Flag's
> first parsing pass.

Now we're ready to re-implement the `init` command from `myctl` using our new
declarative superpowers:

```javascript
export const builder = withBuilderExtensions(function (blackFlag) {
  blackFlag.parserConfiguration({ 'parse-numbers': false });

  return {
    lang: {
      choices: ['node', 'python'],
      demandOption: true,
      subOptionOf: {
        lang: [
          {
            when: (lang) => lang === 'node',
            // ▼ Remember: updates overwrite any old config (including baseline)
            update: {
              choices: ['node'],
              demandOption: true
            }
          },
          {
            when: (lang) => lang !== 'node',
            update: {
              choices: ['python'],
              demandOption: true
            }
          }
        ]
      }
    },
    version: {
      string: true,
      default: 'latest',
      subOptionOf: {
        version: [
          {
            when: (_, { lang }) => lang === 'node',
            update: {
              choices: ['19.8', '20.9', '21.1'],
              default: '21.1'
            }
          },
          {
            when: (_, { lang }) => lang !== 'node',
            update: {
              choices: ['3.10', '3.11', '3.12'],
              default: '3.12'
            }
          }
        ]
      }
    }
  };
});
```

Easy peasy!

#### Support for `default` with `conflicts`/`requires` Et Al

<!-- TODO: fix me -->

TODO: footgun avoided for free

#### Impossible Configurations

Note that **there are no sanity checks performed to prevent demands that are
impossible to fulfil**, so care must be taken not to ask for something insane.

For example, the following configurations are impossible to fulfil:

```jsonc
{
  "x": { "requires": "y" },
  "y": { "conflicts": "x" }
}
```

```jsonc
{
  "x": { "requires": "y", "demandThisOptionXor": "y" },
  "y": {}
}
```

#### Automatic Grouping of Related Options

<!-- TODO: fix me -->

Automatic grouping of related options. To support this functionality,
`blackFlag.options(...)` and `blackFlag.option(...)` are no longer callable from
within commands' `builder` functions. To add options to a command, they must be
described declaratively, i.e. via returning an options object from your
`builder` function.

This feature can be disabled by passing `{ disableAutomaticGrouping: true }` to
`withBuilderExtensions`.

### `withUsageExtensions`

<!-- TODO: fix me -->

TODO

## Examples

<!-- TODO: fix me -->

TODO

### Using `demandThisOption`, `demandThisOptionIf`, `conflicts`, `requires`, and `default`

<!-- TODO: fix me -->

Suppose we wanted a "deploy" command with the following features:

- Ability to deploy to a Vercel production target, a Vercel preview target, or
  to a remote target via SSH.

- When deploying to Vercel, allow the user to choose to deploy _only_ to preview
  or _only_ to production, if desired.

- When deploying to Vercel, deploy to the preview target by default.

- When deploying to a remote target via SSH,

What follows is an example implementation:

```typescript
import { type ChildConfiguration } from '@black-flag/core';
import {
  withBuilderExtensions,
  withUsageExtensions
} from '@black-flag/extensions';

import { type CustomExecutionContext } from '../configure.ts';

// ▼ Let's keep our custom CLI arguments strongly 💪🏿 typed
export type CustomCliArguments = { target: DeployTargets } & (
  | {
      target: 'vercel';
      // We could make this subtype even stronger, but the returns are diminishing...
      production: boolean;
      preview: boolean;
    }
  | {
      target: 'ssh';
      host: string;
      toPath: string;
    }
);

export const deployTargets = ['vercel', 'ssh'] as const;
export type DeployTargets = (typeof deployTargets)[number];

export default function command({ state }: CustomExecutionContext) {
  const [builder, withHandlerExtensions] =
    withBuilderExtensions<CustomCliArguments>({
      target: {
        demandThisOption: true, // ◄ Just an alias for { demandOption: true }
        choices: deployTargets,
        description: 'Select deployment target and strategy'
      },
      production: {
        alias: ['prod'],
        boolean: true,
        conflicts: 'preview', // ◄ Normal vanilla yargs "conflicts" semantics
        requires: { target: 'vercel' }, // ◄ Error if --target != vercel
        default: false, // ◄ Works in a sane way alongside "conflicts"/"requires"
        description: 'Only deploy to the remote production environment'
      },
      preview: {
        boolean: true,
        conflicts: 'production',
        requires: { target: 'vercel' },
        default: true,
        description: 'Only deploy to the remote preview environment'
      },
      host: {
        string: true,
        // ▼ Inverse of { conflicts: { target: 'vercel' }} in this example
        requires: { target: 'ssh' }, // ◄ Error if --target != ssh
        demandThisOptionIf: { target: 'ssh' }, // ◄ Demand --host if --target=ssh
        description: 'The host to use'
      },
      'to-path': {
        string: true,
        requires: { target: 'ssh' }, // ◄ Error if --target != ssh
        demandThisOptionIf: { target: 'ssh' }, // ◄ Demand --to-path if --target=ssh
        description: 'The deploy destination path to use'
      }
    });

  return {
    builder,
    description: 'Deploy distributes to the appropriate remote',
    usage: withUsageExtensions('$1.\n\nSupports both Vercel and SSH targets!'),
    handler: withHandlerExtensions<CustomCliArguments>(async function ({
      target,
      production: productionOnly,
      preview: previewOnly,
      host,
      toPath
    }) {
      switch (target) {
        case 'vercel': {
          // if(productionOnly) ...
          break;
        }

        case 'ssh': {
          // ...
          break;
        }
      }
    })
  } satisfies ChildConfiguration<CustomCliArguments, CustomExecutionContext>;
}
```

#### Example Outputs

<!-- TODO: fix me -->

TODO: takes advantage of dynamic options support! Better help text with
requires + demandThisOptionIf

<!-- TODO: fix me -->

TODO: demandThisOptionIf + demandOption + others and how they overlap

<!-- TODO: fix me -->

TODO: python example again?

## Appendix

Further documentation can be found under [`docs/`][x-repo-docs].

### Differences between Black Flag Extensions and Yargs

When using BFE, command options must be configured by [returning an `opt`
object][5] from your command's `builder` rather than imperatively invoking the
yargs API.

For example:

```diff
export function builder(blackFlag) {
- // DO NOT use yargs's imperative API to define options! This *BREAKS* BFE!
- blackFlag.option('f', {
-   alias: 'file',
-   demandOption: true,
-   default: '/etc/passwd',
-   describe: 'x marks the spot',
-   type: 'string'
- });
-
- // DO NOT use yargs's imperative API to define options! This *BREAKS* BFE!
- blackFlag
-   .alias('f', 'file')
-   .demandOption('f')
-   .default('f', '/etc/passwd')
-   .describe('f', 'x marks the spot')
-   .string('f');
+
+ // INSTEAD, use Black Flag's declarative API to define options 🙂
+ return {
+   f: {
+     alias: 'file',
+     demandOption: true,
+     default: '/etc/passwd',
+     describe: 'x marks the spot',
+     type: 'string'
+   }
+ };
}
```

> The yargs API can still be invoked for purposes other than defining options on
> a command, e.g. `blackFlag.strict(false)`.

To this end, the following [yargs API functions][26] are soft-disabled via
intellisense:

- `option`
- `options`

However, no attempt is made by BFE to restrict your use of the yargs API at
runtime. Therefore, using yargs's API to work around these artificial
limitations, e.g. in your command's [`builder`][24] function or via the
[`configureExecutionPrologue`][27] hook, will result in **undefined behavior**.

### Black Flag versus Black Flag Extensions

The goal of [Black Flag (@black-flag/core)][28] is to be as close to a drop-in
replacement as possible for vanilla yargs, specifically for users of
[`yargs::commandDir()`][29]. This means Black Flag must go out of its way to
maintain 1:1 parity with the vanilla yargs API ([with a few minor
exceptions][30]).

As a consequence, yargs's imperative nature tends to leak through Black Flag's
abstraction at certain points, such as with [the `blackFlag` parameter of the
`builder` export][24]. **This is a good thing!** Since we want access to all of
yargs's killer features without Black Flag getting in the way.

However, this comes with costs. For one, the yargs's API has suffered from a bit
of feature creep over the years. A result of this is a rigid API [with][31]
[an][32] [abundance][33] [of][34] [footguns][35] and an [inability][36] to
[address][37] them without introducing [massively][38] [breaking][39]
[changes][40].

BFE takes the "YOLO" approach by exporting several functions that build on top
of Black Flag's feature set without worrying too much about maintaining 1:1
parity with the vanilla yargs's API. This way, one can opt-in to a more
opinionated but (in my opinion) cleaner, more consistent, and more intuitive
developer experience.

### Published Package Details

This is a [CJS2 package][x-pkg-cjs-mojito] with statically-analyzable exports
built by Babel for Node.js versions that are not end-of-life. For TypeScript
users, this package supports both `"Node10"` and `"Node16"` module resolution
strategies.

<details><summary>Expand details</summary>

That means both CJS2 (via `require(...)`) and ESM (via `import { ... } from ...`
or `await import(...)`) source will load this package from the same entry points
when using Node. This has several benefits, the foremost being: less code
shipped/smaller package size, avoiding [dual package
hazard][x-pkg-dual-package-hazard] entirely, distributables are not
packed/bundled/uglified, a drastically less complex build process, and CJS
consumers aren't shafted.

Each entry point (i.e. `ENTRY`) in [`package.json`'s
`exports[ENTRY]`][x-repo-package-json] object includes one or more [export
conditions][x-pkg-exports-conditions]. These entries may or may not include: an
[`exports[ENTRY].types`][x-pkg-exports-types-key] condition pointing to a type
declarations file for TypeScript and IDEs, an
[`exports[ENTRY].module`][x-pkg-exports-module-key] condition pointing to
(usually ESM) source for Webpack/Rollup, an `exports[ENTRY].node` condition
pointing to (usually CJS2) source for Node.js `require` _and `import`_, an
`exports[ENTRY].default` condition pointing to source for browsers and other
environments, and [other conditions][x-pkg-exports-conditions] not enumerated
here. Check the [package.json][x-repo-package-json] file to see which export
conditions are supported.

Though [`package.json`][x-repo-package-json] includes
[`{ "type": "commonjs" }`][x-pkg-type], note that any ESM-only entry points will
be ES module (`.mjs`) files. Finally, [`package.json`][x-repo-package-json] also
includes the [`sideEffects`][x-pkg-side-effects-key] key, which is `false` for
optimal [tree shaking][x-pkg-tree-shaking] where appropriate.

</details>

### License

See [LICENSE][x-repo-license].

## Contributing and Support

**[New issues][x-repo-choose-new-issue] and [pull requests][x-repo-pr-compare]
are always welcome and greatly appreciated! 🤩** Just as well, you can [star 🌟
this project][x-badge-repo-link] to let me know you found it useful! ✊🏿 Or you
could [buy me a beer][x-repo-sponsor] 🥺Thank you!

See [CONTRIBUTING.md][x-repo-contributing] and [SUPPORT.md][x-repo-support] for
more information.

### Contributors

<!-- remark-ignore-start -->
<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->

[![All Contributors](https://img.shields.io/badge/all_contributors-1-orange.svg?style=flat-square)](#contributors-)

<!-- ALL-CONTRIBUTORS-BADGE:END -->
<!-- remark-ignore-end -->

Thanks goes to these wonderful people ([emoji
key][x-repo-all-contributors-emojis]):

<!-- remark-ignore-start -->
<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->

<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://xunn.io/"><img src="https://avatars.githubusercontent.com/u/656017?v=4?s=100" width="100px;" alt="Bernard"/><br /><sub><b>Bernard</b></sub></a><br /><a href="#infra-Xunnamius" title="Infrastructure (Hosting, Build-Tools, etc)">🚇</a> <a href="https://github.com/Xunnamius/black-flag-extensions/commits?author=Xunnamius" title="Code">💻</a> <a href="https://github.com/Xunnamius/black-flag-extensions/commits?author=Xunnamius" title="Documentation">📖</a> <a href="#maintenance-Xunnamius" title="Maintenance">🚧</a> <a href="https://github.com/Xunnamius/black-flag-extensions/commits?author=Xunnamius" title="Tests">⚠️</a> <a href="https://github.com/Xunnamius/black-flag-extensions/pulls?q=is%3Apr+reviewed-by%3AXunnamius" title="Reviewed Pull Requests">👀</a></td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td align="center" size="13px" colspan="7">
        <img src="https://raw.githubusercontent.com/all-contributors/all-contributors-cli/1b8533af435da9854653492b1327a23a4dbd0a10/assets/logo-small.svg">
          <a href="https://all-contributors.js.org/docs/en/bot/usage">Add your contributions</a>
        </img>
      </td>
    </tr>
  </tfoot>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->
<!-- ALL-CONTRIBUTORS-LIST:END -->
<!-- remark-ignore-end -->

This project follows the [all-contributors][x-repo-all-contributors]
specification. Contributions of any kind welcome!

[x-badge-blm-image]: https://xunn.at/badge-blm 'Join the movement!'
[x-badge-blm-link]: https://xunn.at/donate-blm
[x-badge-codecov-image]:
  https://img.shields.io/codecov/c/github/Xunnamius/black-flag-extensions/main?style=flat-square&token=HWRIOBAAPW
  'Is this package well-tested?'
[x-badge-codecov-link]: https://codecov.io/gh/Xunnamius/black-flag-extensions
[x-badge-downloads-image]:
  https://img.shields.io/npm/dm/@black-flag/extensions?style=flat-square
  'Number of times this package has been downloaded per month'
[x-badge-lastcommit-image]:
  https://img.shields.io/github/last-commit/xunnamius/black-flag-extensions?style=flat-square
  'Latest commit timestamp'
[x-badge-license-image]:
  https://img.shields.io/npm/l/@black-flag/extensions?style=flat-square
  "This package's source license"
[x-badge-license-link]:
  https://github.com/Xunnamius/black-flag-extensions/blob/main/LICENSE
[x-badge-npm-image]:
  https://xunn.at/npm-pkg-version/@black-flag/extensions
  'Install this package using npm or yarn!'
[x-badge-npm-link]: https://npmtrends.com/@black-flag/extensions
[x-badge-repo-link]: https://github.com/xunnamius/black-flag-extensions
[x-badge-semanticrelease-image]:
  https://xunn.at/badge-semantic-release
  'This repo practices continuous integration and deployment!'
[x-badge-semanticrelease-link]:
  https://github.com/semantic-release/semantic-release
[x-pkg-cjs-mojito]:
  https://dev.to/jakobjingleheimer/configuring-commonjs-es-modules-for-nodejs-12ed#publish-only-a-cjs-distribution-with-property-exports
[x-pkg-dual-package-hazard]:
  https://nodejs.org/api/packages.html#dual-package-hazard
[x-pkg-exports-conditions]:
  https://webpack.js.org/guides/package-exports#reference-syntax
[x-pkg-exports-module-key]:
  https://webpack.js.org/guides/package-exports#providing-commonjs-and-esm-version-stateless
[x-pkg-exports-types-key]:
  https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta#packagejson-exports-imports-and-self-referencing
[x-pkg-side-effects-key]:
  https://webpack.js.org/guides/tree-shaking#mark-the-file-as-side-effect-free
[x-pkg-tree-shaking]: https://webpack.js.org/guides/tree-shaking
[x-pkg-type]:
  https://github.com/nodejs/node/blob/8d8e06a345043bec787e904edc9a2f5c5e9c275f/doc/api/packages.md#type
[x-repo-all-contributors]: https://github.com/all-contributors/all-contributors
[x-repo-all-contributors-emojis]: https://allcontributors.org/docs/en/emoji-key
[x-repo-choose-new-issue]:
  https://github.com/xunnamius/black-flag-extensions/issues/new/choose
[x-repo-contributing]: /CONTRIBUTING.md
[x-repo-docs]: docs
[x-repo-license]: ./LICENSE
[x-repo-package-json]: package.json
[x-repo-pr-compare]: https://github.com/xunnamius/black-flag-extensions/compare
[x-repo-sponsor]: https://github.com/sponsors/Xunnamius
[x-repo-support]: /.github/SUPPORT.md
[1]: https://github.com/yargs/yargs/issues
[2]: #black-flag-versus-black-flag-extensions
[3]: #differences-between-black-flag-extensions-and-yargs
[4]: ./docs/functions/withBuilderExtensions.md
[5]: https://yargs.js.org/docs#api-reference-optionskey-opt
[6]: #requires
[7]: #conflicts
[8]: #demandthisoptionif
[9]: #demandthisoption
[10]: #demandthisoptionor
[11]: #demandthisoptionxor
[12]: #check
[13]: #subOptionOf
[14]: https://yargs.js.org/docs#implies
[15]: https://github.com/yargs/yargs/issues/1322#issuecomment-1538709884
[16]: https://yargs.js.org/docs#conflicts
[17]: https://yargs.js.org/docs#demandOption
[18]: https://yargs.js.org/docs#api-reference-checkfn-globaltrue
[19]:
  https://github.com/Xunnamius/black-flag/tree/main?tab=readme-ov-file#motivation
[20]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/ConfigureErrorHandlingEpilogue.md
[21]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/classes/GracefulEarlyExitError.md
[22]:
  https://github.com/Xunnamius/black-flag/tree/main?tab=readme-ov-file#irrelevant-differences
[23]:
  https://github.com/Xunnamius/black-flag/tree/main?tab=readme-ov-file#built-in-support-for-dynamic-options-
[24]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/Configuration.md#builder
[25]: https://github.com/Xunnamius/black-flag-demo/blob/main/commands/init.js
[26]: https://yargs.js.org/docs#api-reference
[27]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/ConfigureExecutionPrologue.md
[28]: https://npm.im/@black-flag/core
[29]: https://yargs.js.org/docs#api-reference-commanddirdirectory-opts
[30]:
  https://github.com/Xunnamius/black-flag?tab=readme-ov-file#differences-between-black-flag-and-yargs
[31]: https://github.com/yargs/yargs/issues/1323
[32]: https://github.com/yargs/yargs/issues/1442
[33]: https://github.com/yargs/yargs/issues/2340
[34]: https://github.com/yargs/yargs/issues/1322
[35]: https://github.com/yargs/yargs/issues/2089
[36]: https://github.com/yargs/yargs/issues/1975
[37]: https://github.com/yargs/yargs-parser/issues/412
[38]: https://github.com/yargs/yargs/issues/1680
[39]: https://github.com/yargs/yargs/issues/1599
[40]: https://github.com/yargs/yargs/issues/1611
