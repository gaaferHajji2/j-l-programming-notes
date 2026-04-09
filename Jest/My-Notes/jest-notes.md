To use **Jest** with **Node.js** when your project uses **ES Modules (`"type": "module"`)** and `import` syntax, you need a slightly different setup than CommonJS because Jest was originally designed around `require()`.

Below is the clean modern setup senior Node.js developers usually use ✅🚀

---

# 1) package.json configuration

Set your project as ESM:

```json
{
  "type": "module",
  "scripts": {
    "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js"
  }
}
```

### Why `--experimental-vm-modules`?

Jest internally still needs this for stable ESM execution in Node.

---

# 2) Install Jest

```bash
npm install --save-dev jest
```

If using TypeScript too:

```bash
npm install --save-dev jest ts-jest @types/jest
```

---

# 3) Create jest.config.js (ESM version)

Because your project uses modules:

```js
export default {
  testEnvironment: 'node',
  transform: {},
};
```

Important:
Since you are in ESM mode, use:

```js
export default
```

NOT:

```js
module.exports =
```

---

# 4) Example project structure

```plaintext
project/
│── src/
│   └── math.js
│── tests/
│   └── math.test.js
│── package.json
│── jest.config.js
```

---

# 5) Example module using import/export

## src/math.js

```js
export function sum(a, b) {
  return a + b;
}
```

---

# 6) Write test using import

## tests/math.test.js

```js
import { sum } from '../src/math.js';

test('adds two numbers', () => {
  expect(sum(2, 3)).toBe(5);
});
```

---

# 7) Run tests

```bash
npm test
```

---

# 8) Mocking imports in ESM

In ESM, mocking is different:

```js
import { jest } from '@jest/globals';
```

Example:

```js
import { jest } from '@jest/globals';

const mockFn = jest.fn();
```

---

# 9) Mock external module

```js
jest.unstable_mockModule('./service.js', () => ({
  getData: jest.fn(() => 'mocked')
}));
```

Then:

```js
const service = await import('./service.js');
```

Because ESM mocking is async.

---

# 10) Recommended senior setup (Best practice) ⭐

Use separate config file:

## jest.config.mjs

```js
export default {
  testEnvironment: 'node',
  extensionsToTreatAsEsm: ['.js'],
  transform: {},
};
```

Then package:

```json
"scripts": {
  "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js"
}
```

---

# 11) Common problem: Cannot use import statement outside module

Fix:

Make sure:

✅ `"type": "module"` exists
✅ test files use `.js` or `.mjs`
✅ config uses `export default`

---

# 12) If using Babel (advanced production setup)

Install:

```bash
npm install --save-dev babel-jest @babel/preset-env
```

babel.config.json:

```json
{
  "presets": ["@babel/preset-env"]
}
```

Then Jest config:

```js
export default {
  transform: {
    '^.+\\.js$': 'babel-jest'
  }
};
```

This is useful when:

✅ decorators
✅ advanced syntax
✅ enterprise apps

---

# 13) Recommended production stack for Node ESM testing 🚀

Senior teams usually combine:

* Jest for unit testing
* SuperTest for API testing
* Sinon.js only if advanced spies needed
* native Node test runner for lightweight microservices

---

# 14) Better alternative in 2026 (important) ⭐

For pure ESM projects many teams now prefer:

Vitest

Because:

✅ Faster
✅ Native ESM
✅ Cleaner mocking
✅ Less config

Example:

```bash
npm install -D vitest
```

Then:

```json
"scripts": {
  "test": "vitest"
}
```

---

# 15) Senior advice ⚡

For modern Node projects:

* If pure Node ESM → prefer Vitest
* If enterprise legacy / ecosystem → Jest
* If microservices → combine Jest + integration tests

---

If you want, I can also generate a **full senior Node.js testing structure with:**

✅ Jest + ESM
✅ Mock services
✅ Repository tests
✅ Controller tests
✅ Integration tests
✅ Coverage setup
✅ CI pipeline

This is the production style used in large backend teams 🚀
