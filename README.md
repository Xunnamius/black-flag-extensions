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
line arguments (argv).

Note that options provided to configuration keys like `demandThisOptionXor` are
represented by their exact names as defined (e.g. `'my-argument'`) and not their
aliases (`'arg1'`) or camelCase forms (`'myArgument'`).

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
    }
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

> In the below definitions, `P`, `Q`, and `R` are arguments configured via
> [`blackFlag.options({ P: { [key]: [Q, R] }})`][5].

| Key                         | Definition                      |
| :-------------------------- | :------------------------------ |
| [`requires`][6]             | `P ⟹ Q ∧ R` or `¬P ∨ Q ∧ R`     |
| [`conflicts`][7]            | `P ⟹ ¬Q ∧ ¬R` or `¬P ∨ ¬Q ∧ ¬R` |
| [`demandThisOptionIf`][8]   | `Q ∨ R ⟹ P` or `P ∨ ¬Q ∧ ¬R`    |
| [`demandThisOption`][9]     | `P`                             |
| [`demandThisOptionOr`][10]  | `P ∨ Q ∨ R`                     |
| [`demandThisOptionXor`][11] | `P ⊕ Q ⊕ R`                     |
| [`subOptionOf`][12]         | N/A                             |

##### `requires`

> `requires` is a superset of and replacement for vanilla yargs's
> [`implies`][13]. However, `implies` is not disallowed by intellisense. If both
> are specified, they will both be considered by Black Flag (and yargs). It is
> recommended to avoid `implies` entirely [due to its ambiguity][14].

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

> `conflicts` is a superset of vanilla yargs's [`conflicts`][15].

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

> `demandThisOptionIf` is a superset of vanilla yargs's [`demandOption`][16].

> `{ P: { demandThisOptionIf: [Q, R] }}` can be read as `Q ∨ R ⟹ P` or
> `P ∨ ¬Q ∧ ¬R`, with truth values denoting existence.

`demandThisOptionIf` enables checks to ensure the specified option is given when
at least one of a group of arguments, or argument-value pairs, is also given.

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

> `demandThisOption` is an alias of vanilla yargs's [`demandOption`][16].
> However, `demandOption` is not disallowed by intellisense. If both are
> specified, the configuration defined last will win.

> `{ P: { demandThisOption: true }}` can be read as `P`, with truth values
> denoting existence.

`demandThisOption` enables checks to ensure the specified argument is given.
This is equivalent to `demandOption` from vanilla yargs. For example:

```jsonc
{
  "x": { "demandThisOption": true }, // ◄ Disallows ∅, y
  "y": { "demandThisOption": false }
}
```

This configuration will trigger a check to ensure that `‑x` is given.

##### `demandThisOptionOr`

> `demandThisOptionOr` is a superset of vanilla yargs's [`demandOption`][16].

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

