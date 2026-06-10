# Animate forms

Learn how to add animations to a multi-step form using Motion.

## Initial steps

We'll show you how to add animations to a multi-step form using [Motion](https://motion.dev/). For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/react-hook-form
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

Finally, install Motion.

```bash
npm install motion
```

## Animate form

We'll update the `FormStatus` type to track whether the user is navigating and in which direction.

```tsx
// types/status.ts
export type Status<T> = FormStatus | DoneStatus<T>;

export type FormStatus = {
  type: "form";
  move: "next" | "back" | false;
  submitting: boolean;
};

export type DoneStatus<T> = {
  type: "done";
  output: T;
};
```

Then, we'll update the `Form` component so that when we move to the next or previous step, the status changes and the corresponding navigation function is called.

We'll also use `AnimatePresence` and a `motion.div` to animate steps in and out.

```tsx
// components/form/index.tsx
import type { DefaultValues, Resolver } from "react-hook-form";
import type { OnBack, OnNext } from "@formity/react";
import type { MotionProps } from "motion/react";

import {
  useMemo,
  useState,
  useEffect,
  useCallback,
  useEffectEvent,
} from "react";

import { useForm, FormProvider } from "react-hook-form";
import { AnimatePresence, motion } from "motion/react";

import type { FormStatus } from "@/types/status";

import { ItemView, type Item } from "./item";
import { Button } from "../button";

interface FormProps<T extends Record<string, unknown>> {
  id: string;
  defaultValues: DefaultValues<T>;
  resolver: Resolver<T>;
  heading: string;
  content: Item[];
  buttons: {
    back: string | null;
    next: string;
  };
  onBack: OnBack<T>;
  onNext: OnNext<T>;
  status: FormStatus;
  setStatus: (status: FormStatus) => void;
}

export function Form<T extends Record<string, unknown>>({
  id,
  onBack,
  onNext,
  status,
  setStatus,
  ...rest
}: FormProps<T>) {
  const [fields, setFields] = useState<T>();

  const move = useEffectEvent((move: FormStatus["move"]) => {
    if (move === "next") return onNext(fields);
    if (move === "back") return onBack(fields);
  });

  useEffect(() => move(status.move), [status.move]);

  const handleNext = useCallback<OnNext<T>>(
    (fields) => {
      setStatus({ type: "form", move: "next", submitting: false });
      setFields(fields);
    },
    [setStatus, setFields],
  );

  const handleBack = useCallback<OnBack<T>>(
    (fields) => {
      setStatus({ type: "form", move: "back", submitting: false });
      setFields(fields);
    },
    [setStatus, setFields],
  );

  const animate = useMemo(
    () => ({ x: 0, opacity: 1, transition: { delay: 0.25, duration: 0.25 } }),
    [],
  );

  return (
    <AnimatePresence mode="popLayout" initial={false}>
      <motion.div
        key={id}
        inert={Boolean(status.move)}
        animate={animate}
        onAnimationComplete={(definition) => {
          if (definition === animate) {
            setStatus({ type: "form", move: false, submitting: false });
          }
        }}
        {...motionProps(status.move)}
        className="h-full"
      >
        <View
          key={id}
          onBack={handleBack}
          onNext={handleNext}
          status={status}
          {...rest}
        />
      </motion.div>
    </AnimatePresence>
  );
}

function motionProps(move: FormStatus["move"]): MotionProps {
  if (move === "next") {
    return {
      initial: { x: 50, opacity: 0 },
      exit: { x: -50, opacity: 0, transition: { delay: 0, duration: 0.25 } },
    };
  }
  if (move === "back") {
    return {
      initial: { x: -50, opacity: 0 },
      exit: { x: 50, opacity: 0, transition: { delay: 0, duration: 0.25 } },
    };
  }
  return {};
}

function View<T extends Record<string, unknown>>({
  defaultValues,
  resolver,
  heading,
  content,
  buttons,
  onBack,
  onNext,
  status,
}: Omit<FormProps<T>, "id" | "setStatus">) {
  const form = useForm({ defaultValues, resolver });
  return (
    <form
      onSubmit={form.handleSubmit(onNext)}
      className="flex h-screen w-full items-center justify-center px-4 py-8"
      autoComplete="off"
    >
      <FormProvider {...form}>
        <div className="w-full max-w-md">
          <h2 className="mb-6 text-center text-4xl font-bold text-gray-950">
            {heading}
          </h2>
          <div className="mb-6 flex flex-col gap-4">
            {content.map((field, index) => (
              <ItemView key={index} {...field} />
            ))}
          </div>
          <div className="flex gap-4">
            {buttons.back && (
              <Button
                type="button"
                variant="secondary"
                disabled={status.submitting}
                onClick={() => onBack(form.getValues())}
              >
                {buttons.back}
              </Button>
            )}
            <Button
              type="submit"
              variant="primary"
              disabled={status.submitting}
            >
              {status.submitting ? "Submitting..." : buttons.next}
            </Button>
          </div>
        </div>
      </FormProvider>
    </form>
  );
}
```

