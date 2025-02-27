---
theme: seriph
background: https://cover.sli.dev
title: Let's talk about TypeScript
class: text-center
transition: slide-left
mdc: true
---

# 4 reasons you want to <br /> update TypeScript

Son Nguyen - SMB Engineering Dublin

<!-- Sample note here -->

---
layout: center
---

<h2 class="text-center mb-8">
📃 Table of content
</h2>

<Toc maxDepth="1" />

---
layout: cover
background: https://cover.sli.dev
---

<div class="m-0">Reason 1</div>

# Alignment with other <br /> Centro modules

---
layout: image-right
image: https://cover.sli.dev
---

Two versions are being used: <span v-mark.circle.red>4.2.3</span> (default) and <span v-mark.circle.red>5.4.5</span>

Modules are encouraged to upgrade

⚡ Performance gain
<br />
🪶 Minimal effort
<br />
📦 Better support for packages, e.g. Fluent UI

---

## How to upgrade

Find detailed instruction in this email

![TypeScript upgrade instruction for Centro modules](/upgrade-typescript-in-centro.png)

---
layout: cover
background: https://cover.sli.dev
---

<div class="m-0">Reason 2</div>

# Improved performance <br /> and code safety

---
layout: fact
---

<div class="flex justify-center gap-32 mx-auto">
  <div class="text-center max-w-[200px]">
    <div class="text-[80px]">54%</div>
    <div>Reduced compile time for Settings module (from 69 to 32 seconds)</div>
  </div>
  <div class="text-center max-w-[200px]">
    <div class="text-[80px]">38%</div>
    <div>Reduced type checking time for <code>host-hvc-harness</code></div>
  </div>
</div>

---
layout: cover
background: https://cover.sli.dev
---

<div class="m-0">Reason 3</div>

# Increased developer productivity

---

## Import Statement Completions

Make importing easier in VS Code.

<img src="/features-4-3-auto-import-statement.gif" alt="Visual Studio Code import auto completion" class="w-128 mx-auto">
![Visual Studio Code import auto completion](/features-4-3-auto-import-statement.gif)

---

## Inlay Hints

Add helpful type annotations and argument names.

![Visual Studio Code inlay hints](/features-4-4-vscode-inlay-hints.png)

---

## "Remove Unused Imports" and "Sort Imports" Commands for Editors

Use these commands to organize your imports.

![Visual Studio Code TypeScript imports actions](/features-4-9-vscode-imports-actions.png)

---
layout: cover
background: https://cover.sli.dev
---

<div class="m-0">Reason 4</div>

# Better community and support

---
layout: cover
background: https://cover.sli.dev
---

<div class="m-0">Bonus 1</div>

# Common opportunities for <br /> better TypeScript usage

---

## The `satisfies` Operator

Make type inference a little smarter.

````md magic-move
```ts{*|9-10}
type RGB = [red: number, green: number, blue: number];

const palette: Record<string, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
};

palette.red; // type: string | RGB
palette.green; // type: string | RGB
```

```ts{*|9-10}
type RGB = [red: number, green: number, blue: number];

const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} satisfies Record<string, string | RGB>;

palette.red; // type: RGB
palette.green; // type: string
```
````

---

## Type assertions where type annotations should be used instead

Type annotations can be much safer and more helpful.

````md magic-move
```ts
type IWizardStepButtonProps = {
  title: string,
  onClick: () => void,
}

const _getCancelActionProps = () => {
  return {
    title: 'Close',
    onClick: onDismiss,
  } as IWizardStepButtonProps
}
```

```ts {9}
type IWizardStepButtonProps = {
  title: string,
  onClick: () => void,
}

const _getCancelActionProps = () => {
  return {
    title: 'Close',
    onClick: onDismiss, // let's say we forget to add this
  } as IWizardStepButtonProps
}
```

```ts
type IWizardStepButtonProps = {
  title: string,
  onClick: () => void,
}

const _getCancelActionProps = () => {
  return {
    title: 'Close',
  } as IWizardStepButtonProps // Whoops, TypeScript allows this ⛔
}
```

