---
title: 'Code Reusability: Export and Import'
permalink: /schema/advanced/export-import
category: Data Schema
subCategory: Advanced
menuOrder: 3
redirect_from:
  - /export-import
  - /recipes/export-import
---

<InfoBox>

This functionality only works with schemas written in JavaScript, not YAML.

</InfoBox>

In Cube, your data schema is code, and code is much easier to manage when it
is in small, digestible chunks. It is best practice to keep files small and
containing only relevant and non-duplicated code. As your data schema grows,
maintaining and debugging is much easier with a well-organized codebase.

Cube schemas in JavaScript supports ES6-style [`export`][mdn-js-es6-export] and
[`import`][mdn-js-es6-import] statements, which allow writing code in one file
and sharing it, so it can be used by another file or files.

There are several typical use cases in Cube where it is considered best
practice to extract some variables or functions and then import it when needed.

## Managing constants

Quite often, you may want to have an array of test user IDs to exclude from your
analysis. You can define it once and `export` it like this:

```javascript
// in constants.js
export const TEST_USER_IDS = [1, 2, 3, 4, 5];
```

Later, you can `import` into the cube whenever needed:

```javascript
// in Users.js
import { TEST_USER_IDS } from './constants';

cube(`Users`, {
  sql: `...`,
  measures: {
    /* ... */
  },

  dimensions: {
    /* ... */
  },

  segments: {
    excludeTestUsers: {
      sql: `${CUBE}.id NOT IN (${TEST_USER_IDS.join(', ')})`,
    },
  },
});
```

## Helper functions

You can assign some commonly used SQL snippets to JavaScript functions. The
example below shows a parsing helper function, which can be used across any
number of cubes to correctly parse a date if it was stored as a string.

You can read more about working with [string time dimensions
here][ref-schema-string-time-dims].

```javascript
// in helpers.js
export const parseDateWithTimeZone = (column) =>
  `PARSE_TIMESTAMP('%F %T %Ez', ${column})`;
```

```javascript
// in events.js
import { parseDateWithTimeZone } from './helpers';

cube(`Events`, {
  sql: `SELECT * FROM events`,

  // ...

  dimensions: {
    date: {
      sql: `${parseDateWithTimeZone('date')}`,
      type: `time`,
    },
  },
});
```

## Import from parent directories

You may need to import from parent directories as Cube flattens nested directories.
The example below shows a correct way to import a helper function, which is located in a parent directory.

```shell
.
├── README.md
├── cube.js
├── package.json
└── schema
    ├── utils.js
    └── Sales
        └── Orders.js
```

```javascript
// in schema/Sales/Orders.js
import { capitalize } from './utils';
```

```javascript
// in schema/utils.js
export const capitalize = (s) => s.charAt(0).toUpperCase() + s.slice(1);
```

[mdn-js-es6-export]:
  https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export
[mdn-js-es6-import]:
  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import
[ref-schema-string-time-dims]:
  /schema/fundamentals/additional-concepts#string-time-dimensions