> `demandThisOptionXor` is a superset of vanilla yargs's [`demandOption`][16] +
> [`conflicts`][15].

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
  // ▼ Demands x or y == 'one' or z
  "x": { "demandThisOptionXor": [{ "y": "one" }, "z"] },
  "y": {},
  "z": {}
}
```

This configuration allows the following arguments: `‑x`, `‑y=one`, `‑z`,
`‑x ‑y=...`, `‑y=... ‑z`; and disallows: no arguments (`∅`), `‑y=...`,
`‑x ‑y=one`, `‑xz`, `‑y=one ‑z`, `‑xz ‑y=...`.

##### `subOptionOf`

<!-- TODO: fix me -->

TODO (reconfigure option depending on values of other options; overlap = merged
with latest definitions winning)

<!-- TODO: fix me -->

TODO: happens on second parse, not first (normal "fallback" config happens on
first)

```javascript
{
  version: {
    subOptionOf: {
      language: [
        {
          when: (languageArgv) => boolean === true,
          update:
            ((oldConfig, fullArgv) => {
              return newConfig;
            }) || newConfig
        },
        {
          when: (languageArgv) => anotherBoolean === true,
          update:
            ((oldConfig, fullArgv) => {
              return newConfig;
            }) || newConfig
        }
      ];
    }
  }
}
```

#### `requires` versus `demandThisOptionIf`

The utility of [`demandThisOptionIf`][8] versus [`requires`][6] (and
[`demandThisOptionOr`][10]) becomes apparent when checking against multiple
arguments. For example:

```jsonc
{
  // ▼ Demands y == 'one' and z if x is given
  "x": { "requires": [{ "y": "one" }, "z"] },
  "y": {},
  "z": { "requires": "y" } // ◄ Demands y if z is given
}
```

Versus:

```jsonc
{
  // ▼ Demands x or y == 'one' or z
  "x": { "demandThisOptionOr": [{ "y": "one" }, "z"] },
  "y": {},
  // ▼ Demands z or y
  "z": { "demandThisOptionOr": "y" }
}
```

Versus:

```jsonc
{
  "x": {},
  "y": { "demandThisOptionIf": "x" }, // ◄ Demands y if x is given
  // ▼ Demands z if x or y == one are given
  "z": { "demandThisOptionIf": ["x", { "y": "one" }] }
}
```

The first `requires` configuration allows the following arguments: no arguments
(`∅`), `‑y=...`, `‑y=... ‑z`, `‑xz ‑y=one`; and disallows: `‑x`, `‑z`,
`‑x ‑y=...`, `‑xz`, `‑xz ‑y=...`.

The second `demandThisOptionOr` configuration allows the following arguments:
`‑y=one`, `‑z`, `‑x ‑y=...`, `‑xz`, `‑y=... ‑z`, `‑xz ‑y=...`; and disallows: no
arguments (`∅`), `‑x`, `‑y=...`.

While the final `demandThisOptionIf` configuration allows the following
arguments: no arguments (`∅`), `‑y=...`, `‑z`, `‑y=... ‑z`, `‑xz ‑y=one`; and
disallows: `‑x`, `‑y=one`, `‑x ‑y=...`, `‑xz`, `‑xz ‑y=...`.

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

To this end, the following [yargs API functions][17] are soft-disabled via
intellisense:

- `option`
- `options`

However, no attempt is made by BFE to restrict your use of the yargs API at
runtime. Therefore, using yargs's API to work around these artificial
limitations, e.g. in your command's [`builder`][18] function or via the
[`configureExecutionPrologue`][19] hook, will result in **undefined behavior**.

### Black Flag versus Black Flag Extensions

The goal of [Black Flag (@black-flag/core)][20] is to be as close to a drop-in
replacement as possible for vanilla yargs, specifically for users of
[`yargs::commandDir()`][21]. This means Black Flag must go out of its way to
maintain 1:1 parity with the vanilla yargs API ([with a few minor
exceptions][22]).

As a consequence, yargs's imperative nature tends to leak through Black Flag's
abstraction at certain points, such as with [the `blackFlag` parameter of the
`builder` export][18]. **This is a good thing!** Since we want access to all of
yargs's killer features without Black Flag getting in the way.

However, this comes with costs. For one, the yargs's API has suffered from a bit
of feature creep over the years. A result of this is a rigid API [with][23]
[an][24] [abundance][25] [of][26] [footguns][27] and an [inability][28] to
[address][29] them without introducing [massively][30] [breaking][31]
[changes][32].

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
[12]: #subOptionOf
[13]: https://yargs.js.org/docs#implies
[14]: https://github.com/yargs/yargs/issues/1322#issuecomment-1538709884
[15]: https://yargs.js.org/docs#conflicts
[16]: https://yargs.js.org/docs#demandOption
[17]: https://yargs.js.org/docs#api-reference
[18]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/Configuration.md#builder
[19]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/ConfigureExecutionPrologue.md
[20]: https://npm.im/@black-flag/core
[21]: https://yargs.js.org/docs#api-reference-commanddirdirectory-opts
[22]:
  https://github.com/Xunnamius/black-flag?tab=readme-ov-file#differences-between-black-flag-and-yargs
[23]: https://github.com/yargs/yargs/issues/1323
[24]: https://github.com/yargs/yargs/issues/1442
[25]: https://github.com/yargs/yargs/issues/2340
[26]: https://github.com/yargs/yargs/issues/1322
[27]: https://github.com/yargs/yargs/issues/2089
[28]: https://github.com/yargs/yargs/issues/1975
[29]: https://github.com/yargs/yargs-parser/issues/412
[30]: https://github.com/yargs/yargs/issues/1680
[31]: https://github.com/yargs/yargs/issues/1599
[32]: https://github.com/yargs/yargs/issues/1611
