# Introduction

Define your entire multi-step form as a single, programmable flow.

## What is Formity?

Formity is a React library for building multi-step forms as programmable flows that can branch, loop, jump, and track variables across steps, giving you full control over the form's behavior.

Fully type-safe and compatible with [React Hook Form](https://react-hook-form.com/), [Formik](https://formik.org/), [TanStack Form](https://tanstack.com/form/latest), and any other form library. It works for onboarding flows, lead capture forms, job applications, and more.

## Installation

```bash
npm install @formity/react
```

## How it works

The core of Formity is the flow, an array of elements that each define a piece of the form's behavior. By combining them, you can build any multi-step form you can imagine.

```tsx
const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({ softwareDeveloper: ["yes", []] }),
      render: ({ fields, onNext }) => (
        <SoftwareDeveloperForm fields={fields} onNext={onNext} />
      ),
    },
  },
  {
    variables: () => ({ language: null }),
  },
  {
    condition: {
      if: ({ softwareDeveloper }) => softwareDeveloper === "yes",
      then: [
        {
          form: {
            fields: () => ({ language: ["", []] }),
            render: ({ fields, onBack, onNext }) => (
              <LanguageForm fields={fields} onBack={onBack} onNext={onNext} />
            ),
          },
        },
      ],
      else: [],
    },
  },
  {
    return: ({ softwareDeveloper, language }) => ({
      softwareDeveloper,
      language,
    }),
  },
];

return <Formity<Schema> flow={flow} onReturn={onReturn} />;
```
