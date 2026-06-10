# Sidebar navigation

Learn how to add a sidebar that lets users jump back to any previously completed step.

## Initial steps

We'll add the functionality to the sidebar that tracks the user's progress and lets them jump back to any step they've already completed. For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/sidebar-navigation
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

## Computing completed steps

The sidebar needs to show which steps are completed. To compute that, we need to validate the values of the multi-step form against each step's zod schema in order and stop at the first one that fails.

For that to work, the parent needs to know the values of the multi-step form, and that can be achieved by making the `Form` component notify the parent with the updated values when changes occur.

```tsx
// components/form/index.tsx
import type { DefaultValues } from "react-hook-form";
import type { OnNext, OnJump } from "@formity/react";
import type { z } from "zod";

import { useEffect, useEffectEvent } from "react";
import { useForm, FormProvider } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

import { ItemView, type Item } from "./item";
import { Button } from "../button";

interface FormProps<T extends Record<string, unknown>, U extends T> {
  defaultValues: DefaultValues<T>;
  validate: z.ZodType<T, T>;
  heading: string;
  message: string;
  content: Item[];
  buttons: { next: string; back: string | null };
  onNext: OnNext<T>;
  onJump: OnJump<T>;
  prevId: string | null;
  values: U;
  onValuesChange: (values: U) => void;
}

export function Form<T extends Record<string, unknown>, U extends T>({
  defaultValues,
  validate,
  heading,
  message,
  content,
  buttons,
  onNext,
  onJump,
  prevId,
  values,
  onValuesChange,
}: FormProps<T, U>) {
  const form = useForm({ defaultValues, resolver: zodResolver(validate) });

  const onFieldsChange = useEffectEvent(({ values: fields }: { values: T }) => {
    onValuesChange({ ...values, ...fields });
  });

  useEffect(() => {
    return form.subscribe({
      formState: { values: true },
      callback: onFieldsChange,
    });
  }, [form]);

  return (
    <form
      noValidate
      autoComplete="off"
      onSubmit={form.handleSubmit(onNext)}
      className="flex flex-1 flex-col overflow-hidden"
    >
      <FormProvider {...form}>
        <div className="flex-1 overflow-y-auto px-12 pt-12 pb-8">
          <div className="max-w-lg">
            <div className="mb-10">
              <h2 className="mb-1.5 text-2xl font-bold text-gray-950">{heading}</h2>
              <p className="text-sm font-medium text-gray-400">{message}</p>
            </div>
            <div className="mb-8 flex flex-col gap-8">
              {content.map((item, i) => (
                <ItemView key={i} {...item} />
              ))}
            </div>
            <div className="flex w-full items-center justify-end gap-8">
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
          </div>
        </div>
      </FormProvider>
    </form>
  );
}
```

Next, update `flow.tsx` to expose `values` and `onValuesChange` through `params` and pass them to each `Form`. Also export `initialValues` so the parent can initialize its state.

