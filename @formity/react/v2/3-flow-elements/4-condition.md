# Condition

Learn how the condition element is used in the flow.

## Usage

The **condition** element branches the flow based on a condition.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ working: string }>,
    s.Variables<{ company: string | null }>,
    s.Condition<{
      then: [s.Form<{ company: string }>];
      else: [];
    }>,
    s.Form<{ study: string }>,
    s.Return<{
      working: string;
      company: string | null;
      study: string;
    }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({ working: ["yes", []] }),
      render: ({ fields, onBack, onNext }) => (
        <WorkingForm fields={fields} onBack={onBack} onNext={onNext} />
      ),
    },
  },
  {
    variables: () => ({ company: null }),
  },
  {
    condition: {
      if: ({ working }) => working === "yes",
      then: [
        {
          form: {
            fields: () => ({ company: ["", []] }),
            render: ({ fields, onBack, onNext }) => (
              <CompanyForm fields={fields} onBack={onBack} onNext={onNext} />
            ),
          },
        },
      ],
      else: [],
    },
  },
  {
    form: {
      fields: () => ({ study: ["business", []] }),
      render: ({ fields, onBack, onNext }) => (
        <StudyForm fields={fields} onBack={onBack} onNext={onNext} />
      ),
    },
  },
  {
    return: ({ working, company, study }) => ({ working, company, study }),
  },
];
```

The `if` function receives the flow values and returns a boolean that determines which branch to execute. Both `then` and `else` accept any flow elements.
