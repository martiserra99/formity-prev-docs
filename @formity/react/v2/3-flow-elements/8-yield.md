# Yield

Learn how the yield element is used in the flow.

## Usage

The **yield** element calls the `onYield` callback when the user navigates between steps, yielding values both when going forward and backward.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Yield<{
      next: [{ name: string; surname: string; age: number }];
      back: [];
    }>,
    s.Form<{ softwareDeveloper: string }>,
    s.Yield<{
      next: [{ softwareDeveloper: string }];
      back: [];
    }>,
    s.Return<{
      name: string;
      surname: string;
      age: number;
      softwareDeveloper: string;
    }>,
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
    yield: {
      next: ({ name, surname, age }) => [{ name, surname, age }],
      back: () => [],
    },
  },
  {
    form: {
      fields: () => ({ softwareDeveloper: ["", []] }),
      render: ({ fields, onBack, onNext }) => (
        <SoftwareDeveloperForm
          fields={fields}
          onBack={onBack}
          onNext={onNext}
        />
      ),
    },
  },
  {
    yield: {
      next: ({ softwareDeveloper }) => [{ softwareDeveloper }],
      back: () => [],
    },
  },
  {
    return: ({ name, surname, age, softwareDeveloper }) => ({
      name,
      surname,
      age,
      softwareDeveloper,
    }),
  },
];
```

When navigating forward, `next` is called; when navigating backward, `back` is called. Both receive the flow values and return an array in which each element triggers one call to `onYield`.
