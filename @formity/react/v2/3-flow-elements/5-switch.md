# Switch

Learn how the switch element is used in the flow.

## Usage

The **switch** element branches the flow by checking each `case` in order and executing the first one that evaluates to `true`. If none match, `default` is used.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ interested: string }>,
    s.Switch<{
      branches: [
        [
          s.Form<{ whyYes: string }>,
          s.Return<{ interested: "yes"; whyYes: string }>,
        ],
        [
          s.Form<{ whyNot: string }>,
          s.Return<{ interested: "no"; whyNot: string }>,
        ],
      ];
      default: [
        s.Form<{ whyNotSure: string }>,
        s.Return<{ interested: "notSure"; whyNotSure: string }>,
      ];
    }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({ interested: ["yes", []] }),
      render: ({ fields, onBack, onNext }) => (
        <InterestedForm fields={fields} onBack={onBack} onNext={onNext} />
      ),
    },
  },
  {
    switch: {
      branches: [
        {
          case: ({ interested }) => interested === "yes",
          then: [
            {
              form: {
                fields: () => ({ whyYes: ["", []] }),
                render: ({ fields, onBack, onNext }) => (
                  <WhyYesForm fields={fields} onBack={onBack} onNext={onNext} />
                ),
              },
            },
            {
              return: ({ whyYes }) => ({ interested: "yes", whyYes }),
            },
          ],
        },
        {
          case: ({ interested }) => interested === "no",
          then: [
            {
              form: {
                fields: () => ({ whyNot: ["", []] }),
                render: ({ fields, onBack, onNext }) => (
                  <WhyNotForm fields={fields} onBack={onBack} onNext={onNext} />
                ),
              },
            },
            {
              return: ({ whyNot }) => ({ interested: "no", whyNot }),
            },
          ],
        },
      ],
      default: [
        {
          form: {
            fields: () => ({ whyNotSure: ["", []] }),
            render: ({ fields, onBack, onNext }) => (
              <WhyNotSureForm fields={fields} onBack={onBack} onNext={onNext} />
            ),
          },
        },
        {
          return: ({ whyNotSure }) => ({ interested: "notSure", whyNotSure }),
        },
      ],
    },
  },
];
```

Each `case` receives the flow values and returns a boolean to determine whether the branch executes. The `then` array within each branch and the `default` array accept any flow elements.