```ts
type IWizardStepButtonProps = {
  title: string,
  onClick: () => void,
}

const _getCancelActionProps = (): IWizardStepButtonProps => {
  return {
    title: 'Close',
  }
  // ^^^ Property 'onClick' is missing in type '{ title: string; }' but required in type 'IWizardStepButtonProps' ✅
}
```
````

---
layout: two-cols-header
layoutClass: gap-4
---

## Not using `unknown` to type thrown error

In JS/TS, *anything* can be thrown from *anywhere*, so an error may not be what you prepare to catch.

Use `unknown` to ensure code safety. In TS 5.4.5, `error` is `unknown` by default.


````md magic-move
```ts {*|5-10}
try {
  const response = await deleteSubscription(subId)
  // Update states...
} catch (error) {
  // error is `any` by default
  // This is allowed by TypeScript, but can throw if `error` is not an object
  Analytics.logClick(
    'OcpSubscriptions-deleteSubscription',
    `LoadFail:${stringifyFetchableDataError(err)}`
  )
}
```

```ts
try {
  const response = await deleteSubscription(subId)
  // Update states...
} catch (error: unknown) {
  if (typeof error === 'object' && error !== null && !Array.isArray(error)) {
    Analytics.logClick(
      'OcpSubscriptions-deleteSubscription',
      `LoadFail:${stringifyFetchableDataError(err)}`
    )
  }
}
```
````

---

## Difference between `any` and `unknown`

They can seem similar, but there are subtle differences.

```ts twoslash
let foo: any
foo.hello()

let bar: unknown
bar.hello()
```

<!-- `any` and `unknown` are both used for unknown types, but different in semantic and usage. Use `any` when you want to skip type checking (not recommended), and `unknown` when you want to enforce type checking before accessing a value for safety -->


---

## Not typing deferred components

If you forget to type a deferred component's props, it will be `any` and all the sweet TypeScript safety is gone.

````md magic-move
```tsx
export type Props = { title: string, header: string }
export const Component = (props: Props) => { ... }
export const DeferredComponent = deferred('Component', () => moduleRef('./Component'))

<DeferredComponent > // No error here ⛔
```

```tsx
export type Props = { title: string, header: string }
export const Component = (props: Props) => { ... }
export const DeferredComponent = deferred<Props>('Component', () => moduleRef('./Component'))

<DeferredComponent > // TypeScript reminds us to provide `title` and `header` props ✅
```
````

---
layout: cover
background: https://cover.sli.dev
---

<div class="m-0">Bonus 2</div>

# Cool new JS and TS features

<!--
Curated list of most relevant features you can use (once updated)

Overall theme: refinement, makes TS smarter and work as you would expect. Add support for edge/complex cases (usage with abstract classes, getter/setter)
-->

---

## Template String Type Improvements

Actually you can use it today!

```ts
type VerticalPosition = "Top" | "Bottom";
type HorizontalPosition = "Left" | "Right";

// "TopLeft" | "TopRight" | "BottomLeft" | "BottomRight"
type Position = `${VerticalPosition}${HorizontalPosition}`;
```

Compatible types

```ts
declare let s1: `${number}-${number}-${number}`;
declare let s2: `1-2-3`;

// TypeScript recognizes that s2 is compatible with s1,
// therefore s1 can be assigned to s2's value
s1 = s2;
```

---

## The `Awaited` Type and Promise Improvements

New utility type to unwrap `Promise` types.

```ts
// A = string
type A = Awaited<Promise<string>>;

// B = number
type B = Awaited<Promise<Promise<number>>>;

// C = boolean | number
type C = Awaited<boolean | Promise<number>>;
```

---
layout: two-cols-header
layoutClass: gap-2
---

## Decorators

Easily enhance functions and methods.

::left::

```ts
class Person {
  constructor(private name: string) {}

  @loggedMethod("⚠️")
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

const p = new Person("Ron");
p.greet();

// Output:
//
//   ⚠️ Entering method 'greet'.
//   Hello, my name is Ron.
//   ⚠️ Exiting method 'greet'.
```

::right::

