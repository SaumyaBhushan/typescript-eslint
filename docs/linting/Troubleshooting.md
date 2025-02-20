---
id: troubleshooting
title: Troubleshooting & FAQs
---

## I am using a rule from ESLint core, and it doesn't work correctly with TypeScript code

This happens because TypeScript adds new features that ESLint doesn't know about.

The first step is to [check our list of "extension" rules here](https://typescript-eslint.io/rules/#extension-rules).
An extension rule is a rule which extends the base ESLint rules to support TypeScript syntax.
If you find it in there, give it a go to see if it works for you.
You can configure it by disabling the base rule, and turning on the extension rule.
Here's an example with the `semi` rule:

```json
{
  "rules": {
    "semi": "off",
    "@typescript-eslint/semi": "error"
  }
}
```

If you don't find an existing extension rule, or the extension rule doesn't work for your case, then you can go ahead and check our issues.
[The contributing guide outlines the best way to raise an issue](https://github.com/typescript-eslint/typescript-eslint/blob/main/CONTRIBUTING.md#raising-issues).

> We release a new version our tooling every week.
> _Please_ ensure that you [check our the latest list of "extension" rules](https://typescript-eslint.io/rules/#extension-rules) **_before_** filing an issue.

## I get errors telling me "ESLint was configured to run ... However, that TSConfig does not / none of those TSConfigs include this file"

### Fixing the Error

- If you **do not** want to lint the file:
  - Use [one of the options ESLint offers](https://eslint.org/docs/latest/user-guide/configuring/ignoring-code) to ignore files, namely a `.eslintignore` file, or `ignorePatterns` config.
- If you **do** want to lint the file:
  - If you **do not** want to lint the file with [type-aware linting](./Typed_Linting.md):
    - Use [ESLint's `overrides` configuration](https://eslint.org/docs/latest/user-guide/configuring/configuration-files#configuration-based-on-glob-patterns) to configure the file to not be parsed with type information.
      - A popular setup is to omit the above additions from top-level configuration and only apply them to TypeScript files via an override.
      - Alternatively, you can add `parserOptions: { project: null }` to an override for the files you wish to exclude. Note that `{ project: undefined }` will not work.
  - If you **do** want to lint the file with [type-aware linting](./Typed_Linting.md):
    - Check the `include` option of each of the tsconfigs that you provide to `parserOptions.project` - you must ensure that all files match an `include` glob, or else our tooling will not be able to find it.
    - If your file shouldn't be a part of one of your existing tsconfigs (for example, it is a script/tool local to the repo), then consider creating a new tsconfig (we advise calling it `tsconfig.eslint.json`) in your project root which lists this file in its `include`. For an example of this, you can check out the configuration we use in this repo:
      - [`tsconfig.eslint.json`](https://github.com/typescript-eslint/typescript-eslint/blob/main/tsconfig.eslint.json)
      - [`.eslintrc.js`](https://github.com/typescript-eslint/typescript-eslint/blob/main/.eslintrc.js)

### More Details

This error may appear from the combination of two things:

- The ESLint configuration for the source file specifies at least one TSConfig file in `parserOptions.project`
- None of those TSConfig files includes the source file being linted
  - Note that files with the same name and different extension may not be recognized by TypeScript: see [`parserOptions.project` docs](https://github.com/typescript-eslint/typescript-eslint/tree/main/packages/parser#parseroptionsproject)

When TSConfig files are specified for parsing a source file, `@typescript-eslint/parser` will use the first TSConfig that is able to include that source file (per [aka.ms/tsconfig#include](https://www.typescriptlang.org/tsconfig#include)) to generate type information.
However, if no specified TSConfig includes the source file, the parser won't be able to generate type information.

This error most commonly happens on config files or similar that are not included in their project TSConfig(s).
For example, many projects have files like:

- An `.eslintrc.cjs` with `parserOptions.project: ["./tsconfig.json"]`
- A `tsconfig.json` with `include: ["src"]`

In that case, viewing the `.eslintrc.cjs` in an IDE with the ESLint extension will show the error notice that the file couldn't be linted because it isn't included in `tsconfig.json`.

See our docs on [type aware linting](./Typed_Linting.md) for more information.

## I get errors telling me "The file must be included in at least one of the projects provided"

You're using an outdated version of `@typescript-eslint/parser`.
Update to the latest version to see a more informative version of this error message, explained [above](#i-get-errors-telling-me-eslint-was-configured-to-run--however-that-tsconfig-does-not--none-of-those-tsconfigs-include-this-file 'backlink to I get errors telling me ESLint was configured to run ...').

## I use a framework (like Vue) that requires custom file extensions, and I get errors like "You should add `parserOptions.extraFileExtensions` to your config"

You can use `parserOptions.extraFileExtensions` to specify an array of non-TypeScript extensions to allow, for example:

```js title=".eslintrc.js"
module.exports = {
  parserOptions: {
    tsconfigRootDir: __dirname,
    project: ['./tsconfig.json'],
    // Add this line
    extraFileExtensions: ['.vue'],
  },
};
```

## I am running into errors when parsing TypeScript in my .vue files

If you are running into issues parsing .vue files, it might be because parsers like [`vue-eslint-parser`](https://www.npmjs.com/package/vue-eslint-parser) are required to parse `.vue` files. In this case you can move `@typescript-eslint/parser` inside `parserOptions` and use `vue-eslint-parser` as the top level parser.

```diff
- "parser": "@typescript-eslint/parser",
+ "parser": "vue-eslint-parser",
  "parserOptions": {
+     "parser": "@typescript-eslint/parser",
      "sourceType": "module"
  }
```

The `parserOptions.parser` option can also specify an object to specify multiple parsers. See the [`vue-eslint-parser` usage guide](https://eslint.vuejs.org/user-guide/#usage) for more details.

## One of my lint rules isn't working correctly on a pure JavaScript file

This is to be expected - ESLint rules do not check file extensions on purpose, as it causes issues in environments that use non-standard extensions (for example, a `.vue` and a `.md` file can both contain TypeScript code to be linted).

If you have some pure JavaScript code that you do not want to apply certain lint rules to, then you can use [ESLint's `overrides` configuration](https://eslint.org/docs/user-guide/configuring#configuration-based-on-glob-patterns) to turn off certain rules, or even change the parser based on glob patterns.

## TypeScript should be installed locally

Make sure that you have installed TypeScript locally i.e. by using `npm install typescript`, not `npm install -g typescript`,
or by using `yarn add typescript`, not `yarn global add typescript`.
See [#2041](https://github.com/typescript-eslint/typescript-eslint/issues/2041) for more information.

## How can I ban `<specific language feature>`?

ESLint core contains the rule [`no-restricted-syntax`](https://eslint.org/docs/rules/no-restricted-syntax).
This generic rule allows you to specify a [selector](https://eslint.org/docs/developer-guide/selectors) for the code you want to ban, along with a custom error message.

You can use an AST visualization tool such as [TypeScript ESLint playground](https://typescript-eslint.io/play#showAST=es) > _Options_ > _AST Explorer_ on its left sidebar by selecting _ESTree_ to help in figuring out the structure of the AST that you want to ban.

For example, you can ban enums (or some variation of) using one of the following configs:

```jsonc
{
  "rules": {
    "no-restricted-syntax": [
      "error",
      // ban all enums
      {
        "selector": "TSEnumDeclaration",
        "message": "My reason for not using any enums at all"
      },

      // ban just const enums
      {
        "selector": "TSEnumDeclaration[const=true]",
        "message": "My reason for not using const enums"
      },

      // ban just non-const enums
      {
        "selector": "TSEnumDeclaration:not([const=true])",
        "message": "My reason for not using non-const enums"
      }
    ]
  }
}
```

## Why don't I see TypeScript errors in my ESLint output?

TypeScript's compiler (or whatever your build chain may be) is specifically designed and built to validate the correctness of your codebase.
Our tooling does not reproduce the errors that TypeScript provides, because doing so would slow down the lint run [1], and duplicate the errors that TypeScript already outputs for you.

Instead, our tooling exists to **_augment_** TypeScript's built in checks with lint rules that consume the type information in new ways beyond just verifying the runtime correctness of your code.

[1] - TypeScript computes type information lazily, so us asking for the errors it would produce from the compiler would take an _additional_ ~100ms per file.
This doesn't sound like a lot, but depending on the size of your codebase, it can easily add up to between several seconds to several minutes to a lint run.

## I get errors from the `no-undef` rule about global variables not being defined, even though there are no TypeScript errors

The `no-undef` lint rule does not use TypeScript to determine the global variables that exist - instead, it relies upon ESLint's configuration.

We strongly recommend that you do not use the `no-undef` lint rule on TypeScript projects.
The checks it provides are already provided by TypeScript without the need for configuration - TypeScript just does this significantly better.

As of our v4.0.0 release, this also applies to types.
If you use global types from a 3rd party package (i.e. anything from an `@types` package), then you will have to configure ESLint appropriately to define these global types.
For example; the `JSX` namespace from `@types/react` is a global 3rd party type that you must define in your ESLint config.

Note, that for a mixed project including JavaScript and TypeScript, the `no-undef` rule (like any rule) can be turned off for TypeScript files alone by adding an `overrides` section to `.eslintrc.cjs`:

```js title=".eslintrc.cjs"
module.exports = {
  // ... the rest of your config ...
  overrides: [
    {
      files: ['*.ts', '*.mts', '*.cts', '*.tsx'],
      rules: {
        'no-undef': 'off',
      },
    },
  ],
};
```

If you choose to leave on the ESLint `no-undef` lint rule, you can [manually define the set of allowed `globals` in your ESLint config](https://eslint.org/docs/user-guide/configuring/language-options#specifying-globals), and/or you can use one of the [pre-defined environment (`env`) configurations](https://eslint.org/docs/user-guide/configuring/language-options#specifying-environments).

## How do I check to see what versions are installed?

If you have multiple versions of our tooling, it can cause various bugs for you.
This is because ESLint may load a different version each run depending on how you run it - leading to inconsistent lint results.

Installing our tooling in the root of your project does not mean that only one version is installed.
One or more of your dependencies may have its own dependency on our tooling, meaning `npm`/`yarn` will additionally install that version for use by that package.
For example, `react-scripts` (part of `create-react-app`) has a dependency on our tooling.

You can check what versions are installed in your project using the following commands:

```bash npm2yarn
npm list @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

If you see more than one version installed, then you will have to either use [yarn resolutions](https://classic.yarnpkg.com/en/docs/selective-version-resolutions) to force a single version, or you will have to downgrade your root versions to match the dependency versions.

**The best course of action in this case is to wait until your dependency releases a new version with support for our latest versions.**

## How can I specify a TypeScript version / `parserOptions.typescriptLocation`?

You can't, and you don't want to.

You should use the same version of TypeScript for linting as the rest of your project.
TypeScript versions often have slight differences in edge cases that can cause contradictory information between typescript-eslint rules and editor information.
For example:

- `@typescript-eslint/strict-boolean-expressions` might be operating with TypeScript version _X_ and think a variable is `string[] | undefined`
- TypeScript itself might be on version _X+1-beta_ and think the variable is `string[]`

See [this issue comment](https://github.com/typescript-eslint/typescript-eslint/issues/4102#issuecomment-963265514) for more details.

## Changes to one file are not reflected when linting other files in my IDE

> tl;dr: Restart your ESLint server to force an update.

ESLint currently does not have any way of telling parsers such as ours when an arbitrary file is changed on disk.
That means if you change file A that is imported by file B, it won't update lint caches for file B -- even if file B's text contents have changed.
Sometimes the only solution is to restart your ESLint editor extension altogether.

See [this issue comment](https://github.com/typescript-eslint/typescript-eslint/issues/5845#issuecomment-1283248238 'GitHub issue 5845, comment 1283248238: details on ESLint cross-file caching') for more information.

:::tip
[VS Code's ESLint extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) provides an `ESLint: Restart ESLint Server` action.
:::

### I get `no-unsafe-*` complaints for cross-file changes

See [Changes to one file are not reflected in linting other files in my IDE](#changes-to-one-file-are-not-reflected-in-linting-other-files-in-my-ide).
Rules such as [`no-unsafe-argument`](https://typescript-eslint.io/rules/no-unsafe-argument), [`no-unsafe-assignment`](https://typescript-eslint.io/rules/no-unsafe-assignment), and [`no-unsafe-call`](https://typescript-eslint.io/rules/no-unsafe-call) are often impacted.

## My linting feels really slow

As mentioned in the [type-aware linting doc](./Typed_Linting.md), if you're using type-aware linting, your lint times should be roughly the same as your build times.

If you're experiencing times much slower than that, then there are a few common culprits.

### Wide includes in your `tsconfig`

When using type-aware linting, you provide us with one or more tsconfigs.
We then will pre-parse all files so that full and complete type information is available.

If you provide very wide globs in your `include` (such as `**/*`), it can cause many more files than you expect to be included in this pre-parse.
Additionally, if you provide no `include` in your tsconfig, then it is the same as providing the widest glob.

Wide globs can cause TypeScript to parse things like build artifacts, which can heavily impact performance.
Always ensure you provide globs targeted at the folders you are specifically wanting to lint.

### Wide includes in your ESLint options

Specifying `tsconfig.json` paths in your ESLint commands is also likely to cause much more disk IO than expected.
Instead of globs that use `**` to recursively check all folders, prefer paths that use a single `*` at a time.

```diff
- eslint --parser-options project:./**/tsconfig.json
+ eslint --parser-options project:./packages/*/tsconfig.json
```

See [Glob pattern in parser's option "project" slows down linting](https://github.com/typescript-eslint/typescript-eslint/issues/2611) for more details.

### `eslint-plugin-prettier`

This plugin surfaces prettier formatting problems at lint time, helping to ensure your code is always formatted.
However this comes at a quite a large cost - in order to figure out if there is a difference, it has to do a prettier format on every file being linted.
This means that each file will be parsed twice - once by ESLint, and once by Prettier.
This can add up for large codebases.

Instead of using this plugin, we recommend using prettier's `--list-different` flag to detect if a file has not been correctly formatted.
For example, our CI is setup to run the following command automatically, which blocks PRs that have not been formatted:

```bash npm2yarn
npm run prettier --list-different \"./**/*.{ts,mts,cts,tsx,js,mjs,cjs,jsx,json,md,css}\"
```

### `eslint-plugin-import`

This is another great plugin that we use ourselves in this project.
However there are a few rules which can cause your lints to be really slow, because they cause the plugin to do its own parsing, and file tracking.
This double parsing adds up for large codebases.

There are many rules that do single file static analysis, but we provide the following recommendations.

We recommend you do not use the following rules, as TypeScript provides the same checks as part of standard type checking:

- `import/named`
- `import/namespace`
- `import/default`
- `import/no-named-as-default-member`

The following rules do not have equivalent checks in TypeScript, so we recommend that you only run them at CI/push time, to lessen the local performance burden.

- `import/no-named-as-default`
- `import/no-cycle`
- `import/no-unused-modules`
- `import/no-deprecated`

### The `indent` / `@typescript-eslint/indent` rules

This rule helps ensure your codebase follows a consistent indentation pattern.
However this involves a _lot_ of computations across every single token in a file.
Across a large codebase, these can add up, and severely impact performance.

We recommend not using this rule, and instead using a tool like [`prettier`](https://www.npmjs.com/package/prettier) to enforce a standardized formatting.
