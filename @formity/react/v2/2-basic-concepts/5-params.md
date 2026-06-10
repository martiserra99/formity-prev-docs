# Params

Learn how to pass values that can be used when rendering each form.

## Params

The `params` prop lets you pass values to be used when rendering each form step.

Unlike inputs, params are not part of the flow logic — they exist solely for rendering, and any changes to them will immediately reflect in the form.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [s.Form<{ language: string }>, s.Return<{ language: string }>];
  inputs: {
    languages: { label: string; value: string }[];
    defaultLanguage: string;
  };
  params: {
    heading: string;
  };
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: ({ defaultLanguage }) => ({
        language: [defaultLanguage, []],
      }),
      render: ({ fields, values, params, onBack, onNext }) => (
        <LanguageForm
          fields={fields}
          languages={values.languages}
          heading={params.heading}
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
    params={{
      heading: "What language do you speak the most?",
    }}
    onReturn={onReturn}
  />
);
```
