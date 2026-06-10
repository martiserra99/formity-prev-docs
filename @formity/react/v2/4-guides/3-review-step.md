# Review step

Learn how to add a review step that lets users edit any previous answer before submitting.

## Initial steps

We'll add a review step that summarizes the collected data and includes edit buttons that jump back to the corresponding form. For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/review-step
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

## Adding jump elements

The edit buttons don't work yet. To enable jumping to previous steps from the review step, we need to wrap each form in a **jump** element. Update `flow.tsx` as follows.

```tsx
// flow.tsx
import type { UnionToIntersection } from "type-fest";
import type { Flow, s } from "@formity/react";
import type { FormStatus } from "./types/status";

import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

import { Form } from "./components/form";
import { Review } from "./components/review";

import * as constants from "./constants";
import * as format from "./utils/format";

type Values = UnionToIntersection<Fields[keyof Fields]>;

type Fields = {
  personal: { name: string; email: string };
  background: { experience: string; jobTitle: string };
  preferences: { employmentType: string; workArrangement: string };
  about: { whyRole: string; greatestStrength: string };
};

export type Schema = {
  render: React.ReactNode;
  struct: [
    s.Jump<s.Form<Fields["personal"]>>,
    s.Jump<s.Form<Fields["background"]>>,
    s.Jump<s.Form<Fields["preferences"]>>,
    s.Jump<s.Form<Fields["about"]>>,
    s.Jump<s.Form<Record<never, never>>>,
    s.Return<Values>,
  ];
  inputs: Record<never, never>;
  params: { status: FormStatus };
};

export const flow: Flow<Schema> = [
  {
    jump: {
      id: "personal",
      at: {
        form: {
          fields: () => ({ name: ["", []], email: ["", []] }),
          render: ({ fields, onBack, onNext }) => (
            <Form
              key="personal"
              defaultValues={fields}
              resolver={zodResolver(
                z.object({
                  name: z.string().nonempty("Please enter your full name"),
                  email: z.email("Please enter a valid email address"),
                }),
              )}
              heading="Personal details"
              message="Let's start with your basic contact information."
              content={[
                { type: "input", name: "name", label: "Full name", placeholder: "Jane Smith" },
                { type: "input", name: "email", label: "Email address", placeholder: "jane@example.com" },
              ]}
              buttons={{ back: null, next: "Continue" }}
              onBack={onBack}
              onNext={onNext}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: "background",
      at: {
        form: {
          fields: () => ({ experience: ["", []], jobTitle: ["", []] }),
          render: ({ fields, onBack, onNext }) => (
            <Form
              key="background"
              defaultValues={fields}
              resolver={zodResolver(
                z.object({
                  experience: z.string().nonempty("Please select your years of experience"),
                  jobTitle: z.string().nonempty("Please enter your current job title"),
                }),
              )}
              heading="Your background"
              message="Tell us about your professional experience."
              content={[
                { type: "select", name: "experience", label: "Years of experience", placeholder: "Select years of experience", options: constants.yearsOfExperience },
                { type: "input", name: "jobTitle", label: "Current job title", placeholder: "Software Engineer" },
              ]}
              buttons={{ back: "Back", next: "Continue" }}
              onBack={onBack}
              onNext={onNext}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: "preferences",
      at: {
        form: {
          fields: () => ({ employmentType: ["", []], workArrangement: ["", []] }),
          render: ({ fields, onBack, onNext }) => (
            <Form
              key="preferences"
              defaultValues={fields}
              resolver={zodResolver(
                z.object({
                  employmentType: z.string().nonempty("Please select an employment type"),
                  workArrangement: z.string().nonempty("Please select a work arrangement"),
                }),
              )}
              heading="Your preferences"
              message="What kind of role are you looking for?"
              content={[
                { type: "select", name: "employmentType", label: "Employment type", placeholder: "Select employment type", options: constants.employmentTypes },
                { type: "select", name: "workArrangement", label: "Work arrangement", placeholder: "Select work arrangement", options: constants.workArrangements },
              ]}
              buttons={{ back: "Back", next: "Continue" }}
              onBack={onBack}
              onNext={onNext}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: "about",
      at: {
        form: {
          fields: () => ({ whyRole: ["", []], greatestStrength: ["", []] }),
          render: ({ fields, onBack, onNext }) => (
            <Form
              key="about"
              defaultValues={fields}
              resolver={zodResolver(
                z.object({
                  whyRole: z.string().nonempty("Please tell us why you want this role"),
                  greatestStrength: z.string().nonempty("Please share your greatest strength"),
                }),
              )}
              heading="About you"
              message="Help us get to know you a little better."
              content={[
                { type: "textarea", name: "whyRole", label: "Why do you want this role?", placeholder: "Tell us what excites you about this opportunity…" },
                { type: "textarea", name: "greatestStrength", label: "What is your greatest strength?", placeholder: "Describe a strength that sets you apart…" },
              ]}
              buttons={{ back: "Back", next: "Continue" }}
              onBack={onBack}
              onNext={onNext}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: "review",
      at: {
        form: {
          fields: () => ({}),
          render: ({ values, params, onNext }) => (
            <Review
              key="review"
              heading="Review your application"
              message="Everything look right? Submit when you're ready."
              content={[
                { text: "Personal details", edit: "personal", rows: [{ label: "Full name", value: format.text(values.name) }, { label: "Email address", value: format.text(values.email) }] },
                { text: "Background", edit: "background", rows: [{ label: "Years of experience", value: format.experience(values.experience) }, { label: "Current job title", value: format.text(values.jobTitle) }] },
                { text: "Preferences", edit: "preferences", rows: [{ label: "Employment type", value: format.employmentType(values.employmentType) }, { label: "Work arrangement", value: format.workArrangement(values.workArrangement) }] },
                { text: "About you", edit: "about", rows: [{ label: "Why this role", value: format.text(values.whyRole) }, { label: "Greatest strength", value: format.text(values.greatestStrength) }] },
              ]}
              button="Submit application"
              onNext={onNext}
              status={params.status}
            />
          ),
        },
      },
    },
  },
  {
    return: (values) => ({
      name: values.name,
      email: values.email,
      experience: values.experience,
      jobTitle: values.jobTitle,
      employmentType: values.employmentType,
      workArrangement: values.workArrangement,
      whyRole: values.whyRole,
      greatestStrength: values.greatestStrength,
    }),
  },
];
```

