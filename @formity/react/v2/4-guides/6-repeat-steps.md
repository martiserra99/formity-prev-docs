# Repeat steps

Learn how to repeat steps in a multi-step form.

## Initial steps

We'll show you how to repeat steps in a multi-step form. For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/react-hook-form
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

## Repeat steps

Repeating steps is useful when you need to ask the same question about each item in a dynamic list. The **loop** flow element handles this — it repeats a set of steps while a condition is `true`.

In this example, the user picks a set of technologies and rates their experience with each one.

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
    s.Form<{ technologies: string[] }>,
    s.Variables<{
      questions: Record<string, string>;
      i: number;
      results: { technology: string; experience: string }[];
    }>,
    s.Loop<
      [
        s.Variables<{ technology: string }>,
        s.Form<{ experience: string }>,
        s.Variables<{
          i: number;
          results: { technology: string; experience: string }[];
        }>,
      ]
    >,
    s.Return<{
      results: { technology: string; experience: string }[];
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
        technologies: [[], []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          key="technologies"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              technologies: z.array(z.string()).min(1, "Select at least one"),
            }),
          )}
          heading="Which technologies have you learned?"
          content={[
            {
              type: "multi-select",
              name: "technologies",
              label: "Technologies",
              options: [
                { value: "react", label: "React" },
                { value: "vue", label: "Vue" },
                { value: "angular", label: "Angular" },
                { value: "svelte", label: "Svelte" },
              ],
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
    variables: () => ({
      questions: {
        react: "What is your experience with React?",
        vue: "What is your experience with Vue?",
        angular: "What is your experience with Angular?",
        svelte: "What is your experience with Svelte?",
      },
      i: 0,
      results: [],
    }),
  },
  {
    loop: {
      while: ({ i, technologies }) => i < technologies.length,
      do: [
        {
          variables: ({ i, technologies }) => ({
            technology: technologies[i],
          }),
        },
        {
          form: {
            fields: ({ i }) => ({
              experience: ["junior", [i]],
            }),
            render: ({ fields, values, params, onBack, onNext }) => (
              <Form
                key={values.i}
                defaultValues={fields}
                resolver={zodResolver(
                  z.object({
                    experience: z.string().nonempty("Required"),
                  }),
                )}
                heading={values.questions[values.technology]}
                content={[
                  {
                    type: "select",
                    name: "experience",
                    label: "Experience level",
                    placeholder: "Select your level",
                    options: [
                      { value: "junior", label: "Junior (less than 2 years)" },
                      { value: "mid", label: "Mid-level (2–5 years)" },
                      { value: "senior", label: "Senior (more than 5 years)" },
                    ],
                  },
                ]}
                buttons={{
                  back: "Back",
                  next:
                    values.i === values.technologies.length - 1
                      ? "Submit"
                      : "Next",
                }}
                onBack={onBack}
                onNext={onNext}
                status={params.status}
              />
            ),
          },
        },
        {
          variables: ({ i, technology, experience, results }) => ({
            i: i + 1,
            results: [...results, { technology, experience }],
          }),
        },
      ],
    },
  },
  {
    return: ({ results }) => ({ results }),
  },
];

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
    await new Promise((resolve) => setTimeout(resolve, 2000));

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
    <Formity<Schema> flow={flow} params={{ status }} onReturn={onReturn} />
  );
}
```

> The form inside the loop needs a dynamic key to ensure uniqueness across iterations. Additionally, the array of the field must also change between iterations to reset the default value.
