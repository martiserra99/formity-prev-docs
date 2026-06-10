# Variables

Learn how the variables element is used in the flow.

## Usage

The **variables** element defines variables that are added to the flow values.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Variables<{ fullName: string }>,
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
    variables: ({ name, surname }) => ({
      fullName: `${name} ${surname}`,
    }),
  },
  {
    return: ({ fullName, age }) => ({ fullName, age }),
  },
];
```

The `variables` function receives the flow values and returns the variables to be added.
