# Conditional fields

Learn how to add fields that appear when a condition is met.

## Initial steps

We'll show you how to add fields that appear when a condition is met. For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/react-hook-form
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

## Conditional fields

To add a conditional field, start by creating the following component.

```tsx
// components/form/item/condition.tsx
import { useFormContext } from "react-hook-form";

import { ItemView, type Item } from ".";

export interface Condition {
  type: "condition";
  if: (values: unknown) => boolean;
  watch: string[];
  items: Item[];
}

export function ConditionView(props: Condition) {
  const { watch } = useFormContext();
  const variables = watch(props.watch).reduce(
    (acc, value, index) => ({ ...acc, [props.watch[index]]: value }),
    {},
  );
  if (props.if(variables)) {
    return props.items.map((item, index) => <ItemView key={index} {...item} />);
  }
  return null;
}
```

We also need to update the following file to include this new component.

```tsx
// components/form/item/index.tsx
import { ConditionView, type Condition } from "./condition";
import { ColumnsView, type Columns } from "./columns";
import { InputView, type Input } from "./input";
import { NumberView, type Number } from "./number";
import { SelectView, type Select } from "./select";
import { TextareaView, type Textarea } from "./textarea";
import { MultiSelectView, type MultiSelect } from "./multi-select";

export type Item =
  | Condition
  | Columns
  | Input
  | Number
  | Select
  | Textarea
  | MultiSelect;

export function ItemView(item: Item) {
  switch (item.type) {
    case "condition": {
      return <ConditionView {...item} />;
    }
    case "columns": {
      return <ColumnsView {...item} />;
    }
    case "input": {
      return <InputView {...item} />;
    }
    case "number": {
      return <NumberView {...item} />;
    }
    case "select": {
      return <SelectView {...item} />;
    }
    case "textarea": {
      return <TextareaView {...item} />;
    }
    case "multi-select": {
      return <MultiSelectView {...item} />;
    }
  }
}
```

Then, we can update the flow with the code shown below.

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
    s.Form<{ working: string; company: string }>,
    s.Variables<{ company: string | null }>,
    s.Return<{ working: string; company: string | null }>,
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
        working: ["no", []],
        company: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          key="yourself"
          defaultValues={fields}
          resolver={zodResolver(
            z
              .object({
                working: z.string(),
                company: z.string(),
              })
              .superRefine((data, ctx) => {
                if (data.working === "yes") {
                  if (data.company === "") {
                    ctx.addIssue({
                      code: "custom",
                      message: "Required",
                      path: ["company"],
                    });
                  }
                }
              }),
          )}
          heading="Tell us about yourself"
          content={[
            {
              type: "select",
              name: "working",
              label: "Are you working?",
              placeholder: "Select an option",
              options: [
                { value: "yes", label: "Yes" },
                { value: "no", label: "No" },
              ],
            },
            {
              type: "condition",
              if: ({ working }: { working: string }) => working === "yes",
              watch: ["working"],
              items: [
                {
                  type: "input",
                  name: "company",
                  label: "At what company?",
                  placeholder: "Company name",
                },
              ],
            },
          ]}
          buttons={{
            back: null,
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
    variables: ({ working, company }) => ({
      company: working === "yes" ? company : null,
    }),
  },
  {
    return: ({ working, company }) => ({
      working,
      company,
    }),
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

We need to apply validation rules to the conditional field only when its condition is met. Additionally, we should set a default value for the field when it's hidden, using a variable.
