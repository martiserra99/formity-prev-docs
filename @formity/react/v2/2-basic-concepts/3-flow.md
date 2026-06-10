# Flow

Learn what the flow is about and how it can be used to define the forms.

## Flow

The flow is used to define the structure and behavior of the multi-step form. It is an array of different elements, and the existing ones are as follows:

- **Form**: Defines a step in the form.
- **Return**: Defines what values the multi-step form returns.
- **Variables**: Defines variables.
- **Condition**: Defines a condition.
- **Switch**: Defines a switch.
- **Jump**: Defines a jump.
- **Loop**: Defines a loop.
- **Yield**: Defines what values the multi-step form yields.

By combining these elements, we can create advanced multi-step forms with complex logic. Here's a simple example with two form steps and a return.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string }>,
    s.Form<{ softwareDeveloper: string }>,
    s.Return<{ name: string; surname: string; softwareDeveloper: string }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({ name: ["", []], surname: ["", []] }),
      render: ({ fields, onNext }) => (
        <AboutMeForm fields={fields} onNext={onNext} />
      ),
    },
  },
  {
    form: {
      fields: () => ({ softwareDeveloper: ["", []] }),
      render: ({ fields, onBack, onNext }) => (
        <SoftwareDeveloperForm fields={fields} onBack={onBack} onNext={onNext} />
      ),
    },
  },
  {
    return: ({ name, surname, softwareDeveloper }) => ({
      name,
      surname,
      softwareDeveloper,
    }),
  },
];

return <Formity<Schema> flow={flow} onReturn={onReturn} />;
```

The flow is of type `Flow`, which accepts a `Schema` type that ensures complete type safety across all steps. For a deeper understanding of each element, refer to their respective sections.
