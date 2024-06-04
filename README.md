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

<!-- TODO: fix me -->

TODO

#### Automatic Grouping of Related Options

<!-- TODO: fix me -->

Automatic grouping of related options. To support this functionality,
`blackFlag.options(...)` and `blackFlag.option(...)` are no longer callable from
within commands' `builder` functions. To add options to a command, they must be
described declaratively, i.e. via returning an options object from your
`builder` function.

This feature can be disabled by passing `{ disableAutomaticGrouping: true }` to
`withBuilderExtensions`.

#### New Option Configuration Keys

<!-- TODO: fix me -->

TODO

##### `requires`

> `requires` is a superset of and replacement for vanilla yargs's
> [`implies`][4]. However, `implies` is not disallowed by intellisense. If both
> are specified, they will both be considered by Black Flag and yargs.

> `requires` is the inverse of [`conflicts`][5].

This configuration will trigger a check to ensure the specified arguments, or
argument-value pairs, are given conditioned on the existence of another
argument. For example:

```json
{
  "x": { "requires": "y" }, // ◄ Disallows x without y
  "y": {}
}
```

This configuration will trigger a check to ensure that `-y` is given whenever
`-x` is given.

For example:

```json
{
  // ▼ Disallows x unless y == 'one' and z is given
  "x": { "requires": [{ "y": "one" }, "z"] },
  "y": {},
  "z": { "requires": "y" } // ◄ Disallows z unless y is given
}
```

This configuration allows the following arguments: no arguments (`∅`), `-y=...`,
`-y=... z`, `-x -y=one`, `-xz -y=one`; and disallows: `-x`, `-z`, `-x -y=...`,
`-xz`.

##### `conflicts`

> `conflicts` is a superset of vanilla yargs's [`conflicts`][6].

> `conflicts` is the inverse of [`requires`][7].

This configuration will trigger a check to ensure the specified arguments, or
argument-value pairs, are given conditioned on the existence of another
argument. For example:

```json
{
  "x": { "conflicts": "y" }, // ◄ Disallows y if x is given
  "y": {}
}
```

This configuration will trigger a check to ensure that `-y` is never given
whenever `-x` is given.

For example:

```json
{
  // ▼ Disallows y == 'one' or z if x is given
  "x": { "conflicts": [{ "y": "one" }, "z"] },
  "y": {},
  "z": { "conflicts": "y" } // ◄ Disallows y if z is given
}
```

This configuration allows the following arguments: no arguments (`∅`), `-y=...`,
`-x`, `-z`, `-x -y=...`; and disallows: `-y=... z`, `-x -y=one`, `-xz -y=one`,
`-xz`.

##### `demandThisOption`

> `demandThisOption` is an alias of vanilla yargs's [`demandOption`][8].
> However, `demandOption` is not disallowed by intellisense. If both are
> specified, the configuration defined last will win.

This configuration will trigger a check to ensure the specified argument is
given. This is equivalent to `demandOption` from vanilla yargs. For example:

```json
{
  "x": { "demandThisOption": true }, // ◄ Disallows ∅, y
  "y": { "demandThisOption": false }
}
```

This configuration will trigger a check to ensure that `-x` is given.

##### `demandThisOptionOr`

> `demandThisOptionOr` is a superset of vanilla yargs's [`demandOption`][8].

`demandThisOptionOr` enables non-optional inclusive disjunction checks per
group. Put another way, `demandThisOptionOr` enforces a "logical or" relation
within groups of required options. For example:

<!-- TODO: fix me -->

```json
{
  "x": { "demandOption": ["y", "z"] },
  "y": { "demandOption": ["x", "z"] },
  "z": { "demandOption": ["x", "y"] }
}
```

