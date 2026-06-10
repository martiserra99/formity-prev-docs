# Form

Learn how the form element is used in the flow.

## Usage

The **form** element defines a step in the multi-step form.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Return<{ name: string; surname: string; age: number }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, onNext }) => (
        <AboutMeForm fields={fields} onNext={onNext} />
      ),
    },
  },
  {
    return: ({ name, surname, age }) => ({ name, surname, age }),
  },
];
```

## Fields

The `fields` function defines the default values of the form. It receives the flow values and returns an object where each key maps to a tuple of the default value and an array.

```tsx
fields: () => ({
  name: ["", []],
  surname: ["", []],
  age: [20, []],
}),
```

When the user navigates back to a step they already filled out, the previously entered values are used as defaults — as long as the array remains unchanged.

## Render

The `render` function renders the form step. It receives an object with the following properties:

- `fields`: Default values of the form.
- `values`: The flow values.
- `params`: Values passed from outside via the `params` prop.
- `onNext`: Goes to the next step. Takes the form values as argument.
- `onBack`: Goes to the previous step. Takes the form values as argument.
- `onJump`: Jumps to a specific form. Takes the jump element's id and form values.
- `getState`: Returns the current state. Takes the form values as argument.
- `setState`: Updates the state. Takes the new state as argument.

```tsx
render: ({ fields, onNext }) => (
  <AboutMeForm fields={fields} onNext={onNext} />
),
```
