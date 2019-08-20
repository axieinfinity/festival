# Extensible props in components

This page describes our expectations and approaches to export the API of React components, as JS modules. For a TL;DR, please go to [Conclusion](https://github.com/axieinfinity/festival/blob/master/modules_and_components.md#conclusion) section.

## Background

When users use our [components](https://reactjs.org/docs/components-and-props.html#function-and-class-components), they may need to [import](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import) more than just the component itself. For example, they may want to use the [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html) of a prop (e.g., button's color) or its built-in options (e.g., the "primary" color for a button). The export(s) of a component, therefore, should be designed to cover all of these use cases in an effective and intuitive way.

## Expectations

We prefer approaches that are:

- **Friendly:** Should not assume or enforce "import" styles but do expect and follow [common, established practices](https://create-react-app.dev/docs/importing-a-component) of the community.
- **Informative:** Should take advantage of TypeScript to let engineers easily discover not only the available props but also the options of these props.

## Approaches
