# Return

Learn how the return element is used in the flow.

## Usage

The **return** element defines what the multi-step form returns and triggers the `onReturn` callback.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Return<{ fullName: string; age: number }>,
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
      render: ({ fields, onBack, onNext }) => (
        <AboutMeForm fields={fields} onBack={onBack} onNext={onNext} />
      ),
    },
  },
  {
    return: ({ name, surname, age }) => ({
      fullName: `${name} ${surname}`,
      age,
    }),
  },
];
```

The `return` function receives the flow values and returns the values to be passed to `onReturn`.