```tsx
// flow.tsx
import type { UnionToIntersection } from "type-fest";
import type { s, Flow } from "@formity/react";

import { z } from "zod";

import type { Steps, FormStep, ReviewStep } from "./types/steps";
import type { FormStatus } from "./types/status";

import { Form } from "./components/form";
import { Review } from "./components/review";

import * as constants from "./constants";
import * as format from "./utils/format";

type Values = UnionToIntersection<Fields[keyof Fields]>;

type Fields = {
  profile: { name: string; email: string; jobTitle: string };
  company: { companyName: string; industry: string; companySize: string };
  workspace: { workspaceName: string; timezone: string; language: string };
};

const profile: FormStep<Fields["profile"]> = {
  id: "profile",
  label: "Your profile",
  subtitle: "Name and contact details",
  zod: z.object({
    name: z.string().nonempty("Please enter your full name"),
    email: z.string().email("Please enter a valid email address"),
    jobTitle: z.string().nonempty("Please enter your job title"),
  }),
};

const company: FormStep<Fields["company"]> = {
  id: "company",
  label: "Your company",
  subtitle: "Industry and team size",
  zod: z.object({
    companyName: z.string().nonempty("Please enter your company name"),
    industry: z.string().nonempty("Please select your industry"),
    companySize: z.string().nonempty("Please select your company size"),
  }),
};

const workspace: FormStep<Fields["workspace"]> = {
  id: "workspace",
  label: "Your workspace",
  subtitle: "Name, timezone and language",
  zod: z.object({
    workspaceName: z.string().nonempty("Please enter a workspace name"),
    timezone: z.string().nonempty("Please select a timezone"),
    language: z.string().nonempty("Please select a language"),
  }),
};

const review: ReviewStep = {
  id: "review",
  label: "Review & launch",
  subtitle: "Confirm and go live",
};

export const steps: Steps = [profile, company, workspace, review];

export type Schema = {
  render: React.ReactNode;
  struct: [
    s.Jump<s.Form<Fields["profile"]>>,
    s.Jump<s.Form<Fields["company"]>>,
    s.Jump<s.Form<Fields["workspace"]>>,
    s.Jump<s.Form<Record<never, never>>>,
    s.Return<Values>,
  ];
  inputs: Values;
  params: {
    status: FormStatus;
    values: Values;
    onValuesChange: (values: Values) => void;
  };
};

export const flow: Flow<Schema> = [
  {
    jump: {
      id: profile.id,
      at: {
        form: {
          fields: (values) => ({
            name: [values.name, []],
            email: [values.email, []],
            jobTitle: [values.jobTitle, []],
          }),
          render: ({ fields, params, onNext, onJump }) => (
            <Form
              key={profile.id}
              defaultValues={fields}
              validate={profile.zod}
              heading="Tell us about yourself"
              message="We'll use this to personalise your experience."
              content={[
                { type: "input", name: "name", label: "Full name", placeholder: "Jane Smith" },
                { type: "input", name: "email", label: "Work email", placeholder: "jane@company.com", inputType: "email" },
                { type: "input", name: "jobTitle", label: "Job title", placeholder: "Product Manager" },
              ]}
              buttons={{ back: null, next: "Continue" }}
              onNext={onNext}
              onJump={onJump}
              prevId={null}
              values={params.values}
              onValuesChange={params.onValuesChange}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: company.id,
      at: {
        form: {
          fields: (values) => ({
            companyName: [values.companyName, []],
            industry: [values.industry, []],
            companySize: [values.companySize, []],
          }),
          render: ({ fields, params, onNext, onJump }) => (
            <Form
              key={company.id}
              defaultValues={fields}
              validate={company.zod}
              heading="About your company"
              message="Help us understand the context you're working in."
              content={[
                { type: "input", name: "companyName", label: "Company name", placeholder: "Acme Inc." },
                { type: "select", name: "industry", label: "Industry", placeholder: "Select an industry", options: constants.industries },
                { type: "select", name: "companySize", label: "Company size", placeholder: "Select a size", options: constants.companySizes },
              ]}
              buttons={{ back: "Back", next: "Continue" }}
              onNext={onNext}
              onJump={onJump}
              prevId={profile.id}
              values={params.values}
              onValuesChange={params.onValuesChange}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: workspace.id,
      at: {
        form: {
          fields: (values) => ({
            workspaceName: [values.workspaceName, []],
            timezone: [values.timezone, []],
            language: [values.language, []],
          }),
          render: ({ fields, params, onNext, onJump }) => (
            <Form
              key={workspace.id}
              defaultValues={fields}
              validate={workspace.zod}
              heading="Set up your workspace"
              message="Give your workspace a name and configure your region."
              content={[
                { type: "input", name: "workspaceName", label: "Workspace name", placeholder: "My Workspace" },
                { type: "select", name: "timezone", label: "Timezone", placeholder: "Select a timezone", options: constants.timezones },
                { type: "select", name: "language", label: "Language", placeholder: "Select a language", options: constants.languages },
              ]}
              buttons={{ back: "Back", next: "Continue" }}
              onNext={onNext}
              onJump={onJump}
              prevId={company.id}
              values={params.values}
              onValuesChange={params.onValuesChange}
            />
          ),
        },
      },
    },
  },
  {
    jump: {
      id: review.id,
      at: {
        form: {
          fields: () => ({}),
          render: ({ values, params, onNext, onJump }) => (
            <Review
              key={review.id}
              heading="Review & launch"
              message="Everything look right? Launch your workspace."
              content={[
                { text: "Profile", rows: [{ label: "Full name", value: format.text(values.name) }, { label: "Work email", value: format.text(values.email) }, { label: "Job title", value: format.text(values.jobTitle) }] },
                { text: "Company", rows: [{ label: "Company name", value: format.text(values.companyName) }, { label: "Industry", value: format.industry(values.industry) }, { label: "Company size", value: format.companySize(values.companySize) }] },
                { text: "Workspace", rows: [{ label: "Workspace name", value: format.text(values.workspaceName) }, { label: "Timezone", value: format.timezone(values.timezone) }, { label: "Language", value: format.language(values.language) }] },
              ]}
              buttons={{ back: "Back", next: "Launch workspace" }}
              onNext={onNext}
              onJump={onJump}
              prevId={workspace.id}
              status={params.status}
            />
          ),
        },
      },
    },
  },
  {
    return: (values) => values,
  },
];

export const initialValues: Values = {
  name: "",
  email: "",
  jobTitle: "",
  companyName: "",
  industry: "",
  companySize: "",
  workspaceName: "",
  timezone: "",
  language: "",
};
```