This configuration will trigger a check to ensure _at least one_ of `x`, `y`, or
`z` is given. Providing such a value for `demandOption` on one option but not
mirroring that appropriate configuration to the other listed options will result
in an assertion failure. To make it clear that you intend for an inclusive
disjunction, `demandOption: [...]` is aliased to `demandThisOptionOr: [...]`,
which is the same as demandOption\` except it does not accept a boolean value.
Note that an option can be a member of an exclusivity group and an inclusive
disjunction group simultaneously. For example:

```json
{
  "x": {
    "demandThisOptionOr": ["y"],
    "demandThisOptionXor": ["z"]
  },
  "y": { "demandThisOptionOr": ["x"] },
  "z": { "demandThisOptionXor": ["x"] }
}
```

This configuration allows the following argument combinations: `x`, `x` and `y`,
`y` and `z`. Conversely, said configuration disallows the following: no
arguments, `y`, `z`, `x` and `z`, `x` and `y` and `z`. 3. Expanded `requires`
(formerly `implies`) and `conflicts` checks, which allow asserting conflicting
_values_ on keys in addition to existence. For example:

```json
{
  x: { requires: ["y", { z: "some-val" }] },
  y: { conflicts: { "w-2": true } },
  z: { choices: ["some-val", "some-other-val"] }
  "w-2": { boolean: true }
}
```

This configuration will trigger a check to ensure the following:

- Whenever `x` is given, `y` and `z=some-val` must also be given.
- Whenever `y` is given, `w-2` must not be given, or, if it is, must be as
  `--w-2=false` or `--no-w-2`.
- Whenever `--w-2=true` is given, `y` cannot be given, which means `x` cannot be
  given.
- Whenever `--z=some-other-val` is given, or if `z` is not given, `x` cannot be
  given. Despite these checks, all of the options above are still optional in
  that the command will still run even if `argv` is empty.

##### `demandThisOptionXor`

> `demandThisOptionXor` is a superset of vanilla yargs's [`demandOption`][8] +
> [`conflicts`][6].

`demandThisOptionXor` enables non-optional exclusive disjunction checks per
exclusivity group. Put another way, `demandThisOptionXor` enforces mutual
exclusivity within groups of required options. For example:

```json
{
  "x": { "demandThisOptionXor": ["y"] }, // ◄ Disallows ∅, z, w, xy, xyw, xyz, xyzw
  "y": { "demandThisOptionXor": ["x"] }, // ◄ Disallows ∅, z, w, xy, xyw, xyz, xyzw
  "z": { "demandThisOptionXor": ["w"] }, // ◄ Disallows ∅, x, y, zw, xzw, yzw, xyzw
  "w": { "demandThisOptionXor": ["z"] } // ◄ Disallows ∅, x, y, zw, xzw, yzw, xyzw
}
```

This configuration will trigger a check to ensure _exactly one_ of `-x` or `-y`
is given, and _exactly one_ of `-z` or `-w` is given. In other words, this
configuration allows the following arguments: `-xz`, `-xw`, `-yz`, `-yw`; and
disallows: no arguments (`∅`), `-x`, `-y`, `-z`, `-w`, `-xy`, `-zw`, `-xyz`,
`-xyw`, `-xzw`, `-yzw`, `-xyzw`.

In the interest of readability, is recommended to mirror conflicts across
options within the same group when your intent is mutual exclusivity. The
previous example demonstrates this. However, unlike `demandThisOptionOr`, this
mirroring is _not_ required.

Still, it is useful to note how, in that same example, the first
`demandThisOptionXor` mirrors the second, and the third `demandThisOptionXor`
mirrors the fourth. To Black Flag, they are redundant. Therefore, the following
is equivalent.

```json
{
  "x": { "demandThisOptionXor": ["y"] }, // ◄ Disallows ∅, z, w, xy, xyw, xyz, xyzw
  "y": {},
  "z": { "demandThisOptionXor": ["w"] }, // ◄ Disallows ∅, x, y, zw, xzw, yzw, xyzw
  "w": {}
}
```

Here's a more complex albeit impractical example:

```json
{
  "x": { "demandThisOptionXor": ["y", "z"] }, // ◄ Disallows ∅, xy, xz, yz, xyz
  "y": { "demandThisOptionXor": ["x"] }, // ◄ Disallows ∅, z, xy, xyz
  "z": { "demandThisOptionXor": ["y"] } // ◄ Disallows ∅, x, yz, xyz
}
```

This configuration allows the following arguments: `-y`; and disallows: no
arguments (`∅`), `-x`, `-z`, `-xy`, `-xz`, `-yz`, `-xyz`.

What's more, like its sibling configurations keys, `demandThisOptionXor` also
supports demanding argument-value pairs. For example:

```json
{
  "x": { "demandThisOptionXor": { "y": "one" } },
  "y": {}
}
```

This configuration allows the following arguments: `-x`, `-y=one`; and
disallows: no arguments (`∅`), `-y=...`.

##### `demandThisOptionIf`

<!-- TODO: fix me -->

TODO

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

#### Support for `default` + `conflicts` and `default` + `requires`

<!-- TODO: fix me -->

TODO: footgun avoided for free

#### Impossible Configurations

Note that **there are no sanity checks performed to prevent demands that are
impossible to fulfil**, so care must be taken not to ask for something insane.

For example, the following configurations are impossible to fulfil:

```json
{
  "x": { "requires": "y" },
  "y": { "conflicts": "x" }
}
```

```json
{
  "x": { "requires": "y", "demandThisOptionXor": "y" },
  "y": {}
}
```

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
        // ▼ Inverse of { conflicts: { target: 'vercel' }} (equivalent in this example)
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
object][9] from your command's `builder` rather than imperatively invoking the
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

To this end, the following [yargs API functions][10] are soft-disabled via
intellisense:

- `option`
- `options`

However, no attempt is made by BFE to restrict your use of the yargs API at
runtime. Therefore, using yargs's API to work around these artificial
limitations, e.g. in your command's [`builder`][11] function or via the
[`configureExecutionPrologue`][12] hook, will result in **undefined behavior**.

### Black Flag versus Black Flag Extensions

The goal of [Black Flag (@black-flag/core)][13] is to be as close to a drop-in
replacement as possible for vanilla yargs, specifically for users of
[`yargs::commandDir()`][14]. This means Black Flag must go out of its way to
maintain 1:1 parity with the vanilla yargs API ([with a few minor
exceptions][15]).

This means yargs's imperative nature tends to leak through Black Flag's
abstraction at certain points, such as with [the `blackFlag` parameter of the
`builder` export][11]. **This is a good thing!** Since we want access to all of
yargs's killer features without Black Flag getting in the way.

However, this comes with costs. For one, the yargs's API has suffered from a bit
of feature creep over the years. A result of this is a rigid API [with][16]
[an][17] [abundance][18] [of][19] [footguns][20] and an [inability][21] to
[address][22] them without introducing [massively][23] [breaking][24]
[changes][25].

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
[4]: https://yargs.js.org/docs#implies
[5]: #conflicts
[6]: https://yargs.js.org/docs#conflicts
[7]: #requires
[8]: https://yargs.js.org/docs#demandOption
[9]: https://yargs.js.org/docs#api-reference-optionskey-opt
[10]: https://yargs.js.org/docs#api-reference
[11]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/Configuration.md#builder
[12]:
  https://github.com/Xunnamius/black-flag/blob/main/docs/index/type-aliases/ConfigureExecutionPrologue.md
[13]: https://npm.im/@black-flag/core
[14]: https://yargs.js.org/docs#api-reference-commanddirdirectory-opts
[15]:
  https://github.com/Xunnamius/black-flag?tab=readme-ov-file#differences-between-black-flag-and-yargs
[16]: https://github.com/yargs/yargs/issues/1323
[17]: https://github.com/yargs/yargs/issues/1442
[18]: https://github.com/yargs/yargs/issues/2340
[19]: https://github.com/yargs/yargs/issues/1322
[20]: https://github.com/yargs/yargs/issues/2089
[21]: https://github.com/yargs/yargs/issues/1975
[22]: https://github.com/yargs/yargs-parser/issues/412
[23]: https://github.com/yargs/yargs/issues/1680
[24]: https://github.com/yargs/yargs/issues/1599
[25]: https://github.com/yargs/yargs/issues/1611