Finally, we'll update `app.tsx` to include the new props in `Form` and update the status.

```tsx
// app.tsx
import { useCallback, useState } from "react";

import {
  Formity,
  type s,
  type Flow,
  type OnReturn,
  type ReturnOutput,
} from "@formity/react";

import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

import type { Status, FormStatus } from "./types/status";

import { Form } from "./components/form";
import { Done } from "./components/done";

type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Form<{ softwareDeveloper: string }>,
    s.Condition<{
      then: [
        s.Form<{ expertise: string }>,
        s.Return<{
          name: string;
          surname: string;
          age: number;
          softwareDeveloper: true;
          expertise: string;
        }>,
      ];
      else: [
        s.Form<{ interested: string }>,
        s.Return<{
          name: string;
          surname: string;
          age: number;
          softwareDeveloper: false;
          interested: string;
        }>,
      ];
    }>,
  ];
  inputs: Record<never, never>;
  params: {
    status: FormStatus;
    setStatus: (status: FormStatus) => void;
  };
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          id="yourself"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              name: z.string().nonempty("Required"),
              surname: z.string().nonempty("Required"),
              age: z.number().min(18, "Min. 18").max(99, "Max. 99"),
            }),
          )}
          heading="Tell us about yourself"
          content={[
            {
              type: "columns",
              columns: [
                {
                  type: "input",
                  name: "name",
                  label: "Name",
                  placeholder: "Your name",
                },
                {
                  type: "input",
                  name: "surname",
                  label: "Surname",
                  placeholder: "Your surname",
                },
              ],
            },
            {
              type: "number",
              name: "age",
              label: "Age",
              placeholder: "Your age",
            },
          ]}
          buttons={{
            back: null,
            next: "Next",
          }}
          onBack={onBack}
          onNext={onNext}
          status={params.status}
          setStatus={params.setStatus}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        softwareDeveloper: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          id="softwareDeveloper"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              softwareDeveloper: z.string().nonempty("Required"),
            }),
          )}
          heading="Are you a software developer?"
          content={[
            {
              type: "select",
              name: "softwareDeveloper",
              label: "Software Developer",
              placeholder: "Select an option",
              options: [
                { value: "yes", label: "Yes" },
                { value: "no", label: "No" },
              ],
            },
          ]}
          buttons={{
            back: "Back",
            next: "Next",
          }}
          onBack={onBack}
          onNext={onNext}
          status={params.status}
          setStatus={params.setStatus}
        />
      ),
    },
  },
  {
    condition: {
      if: ({ softwareDeveloper }) => softwareDeveloper === "yes",
      then: [
        {
          form: {
            fields: () => ({
              expertise: ["", []],
            }),
            render: ({ fields, params, onBack, onNext }) => (
              <Form
                id="expertise"
                defaultValues={fields}
                resolver={zodResolver(
                  z.object({
                    expertise: z.string().nonempty("Required"),
                  }),
                )}
                heading="What is your area of expertise?"
                content={[
                  {
                    type: "select",
                    name: "expertise",
                    label: "Expertise",
                    placeholder: "Select an option",
                    options: [
                      { value: "frontend", label: "Frontend development" },
                      { value: "backend", label: "Backend development" },
                      { value: "mobile", label: "Mobile development" },
                    ],
                  },
                ]}
                buttons={{
                  back: "Back",
                  next: "Submit",
                }}
                onBack={onBack}
                onNext={onNext}
                status={params.status}
                setStatus={params.setStatus}
              />
            ),
          },
        },
        {
          return: ({ name, surname, age, expertise }) => ({
            name,
            surname,
            age,
            softwareDeveloper: true,
            expertise,
          }),
        },
      ],
      else: [
        {
          form: {
            fields: () => ({
              interested: ["", []],
            }),
            render: ({ fields, params, onBack, onNext }) => (
              <Form
                id="interested"
                defaultValues={fields}
                resolver={zodResolver(
                  z.object({
                    interested: z.string().nonempty("Required"),
                  }),
                )}
                heading="Are you interested in learning how to code?"
                content={[
                  {
                    type: "select",
                    name: "interested",
                    label: "Interested",
                    placeholder: "Select an option",
                    options: [
                      { value: "yes", label: "Yes, I am interested." },
                      { value: "no", label: "No, it is not for me." },
                      { value: "maybe", label: "Maybe, I am not sure." },
                    ],
                  },
                ]}
                buttons={{
                  back: "Back",
                  next: "Submit",
                }}
                onBack={onBack}
                onNext={onNext}
                status={params.status}
                setStatus={params.setStatus}
              />
            ),
          },
        },
        {
          return: ({ name, surname, age, interested }) => ({
            name,
            surname,
            age,
            softwareDeveloper: false,
            interested,
          }),
        },
      ],
    },
  },
];

export default function App() {
  const [status, setStatus] = useState<Status<ReturnOutput<Schema>>>({
    type: "form",
    move: false,
    submitting: false,
  });

  const onReturn = useCallback<OnReturn<Schema>>(async (output) => {
    setStatus({ type: "form", move: false, submitting: true });

    // Show output in the console
    console.log(output);

    // Simulate a network request
    await new Promise((resolve) => setTimeout(resolve, 2000));

    setStatus({ type: "done", output });
  }, []);

  if (status.type === "done") {
    return (
      <Done
        output={status.output}
        onStartOver={() =>
          setStatus({ type: "form", move: false, submitting: false })
        }
      />
    );
  }

  return (
    <Formity<Schema>
      flow={flow}
      params={{ status, setStatus }}
      onReturn={onReturn}
    />
  );
}
```

