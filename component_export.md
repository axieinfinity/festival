# Exports in components

## Abstract

When users use our [components](https://reactjs.org/docs/components-and-props.html#function-and-class-components), they may need to [import](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import) more than just the component itself. For example, they may want to use the [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html) of a prop (e.g., button's color) or its built-in options (e.g., the "primary" color for a button). The exports of a component, therefore, should be designed to cover all of these use cases in an effective and intuitive way.

This page describes our expectations and approaches in exporting a React component and its related entities.

## TL;DR

`Button.tsx`
```tsx
export interface ButtonColor {
  text: string;
  bg: string;
};

interface Props { color: ButtonColor; }
const Button = (props: Props) => (...);

Button.colors = {
  primary: { text: "", bg: "" },
  neutral: { text: "", bg: "" },
};

export default Button;
```

`Foo.tsx`
```tsx
import Button, { ButtonColor } from "...";

// Common usages, you don't need ButtonColor here
<Button color={Button.colors.primary} />

// Advanced usages
const customColor: ButtonColor = { text: "", bg: "" };
<Button color={customColor} />
```

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
/* A */ export default Button;
/* B */ export const Button = (props) => ...; // global name
/* C */ export const Component = (props) => ...; // local name
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

It's common for users to import and use values (usually options of props) to make the most out of a component. Since we decided to export our component as the default one, we have 3 ways to export our values along with it:

```tsx
/* A */ export const colors = { ... }; // local name
/* B */ export const ButtonColors = { ... }; // global name
/* C */ Button.colors = { ... }; // attach to the component
```

In most cases, if not all, our users should already imported the Button component. Because of this, approach C has a clear advantage because users' editors can easily list these built-in values following the familiar dot notation:

<img width="647" alt="Screen Shot 2019-08-21 at 1 06 08 AM" src="https://user-images.githubusercontent.com/5953369/63372263-ef1f2680-c3af-11e9-9683-865e21199b93.png">

<img width="647" alt="Screen Shot 2019-08-21 at 1 06 22 AM" src="https://user-images.githubusercontent.com/5953369/63372262-ee869000-c3af-11e9-8f9d-6666ddfe8529.png">

This greatly enhances the discoverability of the component as users only need to type the first few characters of a prop after any occurence of the imported component. In constrast, both approach A and B require the users to go back to the import statements (note that TypeScript's auto import [can't help here](https://user-images.githubusercontent.com/4246176/63336293-21f1fc00-c369-11e9-8c09-dd6c52941a6b.png)).

### The types

Unlike the component and the values, types (including interfaces) are not expected to be imported frequently:
- Props with interfaces are advanced use cases in the first place.
- Even in those cases, the built-in values should be enough most of the time.
- Even if we need custom values, it can also be done inline, without the need to import any interfaces:

```tsx
<Button colors={{ text: "...", bg: "..." }} />
```

Therefore, trying to fight with TypeScript to attach them to the component (like values) doesn't worth the effort. Instead, normal named export are perectly fine for this case. The question left is whether we should use local or global names:

```tsx
/* A: local name */
export interface Color = { ... };
import Button, { Color as ButtonColor } from "...";

/* B: global name */
export interface ButtonColor = { ... };
import Button, { ButtonColor } from "...";
```

Although not technically required, we prefer approach B here:
- For types of props, naming conflicts are common (e.g., many components could have the "color" prop), so it's better to help our users in the first place.
- Instead of `export default () => ...`, we prefered `export default Button`, so it's more unified to have other export entries to start with Button. In other words, we already named our export entries with global name.
- It helps a little for code navigation.

## References

- [https://create-react-app.dev/docs/importing-a-component](https://create-react-app.dev/docs/importing-a-component)
- [https://exploringjs.com/impatient-js/ch_modules.html](https://exploringjs.com/impatient-js/ch_modules.html)
- [https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/](https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/)
- [https://stackoverflow.com/questions/42051588/wildcard-or-asterisk-vs-named-or-selective-import-es6-javascript](https://stackoverflow.com/questions/42051588/wildcard-or-asterisk-vs-named-or-selective-import-es6-javascript)
- [https://gist.github.com/Rich-Harris/24a8ddd3947150aa8c15a9367faf1d62](https://gist.github.com/Rich-Harris/24a8ddd3947150aa8c15a9367faf1d62)
- [https://medium.com/@skovy/using-component-dot-notation-with-typescript-to-create-a-set-of-components-b0b2aad4892b](https://medium.com/@skovy/using-component-dot-notation-with-typescript-to-create-a-set-of-components-b0b2aad4892b)