```ts
function loggedMethod(prefix = "LOG:") {
  return function actualDecorator(
    originalMethod: any,
    context: ClassMethodDecoratorContext
  ) {
    const methodName = String(context.name);

    function replacementMethod(this: any, ...args: any[]) {
      console.log(`${prefix} Entering method '${methodName}'.`);
      const result = originalMethod.call(this, ...args);
      console.log(`${prefix} Exiting method '${methodName}'.`);
      return result;
    }

    return replacementMethod;
  };
}
```

---
layout: two-cols-header
layoutClass: gap-4
---

## `using` Declarations and Explicit Resource Management

::left::

Before

```ts {*|11-16|*}
function doSomeWork() {
  const path = ".some_temp_file";
  const file = fs.openSync(path, "w+");

  try {
    // use file...

    if (someCondition()) {
      // do some more work...
      return;
    }
  } finally {
    // Close the file and delete it.
    fs.closeSync(file);
    fs.unlinkSync(path);
  }
}
```

::right::

After

```ts {*|2|*}
function doSomeWork() {
  using file = new TempFile(".some_temp_file");
  // use file...
  if (someCondition()) {
    // do some more work...
    return;
  }
}
```

```ts {*|7-10|*}
class TempFile implements Disposable {
  constructor(private path: string) {
    this.handle = fs.openSync(path, "w+");
  }

  // other methods

  [Symbol.dispose]() {
    fs.closeSync(this.handle);
    fs.unlinkSync(this.path);
  }
}
```

---

## Top level `await`

`await` can now exist outside of an `async` function (top level only).

````md magic-move
```js
let user;

const setUser = async () => {
  const response = await getUserById('123');
  user = response.user;
};

setUser();
```

```js
let user;

const response = await getUserById('123');
user = response.user;

setUser();
```
````

---

## Private instance properties

Let's say we want to make `age` a private property.

````md magic-move
<!-- JS Snippet -->
```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const person = new Person('John', 20);
person.age; // This is allowed ⛔
```

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this._age = age;
  }
}

const person = new Person('John', 20);
person._age; // Still allowed ⛔
```

<!-- TS Snippet -->
```ts
class Person {
  constructor(
    private name: string,
    private age: number
  ) {}
}

const person = new Person('John', 20);
person.age; // TypeScript won't let us access this ✅
// ...but, this code can compile to JavaScript and will work fine ⛔
```

```js {*|2|6|11}
class Person {
  #age; // Private properties must be declared

  constructor(name, age) {
    this.name = name;
    this.#age = age;
  }
}

const person = new Person('John', 20);
person.#age; // JavaScript will throw a SyntaxError ✅
```
````

---

## `Array.findLast()`, `Array.findLastIndex()`, `Array.at()`

New handy array methods.

```js
const array = [5, 12, 50, 130, 44];

array.find((element) => element > 45); // 50
array.findIndex((element) => element > 45); // 2
array[array.length - 1]; // 44

array.findLast((element) => element > 45); // 139
array.findLastIndex((element) => element > 45); // 3
array.at(-1); // 44
```

---

## Copying Array Methods

Methods that return a new array instead of mutating the old one.

```js
const array = [1, 2, 3, 4, 5];

// These methods/operations mutate `array`
array.reverse();
array.sort();
array.splice(0, 2);
array[0] = 42;

// These new methods return a new array while keeping the old one intact
array.toReversed();
array.toSorted();
array.toSpliced(0, 2);
array.with(0, 42);
```

---

## `??` operator

Prevent unexpected fallback values.

````md magic-move
```ts
let numLicenses: number | undefined;

const payload = {
  licenses: numLicenses || 1, // When `numLicenses` is 0, `payload.licenses` is 1 ⛔
}
```

```ts
let numLicenses: number | undefined;

const payload = {
  licenses: numLicenses ?? 1, // `payload.licenses` is 1 only when `numLicenses` is undefined ✅
}
```
````

---

## Numeric separator

Make numbers easier to read.

```js
// Without separator
let num1 = 1000000000;

// With separator
let num2 = 1_000_000_000;
```

---

## Deep object copy

A performant way to deep copy an object.

```js
const obj = {};

// This is slow
const copy1 = JSON.parse(JSON.stringify(obj));

// This is fast
const copy2 = structuredClone(obj);
```

---
layout: cover
background: https://cover.sli.dev
---

# Thank you for the discussion!

Feedback is welcome, and next topics are encouraged.
