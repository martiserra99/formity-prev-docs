# Inputs

Learn how to pass values to be used inside the flow.

## Inputs

As the flow progresses, an object called the **flow values** is made available to each element. It contains the values collected from form elements and variable elements encountered so far.

To include additional values from the start, use the `inputs` prop of the `Formity` component.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [s.Form<{ language: string }>, s.Return<{ language: string }>];
  inputs: {
    languages: { label: string; value: string }[];
    defaultLanguage: string;
  };
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: ({ defaultLanguage }) => ({
        language: [defaultLanguage, []],
      }),
      render: ({ fields, values, onBack, onNext }) => (
        <LanguageForm
          fields={fields}
          languages={values.languages}
          onBack={onBack}
          onNext={onNext}
        />
      ),
    },
  },
  {
    return: ({ language }) => ({ language }),
  },
];

return (
  <Formity<Schema>
    flow={flow}
    inputs={{
      languages: [
        { label: "English", value: "en" },
        { label: "Spanish", value: "es" },
        { label: "Catalan", value: "ca" },
      ],
      defaultLanguage: "en",
    }}
    onReturn={onReturn}
  />
);
```