## Fixing type errors

There are type errors because Formity can't assume previous steps were completed when using jumps. To fix them, we create an `inputs` object with default values that we'll provide to Formity.

Update the `Schema` type to include `inputs` and export the `inputs` object:

```tsx
export type Schema = {
  render: React.ReactNode;
  struct: [
    s.Jump<s.Form<Fields["personal"]>>,
    s.Jump<s.Form<Fields["background"]>>,
    s.Jump<s.Form<Fields["preferences"]>>,
    s.Jump<s.Form<Fields["about"]>>,
    s.Jump<s.Form<Record<never, never>>>,
    s.Return<Values>,
  ];
  inputs: Values;
  params: { status: FormStatus };
};

export const inputs: Values = {
  name: "",
  email: "",
  experience: "",
  jobTitle: "",
  employmentType: "",
  workArrangement: "",
  whyRole: "",
  greatestStrength: "",
};
```

Then, pass the `inputs` object to the `inputs` prop in `app.tsx`.

```tsx
// app.tsx
import type { OnReturn, ReturnOutput } from "@formity/react";
import type { Status } from "./types/status";

import { useState, useCallback } from "react";
import { Formity } from "@formity/react";

import { Done } from "./components/done";

import { flow, inputs, type Schema } from "./flow";

export default function App() {
  const [status, setStatus] = useState<Status<ReturnOutput<Schema>>>({
    type: "form",
    submitting: false,
  });

  const onReturn = useCallback<OnReturn<Schema>>(async (output) => {
    setStatus({ type: "form", submitting: true });

    // Show output in the console
    console.log(output);

    // Simulate a network request
    await new Promise((resolve) => setTimeout(resolve, 1000));

    setStatus({ type: "done", output });
  }, []);

  if (status.type === "done") {
    return (
      <Done
        output={status.output}
        onStartOver={() => setStatus({ type: "form", submitting: false })}
      />
    );
  }

  return (
    <Formity
      flow={flow}
      inputs={inputs}
      params={{ status }}
      onReturn={onReturn}
    />
  );
}
```

