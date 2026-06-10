# State outside the form

Learn how to access the form state outside the form to display a live summary as the user fills it in.

## Initial steps

We'll show you how to track the form state outside the form so you can use it anywhere — for example, to show a live summary panel alongside the form. For that, you will need to navigate to the following folder in the `formity-prev-docs-code` repository. Clone it first if you haven't already.

```bash
git clone https://github.com/martiserra99/formity-prev-docs-code
cd formity-prev-docs-code/@formity/react/v2/state-outside-the-form
```

Make sure you run the following command to install all the dependencies.

```bash
npm install
```

## Watching value changes

We need to update the `Form` component to receive the multi-step form values via a `values` prop. Whenever a field changes, it notifies the parent with the updated form values via `onValuesChange`.

```tsx
// components/form/index.tsx
import type { DefaultValues, Resolver } from "react-hook-form";
import type { OnBack, OnNext } from "@formity/react";

import { useForm, FormProvider } from "react-hook-form";
import { useEffect, useEffectEvent } from "react";

import type { FormStatus } from "@/types/status";

import { ItemView, type Item } from "./item";
import { Button } from "../button";

interface FormProps<T extends Record<string, unknown>, U extends T> {
  defaultValues: DefaultValues<T>;
  resolver: Resolver<T>;
  heading: string;
  message: string;
  content: Item[];
  buttons: {
    back: string | null;
    next: string;
  };
  onBack: OnBack<T>;
  onNext: OnNext<T>;
  status: FormStatus;
  values: U;
  onValuesChange: (values: U) => void;
}

export function Form<T extends Record<string, unknown>, U extends T>({
  defaultValues,
  resolver,
  heading,
  message,
  content,
  buttons,
  onBack,
  onNext,
  status,
  values,
  onValuesChange,
}: FormProps<T, U>) {
  const form = useForm({ defaultValues, resolver });

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
    <form noValidate autoComplete="off" onSubmit={form.handleSubmit(onNext)}>
      <FormProvider {...form}>
        <div className="mb-8">
          <h2 className="mb-1.5 text-2xl font-bold text-gray-950">{heading}</h2>
          <p className="text-sm font-medium text-gray-400">{message}</p>
        </div>
        <div className="@container mb-8 flex flex-col gap-6">
          {content.map((item, i) => (
            <ItemView key={i} {...item} />
          ))}
        </div>
        <div className="flex w-full items-center justify-end gap-4">
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
          <Button type="submit" variant="primary" disabled={status.submitting}>
            {status.submitting ? "Submitting..." : buttons.next}
          </Button>
        </div>
      </FormProvider>
    </form>
  );
}
```

## Updating the flow

We need to update the flow to add `values` and `onValuesChange` to `params` so they can be passed down to each `Form`. We also export `initialValues` so the parent has a starting point for the state.