## Progress bar

We can also add a progress bar. The `useFormity` hook makes this possible by letting each step return any data, not just a `ReactNode`.

```tsx
// app.tsx
import { useCallback, useState } from "react";

import {
  useFormity,
  type s,
  type Flow,
  type OnReturn,
  type ReturnOutput,
} from "@formity/react";

import { motion } from "motion/react";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

import type { Status, FormStatus } from "./types/status";

import { Form } from "./components/form";
import { Done } from "./components/done";

type Schema = {
  render: {
    progress: {
      numberSteps: number;
      currentStep: number;
    };
    form: React.ReactNode;
  };
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Form<{ softwareDeveloper: string }>,
    s.Condition<{
      then: [
        s.Form<{ expertise: string }>,
        s.Return<{
          name: string;
          surname: string;
          age: number;
          softwareDeveloper: true;
          expertise: string;
        }>,
      ];
      else: [
        s.Form<{ interested: string }>,
        s.Return<{
          name: string;
          surname: string;
          age: number;
          softwareDeveloper: false;
          interested: string;
        }>,
      ];
    }>,
  ];
  inputs: Record<never, never>;
  params: {
    status: FormStatus;
    setStatus: (status: FormStatus) => void;
  };
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, params, onBack, onNext }) => ({
        progress: {
          numberSteps: 3,
          currentStep: 1,
        },
        form: (
          <Form
            id="yourself"
            defaultValues={fields}
            resolver={zodResolver(
              z.object({
                name: z.string().nonempty("Required"),
                surname: z.string().nonempty("Required"),
                age: z.number().min(18, "Min. 18").max(99, "Max. 99"),
              }),
            )}
            heading="Tell us about yourself"
            content={[
              {
                type: "columns",
                columns: [
                  {
                    type: "input",
                    name: "name",
                    label: "Name",
                    placeholder: "Your name",
                  },
                  {
                    type: "input",
                    name: "surname",
                    label: "Surname",
                    placeholder: "Your surname",
                  },
                ],
              },
              {
                type: "number",
                name: "age",
                label: "Age",
                placeholder: "Your age",
              },
            ]}
            buttons={{
              back: null,
              next: "Next",
            }}
            onBack={onBack}
            onNext={onNext}
            status={params.status}
            setStatus={params.setStatus}
          />
        ),
      }),
    },
  },
  {
    form: {
      fields: () => ({
        softwareDeveloper: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => ({
        progress: {
          numberSteps: 3,
          currentStep: 2,
        },
        form: (
          <Form
            id="softwareDeveloper"
            defaultValues={fields}
            resolver={zodResolver(
              z.object({
                softwareDeveloper: z.string().nonempty("Required"),
              }),
            )}
            heading="Are you a software developer?"
            content={[
              {
                type: "select",
                name: "softwareDeveloper",
                label: "Software Developer",
                placeholder: "Select an option",
                options: [
                  { value: "yes", label: "Yes" },
                  { value: "no", label: "No" },
                ],
              },
            ]}
            buttons={{
              back: "Back",
              next: "Next",
            }}
            onBack={onBack}
            onNext={onNext}
            status={params.status}
            setStatus={params.setStatus}
          />
        ),
      }),
    },
  },
  {
    condition: {
      if: ({ softwareDeveloper }) => softwareDeveloper === "yes",
      then: [
        {
          form: {
            fields: () => ({
              expertise: ["", []],
            }),
            render: ({ fields, params, onBack, onNext }) => ({
              progress: {
                numberSteps: 3,
                currentStep: 3,
              },
              form: (
                <Form
                  id="expertise"
                  defaultValues={fields}
                  resolver={zodResolver(
                    z.object({
                      expertise: z.string().nonempty("Required"),
                    }),
                  )}
                  heading="What is your area of expertise?"
                  content={[
                    {
                      type: "select",
                      name: "expertise",
                      label: "Expertise",
                      placeholder: "Select an option",
                      options: [
                        { value: "frontend", label: "Frontend development" },
                        { value: "backend", label: "Backend development" },
                        { value: "mobile", label: "Mobile development" },
                      ],
                    },
                  ]}
                  buttons={{
                    back: "Back",
                    next: "Submit",
                  }}
                  onBack={onBack}
                  onNext={onNext}
                  status={params.status}
                  setStatus={params.setStatus}
                />
              ),
            }),
          },
        },
        {
          return: ({ name, surname, age, expertise }) => ({
            name,
            surname,
            age,
            softwareDeveloper: true,
            expertise,
          }),
        },
      ],
      else: [
        {
          form: {
            fields: () => ({
              interested: ["", []],
            }),
            render: ({ fields, params, onBack, onNext }) => ({
              progress: {
                numberSteps: 3,
                currentStep: 3,
              },
              form: (
                <Form
                  id="interested"
                  defaultValues={fields}
                  resolver={zodResolver(
                    z.object({
                      interested: z.string().nonempty("Required"),
                    }),
                  )}
                  heading="Are you interested in learning how to code?"
                  content={[
                    {
                      type: "select",
                      name: "interested",
                      label: "Interested",
                      placeholder: "Select an option",
                      options: [
                        { value: "yes", label: "Yes, I am interested." },
                        { value: "no", label: "No, it is not for me." },
                        { value: "maybe", label: "Maybe, I am not sure." },
                      ],
                    },
                  ]}
                  buttons={{
                    back: "Back",
                    next: "Submit",
                  }}
                  onBack={onBack}
                  onNext={onNext}
                  status={params.status}
                  setStatus={params.setStatus}
                />
              ),
            }),
          },
        },
        {
          return: ({ name, surname, age, interested }) => ({
            name,
            surname,
            age,
            softwareDeveloper: false,
            interested,
          }),
        },
      ],
    },
  },
];

export default function App() {
  const [status, setStatus] = useState<Status<ReturnOutput<Schema>>>({
    type: "form",
    move: false,
    submitting: false,
  });

  const onReturn = useCallback<OnReturn<Schema>>(async (output) => {
    setStatus({ type: "form", move: false, submitting: true });

    // Show output in the console
    console.log(output);

    // Simulate a network request
    await new Promise((resolve) => setTimeout(resolve, 2000));

    setStatus({ type: "done", output });
  }, []);

  if (status.type === "done") {
    return (
      <Done
        output={status.output}
        onStartOver={() =>
          setStatus({ type: "form", move: false, submitting: false })
        }
      />
    );
  }

  return <Formity params={{ status, setStatus }} onReturn={onReturn} />;
}

interface FormityProps {
  params: Schema["params"];
  onReturn: OnReturn<Schema>;
}

function Formity({ params, onReturn }: FormityProps) {
  const { progress, form } = useFormity({ flow, params, onReturn });
  return (
    <div className="relative h-full">
      <div className="absolute inset-x-0 top-0 z-10 h-1.5 bg-gray-200">
        <motion.div
          initial={false}
          animate={{
            transform: `scaleX(${progress.currentStep / progress.numberSteps})`,
          }}
          className="h-full origin-left bg-gray-900"
        />
      </div>
      {form}
    </div>
  );
}
```