## Navigation with jumps

The `onBack` goes to the previously visited step. That means if the user jumped from the review step to a form, pressing back would return to the review step instead of the previous form.

We need to update the `Form` component to use `onJump` instead, so the back button always goes to the right step regardless of where the user came from.

```tsx
// components/form/index.tsx
import type { DefaultValues, Resolver } from "react-hook-form";
import type { OnNext, OnJump } from "@formity/react";

import { useForm, FormProvider } from "react-hook-form";

import { ItemView, type Item } from "./item";
import { Button } from "../button";

interface FormProps<T extends Record<string, unknown>> {
  defaultValues: DefaultValues<T>;
  resolver: Resolver<T>;
  heading: string;
  message: string;
  content: Item[];
  buttons: { back: string | null; next: string };
  onNext: OnNext<T>;
  onJump: OnJump<T>;
  prevId: string | null;
}

export function Form<T extends Record<string, unknown>>({
  defaultValues,
  resolver,
  heading,
  message,
  content,
  buttons,
  onNext,
  onJump,
  prevId,
}: FormProps<T>) {
  const form = useForm({ defaultValues, resolver });
  return (
    <div className="flex h-full w-full items-center justify-center overflow-y-auto bg-white px-4 py-12">
      <div className="w-full max-w-lg">
        <form noValidate autoComplete="off" onSubmit={form.handleSubmit(onNext)}>
          <FormProvider {...form}>
            <div className="mb-8">
              <h2 className="mb-1.5 text-2xl font-bold text-gray-950">{heading}</h2>
              <p className="text-sm font-medium text-gray-400">{message}</p>
            </div>
            <div className="mb-8 flex flex-col gap-6">
              {content.map((item, i) => (
                <ItemView key={i} {...item} />
              ))}
            </div>
            <div className="flex w-full items-center justify-end gap-4">
              {buttons.back && (
                <Button
                  type="button"
                  variant="secondary"
                  onClick={() => onJump(prevId, form.getValues())}
                >
                  {buttons.back}
                </Button>
              )}
              <Button type="submit" variant="primary">
                {buttons.next}
              </Button>
            </div>
          </FormProvider>
        </form>
      </div>
    </div>
  );
}
```

Then update the `Review` component to enable jumping with the edit buttons, and update the flow to pass the corresponding props (`onJump`, `prevId`) to each component.

## Disabling history

Since back navigation is handled via jumps rather than `onBack`, we no longer need Formity to track previous states. We can set the `history` prop to `false` to disable this.

```tsx
return (
  <Formity
    flow={flow}
    inputs={inputs}
    params={{ status }}
    history={false}
    onReturn={onReturn}
  />
);
```

## Jumping to review

When the user clicks an edit button and lands on a previous form, the submit button should jump straight back to the review step instead of continuing forward through the flow.

To support this, add an `edit` boolean to `inputs` with a default of `false`, and a `variables` element just before the review step to switch it to `true`. Then pass this value to each `Form` so it knows whether to call `onJump("review", fields)` or `onNext(fields)` on submit.

```tsx
export type Schema = {
  render: React.ReactNode;
  struct: [
    s.Jump<s.Form<Fields["personal"]>>,
    s.Jump<s.Form<Fields["background"]>>,
    s.Jump<s.Form<Fields["preferences"]>>,
    s.Jump<s.Form<Fields["about"]>>,
    s.Variables<{ edit: boolean }>,
    s.Jump<s.Form<Record<never, never>>>,
    s.Return<Values>,
  ];
  inputs: Values & { edit: boolean };
  params: { status: FormStatus };
};

export const inputs: Schema["inputs"] = {
  name: "",
  email: "",
  experience: "",
  jobTitle: "",
  employmentType: "",
  workArrangement: "",
  whyRole: "",
  greatestStrength: "",
  edit: false,
};
```
