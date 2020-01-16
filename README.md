# Issue repro

`@fluentui/pkg2` depends on `@fluentui/pkg1`.

The packages' tsconfigs use project references.

They inherit from the root tsconfig which has a path config like this (to allow imports directly from the root of the package):

```
"baseUrl": ".",
"paths": {
  "@fluentui/*": ["packages/*/src"]
}
```

`packages/pkg2/src/index.ts` looks like this:

```ts
import { IThings } from '@fluentui/pkg1';

export function fn4() {
  const a: IThings = { thing1: { a: 'b' } };
  return a.thing1;
}
```

Its declaration file `packages/pkg2/lib/src/index.d.ts` is like this:

```ts
export declare function fn4(): import('../../pkg1/src').IThing;
```

**Why is the import relative instead of using the name of the package?**

## Variants

**Doesn't repro without `paths` config.**

Same behavior if `-b` is added to the build command.

Same behavior if using `rootDir` instead of `include`, just different output directory structure.

Same behavior if the config inheritance is removed and `paths` config is moved to each project.

Without project references, you end up with a directory structure like this, which I also don't understand, but is not the same issue:

```
pkg2/
  lib/
    pkg1/src/<files>
    pkg2/src/<files>
```

## Workaround

Remove `baseUrl` and `paths` from tsconfig.

In each project's package.json, set `"main": "src/index.ts"`. (In a real project, add a pre-publish step to correct this to point to `lib/index.js`.)

See the most recent commit in the [`inferred-relative-imports-workaround` branch](https://github.com/ecraig12345/learn-a/tree/inferred-relative-imports-workaround) for full details.
