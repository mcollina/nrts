# borp

Borp is a typescript-aware test runner for `node:test`.
It also support code coverage via [c8](http://npm.im/c8).

Borp is self-hosted, i.e. Borp runs its own tests.

## Install

```bash
npm i borp --save-dev
```

## Usage

```bash
borp --coverage
```

Borp will automatically run all tests files matching `*.test.{js|ts}`.

### Example project setup

```
.
├── src
│   ├── lib
│   │   └── add.ts
│   └── test
│       └── add.test.ts
└── tsconfig.json

```

As an example, consider having a `src/lib/add.ts` file  

```typescript
export function add (x: number, y: number): number {
  return x + y
}
```

and a `src/test/add.test.ts` file:

```typescript
import { test } from 'node:test'
import { add } from '../lib/add.js'
import { strictEqual } from 'node:assert'

test('add', () => {
  strictEqual(add(1, 2), 3)
})
```

and the following `tsconfig.json`:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "outDir": "dist",
    "sourceMap": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "strict": true,
    "resolveJsonModule": true,
    "removeComments": true,
    "newLine": "lf",
    "noUnusedLocals": true,
    "noFallthroughCasesInSwitch": true,
    "isolatedModules": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "lib": [
      "ESNext"
    ],
    "incremental": true
  }
}
```

Note the use of `incremental: true`, which speed up compilation massively.

## Options

* `--coverage` or `-C`, enables code coverage
* `--only` or `-o`, only run `node:test` with the `only` option set  
* `--watch` or `-w`, re-run tests on changes
* `--timeout` or `-t`, timeouts the tests after a given time; default is 30000 ms
* `--coverage-exclude` or `-X`, a list of comma-separated patterns to exclude from the coverage report. All tests files are ignored by default.
* `--ignore` or `-i`, ignore a glob pattern, and not look for tests there
* `--expose-gc`, exposes the gc() function to tests
* `--pattern` or `-p`, run tests matching the given glob pattern
* `--reporter` or `-r`, set up a reporter, use a colon to set a file destination. Default: `spec`.
* `--no-typescript` or `-T`, disable automatic TypeScript compilation if `tsconfig.json` is found.

## Reporters

Here are the available reporters:

* `md`: creates a markdown table, useful for setting up a Summary in your GitHub Action
* `gh`: emits `::error` workflow commands for GitHub Actions to show inlined error. Enabled by default when running on GHA.
* `tap`: outputs the test results in the TAP format.
* `spec`: outputs the test results in a human-readable format.
* `dot`: outputs the test results in a compact format, where each passing test is represented by a ., and each failing test is represented by a X.
* `junit`: outputs test results in a jUnit XML format

## GitHub Action Summary

The following will automatically show the summary of the test run in the summary page of GitHub Actions.

```yaml
name: ci

on:
  push:
    paths-ignore:
      - 'docs/**'
      - '*.md'
  pull_request:
    paths-ignore:
      - 'docs/**'
      - '*.md'
      
jobs:
  test:
    runs-on: ${{matrix.os}}

    strategy:
      matrix:
        node-version: [18.x, 20.x, 21.x]
        os: [ubuntu-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install
        run: |
          npm install

      - name: Lint
        run: |
          npm run lint

      - name: Run tests
        run: |
          npm run unit -- --reporter spec --reporter md:report.md

      - name: Upload report
        shell: bash
        if: success() || failure() 
        run: |
          cat report.md >> "$GITHUB_STEP_SUMMARY"
```

## License

MIT
