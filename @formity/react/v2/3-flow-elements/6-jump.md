# Jump

Learn how the jump element is used in the flow.

## Usage

The **jump** element defines a form that can be jumped to from anywhere in the flow.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ skills: string }>,
    s.Form<{ tools: string }>,
    s.Jump<s.Form<{ notice: string }>>,
    s.Return<{
      skills: string;
      tools: string;
      notice: string;
    }>,
  ];
  inputs: {
    skills: string;
    tools: string;
  };
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({ skills: ["", []] }),
      render: ({ fields, onNext, onJump }) => (
        <SkillsForm fields={fields} onNext={onNext} onJump={onJump} />
      ),
    },
  },
  {
    form: {
      fields: () => ({ tools: ["", []] }),
      render: ({ fields, onBack, onNext }) => (
        <ToolsForm fields={fields} onBack={onBack} onNext={onNext} />
      ),
    },
  },
  {
    jump: {
      id: "notice",
      at: {
        form: {
          fields: () => ({ notice: ["", []] }),
          render: ({ fields, onBack, onNext }) => (
            <NoticeForm fields={fields} onBack={onBack} onNext={onNext} />
          ),
        },
      },
    },
  },
  {
    return: ({ skills, tools, notice }) => ({
      skills,
      tools,
      notice,
    }),
  },
];

return (
  <Formity<Schema>
    flow={flow}
    inputs={{ skills: "", tools: "" }}
    onReturn={onReturn}
  />
);
```

The `id` property identifies the jump target. When `onJump` is called with that `id`, the flow jumps to the form defined in the `at` property.

Since a jump can be triggered from anywhere in the flow, the values from previous steps may not have been collected when the target is reached.

Providing defaults via `inputs` ensures all values are present at the jump target, no matter which path the user took to get there.
