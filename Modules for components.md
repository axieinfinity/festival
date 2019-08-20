When designing and using [React components](https://reactjs.org/docs/components-and-props.html#function-and-class-components), we usually need to [export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) and [import](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import) more than just the component itself. For example, we may want to import the [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html) of a prop (e.g., button's color) or its implementations/options (e.g., the "primary" color for a button).

This page describes our expectations, several approaches, and what we believe is the optimal solution to export and import React components as JS modules (for now).

# Expectations

We prefer approaches that:

- **Enhance discoverability:** easy to learn about props and their options right from the editor.
- **Follow standards:** no adhoc hack, not too far from the outside world, and should allow tree-shaking methods to work.
- **Help with naming:** reduce manual works to address naming conflicts when importing props with same name from different components (e.g., button's color and tag's color).

# Approaches

Although there are only 2 types of export in JavaScript (default and named), there are [many ways](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Syntax) (in term of syntax) to export and import from a module. Here we only discuss the most prominent ones:

## A. All are named without prefix

```tsx
// Button.ts
export interface Color { light: string; dark: string; }
export const Colors: { [key: string]: Color } = { /* ... */}
export const Component = (props: { color: Color }) => (/* ... */);
```

```tsx
// Foo.tsx
import * as Button from "@dvkndn/core/button";
import * as Tag from "@dvkndn/core/tag";
<Button.Component color={Button.Colors.Primary} />
```

## B. All are named with prefix

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

## C. Component is default, others are named without prefix

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

# Current solution

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

# References

- [https://create-react-app.dev/docs/importing-a-component](https://create-react-app.dev/docs/importing-a-component)
- [https://exploringjs.com/impatient-js/ch_modules.html](https://exploringjs.com/impatient-js/ch_modules.html)
- [https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/](https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/)
- [https://stackoverflow.com/questions/42051588/wildcard-or-asterisk-vs-named-or-selective-import-es6-javascript](https://stackoverflow.com/questions/42051588/wildcard-or-asterisk-vs-named-or-selective-import-es6-javascript)
- [https://gist.github.com/Rich-Harris/24a8ddd3947150aa8c15a9367faf1d62](https://gist.github.com/Rich-Harris/24a8ddd3947150aa8c15a9367faf1d62)
- [https://medium.com/@skovy/using-component-dot-notation-with-typescript-to-create-a-set-of-components-b0b2aad4892b](https://medium.com/@skovy/using-component-dot-notation-with-typescript-to-create-a-set-of-components-b0b2aad4892b)
