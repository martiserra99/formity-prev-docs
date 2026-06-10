# Tutorial

Follow this tutorial to grasp the core concepts of Formity and how it has to be used.

## Initial steps

We'll show you how to turn a single-step form into a dynamic multi-step form with conditional logic. For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/tutorial
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

## Single-step form

If you take a look at the `app.tsx` file, you'll find a single-step form already in place. This form is built using [React Hook Form](https://react-hook-form.com/). However, you're not restricted to this library. Formity is designed to work smoothly with any single-step form library you choose.

```tsx
// app.tsx
import { useCallback, useState } from "react";

import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

import { Form } from "./components/form";
import { Output } from "./components/output";

type Values = { name: string; surname: string; age: number };

export default function App() {
  const [output, setOutput] = useState<Values | null>(null);

  const onSubmit = useCallback((output: Values) => {
    setOutput(output);
  }, []);

  if (output) {
    return <Output output={output} onStartOver={() => setOutput(null)} />;
  }

  return (
    <Form
      defaultValues={{
        name: "",
        surname: "",
        age: 20,
      }}
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
      submit="Submit"
      onSubmit={onSubmit}
    />
  );
}
```

## Multi-step form

To get started with Formity, the first thing we'll do is use the `Formity` component. It is the one that renders the multi-step form, and these are the most important props:

- `flow`: Defines the structure and behavior of the multi-step form.
- `onReturn`: A callback function that is triggered when the form is completed.

We'll replace the code that we have in `app.tsx` with the following code.

```tsx
// app.tsx
import { useCallback, useState } from "react";

import {
  Formity,
  type Flow,
  type OnReturn,
  type ReturnOutput,
} from "@formity/react";

import { Output } from "./components/output";

type Schema = {
  render: React.ReactNode;
  struct: [];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [];

export default function App() {
  const [output, setOutput] = useState<ReturnOutput<Schema> | null>(null);

  const onReturn = useCallback<OnReturn<Schema>>((output) => {
    setOutput(output);
  }, []);

  if (output) {
    return <Output output={output} onStartOver={() => setOutput(null)} />;
  }

  return <Formity<Schema> flow={flow} onReturn={onReturn} />;
}
```

The next step is to create the flow, which defines the structure and behavior of the multi-step form. We'll do this by writing the following code.

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

import { Form } from "./components/form";
import { Output } from "./components/output";

type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Form<{ softwareDeveloper: string }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, onNext }) => (
        <Form
          key="yourself"
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
          submit="Next"
          onSubmit={onNext}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        softwareDeveloper: ["", []],
      }),
      render: ({ fields, onNext }) => (
        <Form
          key="softwareDeveloper"
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
          submit="Next"
          onSubmit={onNext}
        />
      ),
    },
  },
];

export default function App() {
  const [output, setOutput] = useState<ReturnOutput<Schema> | null>(null);

  const onReturn = useCallback<OnReturn<Schema>>((output) => {
    setOutput(output);
  }, []);

  if (output) {
    return <Output output={output} onStartOver={() => setOutput(null)} />;
  }

  return <Formity<Schema> flow={flow} onReturn={onReturn} />;
}
```

The `flow` constant is an array of type `Flow`. There are different types of elements you can use within the flow, and in this example, we've included two form elements.

Additionally, to ensure complete type safety, the `Flow` accepts a `Schema` type that defines the structure and values handled at each step.

If you complete the multi-step form, you'll see that the `onReturn` callback is not called. That's because we need to add a return element to the flow, as shown below.

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

import { Form } from "./components/form";
import { Output } from "./components/output";

type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<{ name: string; surname: string; age: number }>,
    s.Form<{ softwareDeveloper: string }>,
    s.Return<{
      name: string;
      surname: string;
      age: number;
      softwareDeveloper: string;
    }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, onNext }) => (
        <Form
          key="yourself"
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
          submit="Next"
          onSubmit={onNext}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        softwareDeveloper: ["", []],
      }),
      render: ({ fields, onNext }) => (
        <Form
          key="softwareDeveloper"
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
          submit="Next"
          onSubmit={onNext}
        />
      ),
    },
  },
  {
    return: ({ name, surname, age, softwareDeveloper }) => ({
      name,
      surname,
      age,
      softwareDeveloper,
    }),
  },
];

export default function App() {
  const [output, setOutput] = useState<ReturnOutput<Schema> | null>(null);

  const onReturn = useCallback<OnReturn<Schema>>((output) => {
    setOutput(output);
  }, []);

  if (output) {
    return <Output output={output} onStartOver={() => setOutput(null)} />;
  }

  return <Formity<Schema> flow={flow} onReturn={onReturn} />;
}
```

