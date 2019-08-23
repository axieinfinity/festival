# Exporting components

## 1. Abstract

When our users use the [components](https://reactjs.org/docs/components-and-props.html#function-and-class-components) that we designed, they may need to [import](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import) more than just the components alone. For example, they may want to utilize the [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html) of a prop (e.g., button's color) or to use its built-in options (e.g., the "primary" color for a button).

What to export from a component, and how to export them, are therefore critical to the design of UI components. This page describes our expectations and approaches in exporting React components and their related entities, for both external and internal uses. So far, what we learned are:

1. Use named export for components
1. Attach built-in values to the exported components
1. Use named export for types and interfaces

For a code representation of these principles, see the [Conclusion](#4-conclusion) section.


## 2. Expectations

We are looking for solutions that are:

- **Friendly:** Should not force or even assume any import style from the users. However, do expect and follow common, established practices of the community.
- **Informative:** Should leverage the type system to let the users easily discover not only the available components but also their props and built-in values for these props.

## 3. Approach

First, we will compare the 2 target places to export your components to: internal (same package) and external (another package). Then, we will go through the 3 types of entities that users often need to import for a component, from most used to least, to see what are the suitable ways to export them:

1. The component itself (usually a function or a class)
2. The built-in values to use with the component's props (e.g., to have a button with "primary" color)
3. The types of the component's props (e.g., the interface of the "color" prop)

### 3.1. External and internal uses

Exported components can be imported from 2 places: internal (i.e., the users are in the same package of the components) or external (i.e., the users import the components from a distributed package). Although there are cases where different decisions must be made for each, we do prefer approaches that work best for both.

In internal case, components are imported via their paths, either relative or absolute:

```tsx
// apps/…/foo.tsx
import … from "./baz/bar/bar";
import … from "src/common/bar/bar";
```

In external case, besides path imports, users can, and usually, use top-level imports:

```tsx
// apps/…/foo.tsx
import { Button, Tag } from "libs/core";
```

In fact, from users perspective, we recommend to avoid using path imports when importing components from other packages because of several design issues:

- Several entry points to a package increase its maintenance cost and decrease its encapsulation.
- Path to files in a package are internal implementation details that should not be exposed to outside.
- Lack of a place to see all public API reduces the discoverability of a package.

Instead, we recommend, both library authors and users, to consider top-level imports as the only entry points to the components. Thanks to modern build tools, there is [no cost for end users](https://webpack.js.org/guides/tree-shaking/#mark-the-file-as-side-effect-free) here. The authors only need to define an index file at root that re-exports all available entities:

```ts
// libs/core/index.ts
export … from "button/button";
export … from "tag/tag";
```



Because of this recommendation, in further discussion, you may see that we:

- Take the index file (at package’s root) into consideration.
- Prefer top-level imports over path imports.

### 3.2. The components

In component-based systems, it is common for components to be the essence of files. Using default exports, therefore, is a natural approach. In fact, it is [recommended by CRA](https://create-react-app.dev/docs/importing-a-component), and _was_ our recommendation for a long time.

#### 3.2.1. Issues of default exports for components

After a long time observing how our components are used in practice, we found issues that require us to reconsider our decision. In general, we learned that default exports can lead to extra work, implicit code and cognitive overhead.

For example, it allows authors of components to export anonymous entities:

```tsx
// libs/core/button/button.tsx
export default (props: Props) => …;
```

Although this [can be prevented](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/no-anonymous-default-export.md), we do prefer approaches that don’t require us to do extra work to get things right.

Default exports also allow users to name an export completely different than what the author intended.

```tsx
// apps/…/foo.tsx
import Tag from "./button/button";
```

Although it’s users’ right to use whatever name they want, we rarely see they “claim” that right in practice. Authors’ intended names for the components are used in most cases, if not all. Name conflicts are rare, and even in those cases, the implicit aliasing make finding usages and refactoring much more difficult.

Moreover, for libraries, default exports require (somewhat weird) extra work in re-exporting at top-level:

```ts
// libs/core/index.ts
// For the component
export { default as Button } from "./button/button";
// For types and interfaces (which must use named exports)
export * from "./button/button";
```

#### 3.2.2. Named exports for components

Because of these reasons, we now recommend to **use named exports for components**:

```tsx
// libs/core/button/button.tsx
export const Button = (props: Props) => …
```

Named exports eliminate all above issues. For example, they are a great help in searching for usages and refactoring, since the original names are always referenced whenever they are used, no matter how users import them:

```tsx
// apps/…/foo.tsx

// alias imports:
import { Button as FooButton } from "libs/core";
//       ^^^^^^

// namespace imports:
import * as core from "libs/core";
core.Button
//   ^^^^^^
```

Named exports are also better for re-exporting in the case of libraries:

```ts
// libs/core/index.ts
export * from "./button/button";
```

### 3.3. The values

It’s obvious that built-in values can’t use default exports. Named exports, however, can also be problematic for components that have many set of built-in values, because using them is common use case:

```tsx
// apps/…/foo.tsx
import { Button, ButtonColors, ButtonHeights } from "lib/core";
```

Fortunately, we are not limited to only default and named exports. We can also attach these values to their (exported) component, just like properties of an object (actually, they are):

```tsx
// libs/core/button/button.tsx
Button.colors = {
  primary: { … },
  neutral: { … },
};
```

In fact, this is a great approach for exporting built-in values. While named exports require users to go back to the import statements to explore all options (since auto import [can't help](https://user-images.githubusercontent.com/4246176/63336293-21f1fc00-c369-11e9-8c09-dd6c52941a6b.png) here), this approach doesn’t require users to even import anything other than the component, and can explore all built-in values from that component directly:

<img width="647" alt="Screen Shot 2019-08-21 at 1 06 08 AM" src="https://user-images.githubusercontent.com/5953369/63372263-ef1f2680-c3af-11e9-9683-865e21199b93.png">

<img width="647" alt="Screen Shot 2019-08-21 at 1 06 22 AM" src="https://user-images.githubusercontent.com/5953369/63372262-ee869000-c3af-11e9-8f9d-6666ddfe8529.png">

### 3.4. The types

It would be great if we can also attach types and interfaces to components, like built-in values:

```
// apps/…/foo.tsx
const customColor: Button.Color
```

However, this is currently not possible in TypeScript, as we can’t attach something that isn’t a real value to objects. A workaround is to use TypeScript’s [namespaces](https://www.typescriptlang.org/docs/handbook/namespaces.html) to achieve the above syntax, but they have many known caveats, such as the use of global objects and weird [reference syntax](https://www.typescriptlang.org/docs/handbook/namespaces.html#multi-file-namespaces).

This takes us back to named exports, which didn’t work for built-in values previously, but work just fine for types. It is mainly because types are not common imports like values, which means the problems of long imports and poor discoverability, although exist, are not practical ones.

```tsx
// libs/core/button/button.tsx
export interface ButtonColor { … }
```

It’s worth to note that users don’t need to explicitly import types and interfaces to use our components effectively, as built-in values should work for most cases. Even when they don’t, users can still define custom ones inline and fully have type safety:

```tsx
// app/…/foo.tsx
import { Button } from "libs/core";
<Button color={{ … }} />
```

## 4. Conclusion

First, use named export for components:

```tsx
// libs/core/button/button.tsx
interface Props { color: ButtonColor; }
export const Button = (props: Props) => (…);
```

Second, attach built-in values to the exported components:

```ts
// libs/core/button/button.tsx
Button.colors = {
  primary: { text: "", bg: "" },
  neutral: { text: "", bg: "" },
};
```

Third, use named export for types and interfaces:

```tsx
// libs/core/button/button.tsx
export interface ButtonColor {
  text: string;
  bg: string;
}
```

Finally, for libraries, re-export all at packages’ root:

```ts
// libs/core/index.ts
export * from "./button/button.tsx";
```

Sample usages:

```tsx
// app/…/foo.tsx
import { Button } from "libs/core";

<Button color={Button.colors.primary} />
```

```tsx
// app/…/foo.tsx
import { Button, ButtonColor } from "libs/core";

const customColor: ButtonColor = { text: "", bg: "" };
<Button color={customColor} />
```

## 5. References

- [https://create-react-app.dev/docs/importing-a-component](https://create-react-app.dev/docs/importing-a-component)
- [https://exploringjs.com/impatient-js/ch_modules.html](https://exploringjs.com/impatient-js/ch_modules.html)
- [https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/](https://humanwhocodes.com/blog/2019/01/stop-using-default-exports-javascript-module/)
- [https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html](https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html)
- [https://github.com/palantir/blueprint/wiki/Coding-guidelines#typescript](https://github.com/palantir/blueprint/wiki/Coding-guidelines#typescript)
