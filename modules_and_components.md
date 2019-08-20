# Modules and components

## Background

When users use our [components](https://reactjs.org/docs/components-and-props.html#function-and-class-components), they may need to [import](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import) more than just the component itself. For example, they may want to use the [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html) of a prop (e.g., button's color) or its built-in options (e.g., the "primary" color for a button). Good components, therefore, should [export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) its API with this in mind.

This page describes our expectations, several approaches, and what we believe is the optimal solution (for now) to export the API of React components, as JS modules.

## Expectations

We prefer approaches that are:

- **Friendly:** Should not assume or enforce "import" styles but do expect and follow [common, established practices](https://create-react-app.dev/docs/importing-a-component) of the community.
- **Informative:** Should take advantage of TypeScript to let engineers easily discover not only the available props but also the options of these props.

## Approaches

Usually, there are 3 types of things that users would want to import/consume from a component, sorted by usage:
1. The component itself (usually a function but could also be a class)
2. The values to use with the component's props (e.g., to have a button with "primary" color)
3. The types of the component's props (e.g., the interface of the "color" prop)

We will go through each type to see what's the suitable way to export them, then we should be able to see how should we export all of them together.

### The component

Like everything else, the component can be exported as default, or named. In case of named export, it can use a global name, or a local one:

```tsx
/* A */ export default (props) => ...
/* B */ export const Button = (props) => ... // global name
/* C */ export const Component = (props) => ... // local name
```

Taking the "Friendly" expectation into consideration, we can see the local name approach (C) is not a good one. In this case, the users must import with either [namespace](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Import_an_entire_module's_contents) or [alias](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Import_an_export_with_a_more_convenient_alias). The former introduces a weird usage, while the latter brings unnecessary, extra works for the users:

```tsx
// Approach C with namespace import
import * as Button from "...";
<Button.Component /> // weird usage
// Approach C with alias import
import { Component as Button } from "..."; // extra work
<Button />
```

Both the A and B approaches should easily pass our "Friendly" expectation, as they are both widely used in UI libraries. However, to choose one, we prefer A over B because:

1. The component is semantically the heart of the file/module, and users expect it in most, if not all, cases.
2. It's the [official recommendation](https://create-react-app.dev/docs/importing-a-component).

### The values

### The types

### A. All named

```tsx
// Button.ts
export interface Color { light: string; dark: string; }
export const Colors: { [key: string]: Color } = { /* ... */}
export const Component = (props: { color: Color }) => (/* ... */);
```

```tsx
// Foo.tsx (common use cases)
import { Button } from "@axie/ds/button";
<Button color={Button.Colors.Primary} />
```

```tsx
// Foo.tsx (rare use cases)
import { Button, ButtonColor } from "@axie/ds/button";
const customColor: ButtonColor = { ... };
<Button color={customColor} />
```

### B. All are named with prefix

```tsx
// Button.ts
export interface ButtonColor { light: string; dark: string; }
export const ButtonColors: { [key: string]: Color } = { /* ... */ };
export const Button = (props: { color: Color }) => (/* ... */);
```

```tsx
// Foo.tsx
import { Button, ButtonColors } from "@dvkndn/core/button";
import { Tag, TagColors } from "@dvkndn/core/tag";
<Button color={ButtonColors.Primary} />
```

### C. Component is default, others are named without prefix

```tsx
// Button.ts
export interface Color { light: string; dark: string; }
export const Colors: { [key: string]: Color } = { /* ... */ };
export default (props: { color: Color }) => (/* ... */);
```

```tsx
// C1. Alias import
// Foo.tsx
import Button, { Colors as ButtonColors } from "@dvkndn/core/button";
import Tag, { Colors as TagColors } from "@dvkndn/core/tag";
<Button color={ButtonColors.Primary} />
```

```tsx
// C2. Namespace import
// Foo.tsx
import Button, * as button from "@dvkndn/core/button";
import Tag, * as tag from "@dvkndn/core/button";
<Button color={button.Colors.Primary} />
```

### D.

```tsx
// Button.ts
export interface Color { light: string; dark: string; }
interface Props { color: Color; }
const Button = (props: Props) => (/* ... */);
Button.colors = { primary: { light: "", dark: "" } };
export default Button;
```

```tsx
// Foo.tsx
import Button, { Color as ButtonColor } from "@dvkndn/core/button";
const customColor: ButtonColor = { light: "", dark: "" };
<Button color={Button.colors.primary} />
<Button color={customColor} />
```

### E.

```tsx
// Button.ts
export interface ButtonColor { light: string; dark: string; }
interface Props { color: Color; }
export const Button = (props: Props) => (/* ... */);
Button.colors = { primary: { light: "", dark: "" } };
```

```tsx
// Foo.tsx
import { Button, ButtonColor } from "@dvkndn/core/button";
const customColor: ButtonColor = { light: "", dark: "" };
<Button color={Button.colors.primary} />
<Button color={customColor} />
```

## Current solution

- Speaking of discoverability, A is better than B and C:
    - In A, users can see all exports at every occurrence of `Button`
    - In B and C1, the exports can only be suggested at the import statement. The "auto import" of some editors may not be able to help here (to discover ButtonColors).
    - in C2, exports may be found at occurrences of `button` but this is obviously not as good as A
- Speaking of learning curve, C is the most common approach.
    - A and B are less common because in most systems the API of components are quite simple and only export the component itself.
    - However, these are all standard methods and any engineer should be able to understand how they work in no time.
- Speaking of naming in importing, both A and B are better than C:
    - A is good because of the enforcement of namespace imports, having a nice, concise import statements.
    - B is good because all definitions are globally unique in the first place, thus make the import easier, but could easily lead to long, repetitive import statements (e.g., `import { FooA, FooB, ..., FooX }`).
    - C is not really good as users must always do extra work to avoid naming conflicts, either via alias or via namespace (more work comparing to A).

**Therefore, for now, we prefer the approach A.**

## References

- [https://create-react-app.dev/docs/importing-a-component](https://create-react-app.dev/docs/importing-a-component)
- [https://exploringjs.com/impatient-js/ch_modules.html](https://exploringjs.com/impatient-js/ch_modules.html)
- [https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/](https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/)
- [https://stackoverflow.com/questions/42051588/wildcard-or-asterisk-vs-named-or-selective-import-es6-javascript](https://stackoverflow.com/questions/42051588/wildcard-or-asterisk-vs-named-or-selective-import-es6-javascript)
- [https://gist.github.com/Rich-Harris/24a8ddd3947150aa8c15a9367faf1d62](https://gist.github.com/Rich-Harris/24a8ddd3947150aa8c15a9367faf1d62)
- [https://medium.com/@skovy/using-component-dot-notation-with-typescript-to-create-a-set-of-components-b0b2aad4892b](https://medium.com/@skovy/using-component-dot-notation-with-typescript-to-create-a-set-of-components-b0b2aad4892b)