```tsx
// flow.tsx
import type { UnionToIntersection } from "type-fest";
import type { Flow, s } from "@formity/react";
import type { FormStatus } from "./types/status";

import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

import { Form } from "./components/form";

import * as constants from "./constants";

type Values = UnionToIntersection<Fields[keyof Fields]>;

type Fields = {
  contact: {
    firstName: string;
    lastName: string;
    email: string;
  };
  shipping: {
    address: string;
    city: string;
    postalCode: string;
    country: string;
  };
  delivery: {
    deliveryMethod: string;
  };
  payment: {
    cardholderName: string;
    cardNumber: string;
    expiryDate: string;
  };
};

export type Schema = {
  render: React.ReactNode;
  struct: [
    s.Form<Fields["contact"]>,
    s.Form<Fields["shipping"]>,
    s.Form<Fields["delivery"]>,
    s.Form<Fields["payment"]>,
    s.Return<Values>,
  ];
  inputs: Record<never, never>;
  params: {
    status: FormStatus;
    values: Values;
    onValuesChange: (values: Values) => void;
  };
};

export const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        firstName: ["", []],
        lastName: ["", []],
        email: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          key="contact"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              firstName: z.string().nonempty("Please enter your first name"),
              lastName: z.string().nonempty("Please enter your last name"),
              email: z.email("Please enter a valid email address"),
            }),
          )}
          heading="Contact"
          message="We'll send your order confirmation here."
          content={[
            {
              type: "columns",
              columns: [
                {
                  type: "input",
                  name: "firstName",
                  label: "First name",
                  placeholder: "Jane",
                },
                {
                  type: "input",
                  name: "lastName",
                  label: "Last name",
                  placeholder: "Smith",
                },
              ],
            },
            {
              type: "input",
              name: "email",
              label: "Email address",
              placeholder: "jane@example.com",
              inputType: "email",
            },
          ]}
          buttons={{ back: null, next: "Continue" }}
          onBack={onBack}
          onNext={onNext}
          status={params.status}
          values={params.values}
          onValuesChange={params.onValuesChange}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        address: ["", []],
        city: ["", []],
        postalCode: ["", []],
        country: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          key="shipping"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              address: z.string().nonempty("Please enter your address"),
              city: z.string().nonempty("Please enter your city"),
              postalCode: z.string().nonempty("Please enter your postal code"),
              country: z.string().nonempty("Please select your country"),
            }),
          )}
          heading="Shipping address"
          message="Where should we deliver your order?"
          content={[
            {
              type: "input",
              name: "address",
              label: "Address",
              placeholder: "123 Main St",
            },
            {
              type: "columns",
              columns: [
                {
                  type: "input",
                  name: "city",
                  label: "City",
                  placeholder: "New York",
                },
                {
                  type: "input",
                  name: "postalCode",
                  label: "Postal code",
                  placeholder: "10001",
                },
              ],
            },
            {
              type: "select",
              name: "country",
              label: "Country",
              placeholder: "Select a country",
              options: constants.countries,
            },
          ]}
          buttons={{ back: "Back", next: "Continue" }}
          onBack={onBack}
          onNext={onNext}
          status={params.status}
          values={params.values}
          onValuesChange={params.onValuesChange}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        deliveryMethod: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          key="delivery"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              deliveryMethod: z
                .string()
                .nonempty("Please select a delivery method"),
            }),
          )}
          heading="Delivery method"
          message="Choose how fast you'd like your order to arrive."
          content={[
            {
              type: "radio",
              name: "deliveryMethod",
              options: constants.deliveryMethods.map((m) => ({
                value: m.value,
                label: m.label,
                description: m.description,
                priceLabel: m.priceLabel,
              })),
            },
          ]}
          buttons={{ back: "Back", next: "Continue" }}
          onBack={onBack}
          onNext={onNext}
          status={params.status}
          values={params.values}
          onValuesChange={params.onValuesChange}
        />
      ),
    },
  },
  {
    form: {
      fields: () => ({
        cardholderName: ["", []],
        cardNumber: ["", []],
        expiryDate: ["", []],
      }),
      render: ({ fields, params, onBack, onNext }) => (
        <Form
          key="payment"
          defaultValues={fields}
          resolver={zodResolver(
            z.object({
              cardholderName: z
                .string()
                .nonempty("Please enter the cardholder name"),
              cardNumber: z.string().nonempty("Please enter your card number"),
              expiryDate: z.string().nonempty("Please enter the expiry date"),
            }),
          )}
          heading="Payment"
          message="Your payment information is encrypted and secure."
          content={[
            {
              type: "input",
              name: "cardholderName",
              label: "Cardholder name",
              placeholder: "Jane Smith",
            },
            {
              type: "cardNumber",
              name: "cardNumber",
              label: "Card number",
            },
            {
              type: "expiryDate",
              name: "expiryDate",
              label: "Expiry date",
            },
          ]}
          buttons={{ back: "Back", next: "Place order" }}
          onBack={onBack}
          onNext={onNext}
          status={params.status}
          values={params.values}
          onValuesChange={params.onValuesChange}
        />
      ),
    },
  },
  {
    return: (values) => values,
  },
];

export const initialValues: Values = {
  email: "",
  firstName: "",
  lastName: "",
  address: "",
  city: "",
  postalCode: "",
  country: "",
  deliveryMethod: "",
  cardholderName: "",
  cardNumber: "",
  expiryDate: "",
};
```

## Using the state

Finally, we update `app.tsx` to hold the form state and make it available outside the form. This state is used in the `Summary` component, showing what the user has entered across all steps.

```tsx
// app.tsx
import type { OnReturn, ReturnOutput } from "@formity/react";
import type { Status, FormStatus } from "./types/status";

import { useState, useCallback } from "react";
import { useFormity } from "@formity/react";

import { Summary } from "./components/summary";
import { Done } from "./components/done";

import { flow, initialValues, type Schema } from "./flow";

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
    onReturn,
  });

  return (
    <div className="flex h-screen w-full overflow-hidden bg-white">
      <main className="flex flex-1 items-start justify-center overflow-y-auto px-6 py-12">
        <div className="w-full max-w-lg">{form}</div>
      </main>
      <Summary
        values={{
          email: values.email,
          city: values.city,
          country: values.country,
          deliveryMethod: values.deliveryMethod,
        }}
      />
    </div>
  );
}
```