Now update `app.tsx` to hold the form values in state and derive how many steps are completed.

```tsx
// app.tsx
import type { OnReturn, ReturnOutput } from "@formity/react";

import { useState, useCallback, useMemo } from "react";
import { useFormity } from "@formity/react";

import type { Status, FormStatus } from "./types/status";
import type { FormStep } from "./types/steps";

import { Sidebar } from "./components/sidebar";
import { Done } from "./components/done";

import { steps, flow, initialValues, type Schema } from "./flow";

export default function App() {
  const [status, setStatus] = useState<Status<ReturnOutput<Schema>>>({
    type: "form",
    submitting: false,
  });

  const onReturn = useCallback<OnReturn<Schema>>(async (output) => {
    setStatus({ type: "form", submitting: true });
    console.log(output);
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

  return <Form status={status} onReturn={onReturn} />;
}

interface FormProps {
  status: FormStatus;
  onReturn: OnReturn<Schema>;
}

function Form({ status, onReturn }: FormProps) {
  const [values, setValues] = useState(initialValues);

  const form = useFormity({
    flow,
    params: { status, values, onValuesChange: setValues },
    history: false,
    onReturn,
  });

  const completed = useMemo(() => {
    for (let i = 0; i < steps.length - 1; i++) {
      const step = steps[i] as FormStep;
      if (!step.zod.safeParse(values).success) return i;
    }
    return steps.length - 1;
  }, [values]);

  return (
    <div className="@container flex h-screen w-full overflow-hidden bg-white">
      <Sidebar steps={steps} current={0} completed={completed} onJump={() => {}} />
      <main className="flex flex-1 flex-col overflow-hidden">{form}</main>
    </div>
  );
}
```

## Getting the current step

The sidebar also needs to know which step is active. Update `flow` so each form returns an object with a `step` index alongside the rendered element, and update the `Schema` type accordingly.

```tsx
export type Schema = {
  render: { step: number; form: React.ReactNode };
  // ...
};
```

Then destructure `step` and `form` from `useFormity` and pass the step to the `Sidebar`.

```tsx
const { step: current, form } = useFormity({
  flow,
  params: { status, values, onValuesChange: setValues },
  history: false,
  onReturn,
});

// ...

<Sidebar steps={steps} current={current} completed={completed} onJump={() => {}} />
```

## Jumping with the sidebar

Since `onJump` can only be called from inside form elements, we can't call it directly from the sidebar. Instead, we use `useImperativeHandle` to expose a `jump` method on a ref, which the sidebar can call.

Update `Form` to accept a ref and expose that method through it:

```tsx
interface FormProps<T extends Record<string, unknown>, U extends T> {
  // ...
  ref: React.Ref<{ jump: (id: string) => void }>;
}

export function Form<T extends Record<string, unknown>, U extends T>({
  // ...
  ref,
}: FormProps<T, U>) {
  // ...

  useImperativeHandle(
    ref,
    () => ({
      jump: (id) => onJump(id, form.getValues()),
    }),
    [form, onJump],
  );

  // ...
}
```

Do the same for the `Review` component. Then add `ref` to `params` in the schema so the flow can forward it to each component.

```tsx
params: {
  status: FormStatus;
  values: Values;
  onValuesChange: (values: Values) => void;
  ref: React.Ref<{ jump: (id: string) => void }>;
};
```

Finally, create the ref in `app.tsx`, pass it via `params`, and call the function when the user clicks a sidebar step.

```tsx
function Form({ status, onReturn }: FormProps) {
  const [values, setValues] = useState(initialValues);
  const ref = useRef<{ jump: (id: string) => void }>(null);

  const { step: current, form } = useFormity({
    flow,
    params: { status, values, onValuesChange: setValues, ref },
    history: false,
    onReturn,
  });

  const completed = useMemo(() => {
    for (let i = 0; i < steps.length - 1; i++) {
      const step = steps[i] as FormStep;
      if (!step.zod.safeParse(values).success) return i;
    }
    return steps.length - 1;
  }, [values]);

  return (
    <div className="@container flex h-screen w-full overflow-hidden bg-white">
      <Sidebar
        steps={steps}
        current={current}
        completed={completed}
        onJump={(step) => ref.current?.jump(step)}
      />
      <main className="flex flex-1 flex-col overflow-hidden">{form}</main>
    </div>
  );
}
```