So far, we've covered the form and return elements. However, Formity supports additional elements that allow you to build any logic you need. One of these is the condition, which can be used like this.

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

import { Form } from "./components/form";
import { Output } from "./components/output";

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
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, onNext }) => (
        <Form
          key="yourself"
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
          submit="Next"
          onSubmit={onNext}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        softwareDeveloper: ["", []],
      }),
      render: ({ fields, onNext }) => (
        <Form
          key="softwareDeveloper"
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
          submit="Next"
          onSubmit={onNext}
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
            render: ({ fields, onNext }) => (
              <Form
                key="expertise"
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
                submit="Submit"
                onSubmit={onNext}
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
            render: ({ fields, onNext }) => (
              <Form
                key="interested"
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
                submit="Submit"
                onSubmit={onNext}
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
  const [output, setOutput] = useState<ReturnOutput<Schema> | null>(null);

  const onReturn = useCallback<OnReturn<Schema>>((output) => {
    setOutput(output);
  }, []);

  if (output) {
    return <Output output={output} onStartOver={() => setOutput(null)} />;
  }

  return <Formity<Schema> flow={flow} onReturn={onReturn} />;
}
```

We also need to include the logic to navigate to previous steps. To enable this functionality we have to update the `Form` component as shown below.

```tsx
// components/form/index.tsx
import type { DefaultValues, Resolver } from "react-hook-form";
import type { OnBack, OnNext } from "@formity/react";

import { useForm, FormProvider } from "react-hook-form";

import { ItemView, type Item } from "./item";
import { Button } from "../button";

interface FormProps<T extends Record<string, unknown>> {
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
}

export function Form<T extends Record<string, unknown>>({
  defaultValues,
  resolver,
  heading,
  content,
  buttons,
  onBack,
  onNext,
}: FormProps<T>) {
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
                onClick={() => onBack(form.getValues())}
              >
                {buttons.back}
              </Button>
            )}
            <Button type="submit" variant="primary">
              {buttons.next}
            </Button>
          </div>
        </div>
      </FormProvider>
    </form>
  );
}
```

Then, we need to update the flow as shown below.

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

import { Form } from "./components/form";
import { Output } from "./components/output";

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
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
        surname: ["", []],
        age: [20, []],
      }),
      render: ({ fields, onBack, onNext }) => (
        <Form
          key="yourself"
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
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        softwareDeveloper: ["", []],
      }),
      render: ({ fields, onBack, onNext }) => (
        <Form
          key="softwareDeveloper"
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
            render: ({ fields, onBack, onNext }) => (
              <Form
                key="expertise"
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
            render: ({ fields, onBack, onNext }) => (
              <Form
                key="interested"
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
  const [output, setOutput] = useState<ReturnOutput<Schema> | null>(null);

  const onReturn = useCallback<OnReturn<Schema>>((output) => {
    setOutput(output);
  }, []);

  if (output) {
    return <Output output={output} onStartOver={() => setOutput(null)} />;
  }

  return <Formity<Schema> flow={flow} onReturn={onReturn} />;
}
```

## Submit form

Until now, we have shown the values we want to submit. Now, we need to create the logic to submit these values and keep track of the submission state.

To do this, we'll start by creating a `types/status.ts` file with the following code.

```ts
// types/status.ts
export type Status = FormStatus | DoneStatus;

export type FormStatus = {
  type: "form";
  submitting: boolean;
};

export type DoneStatus = {
  type: "done";
};
```

Then, we'll update the `app.tsx` file to include the submission logic and display a thank you screen once the values have been successfully submitted.

```tsx
// app.tsx
import { useCallback, useState } from "react";

import { Formity, type s, type Flow, type OnReturn } from "@formity/react";

import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

import type { Status } from "./types/status";

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
          key="yourself"
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
          key="softwareDeveloper"
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
                key="expertise"
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
                key="interested"
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
  const [status, setStatus] = useState<Status>({
    type: "form",
    submitting: false,
  });

  const onReturn = useCallback<OnReturn<Schema>>(async (output) => {
    setStatus({ type: "form", submitting: true });

    // Show output in the console
    console.log(output);

    // Simulate a network request
    await new Promise((resolve) => setTimeout(resolve, 2000));

    setStatus({ type: "done" });
  }, []);

  if (status.type === "done") {
    return (
      <Done
        onStartOver={() => setStatus({ type: "form", submitting: false })}
      />
    );
  }

  return (
    <Formity<Schema> flow={flow} params={{ status }} onReturn={onReturn} />
  );
}
```
