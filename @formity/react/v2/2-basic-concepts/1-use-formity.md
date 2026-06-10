# useFormity

Everything you need to know about the useFormity hook.

## useFormity

The `useFormity` hook is the main hook of this package. It renders the multi-step form and it receives an object with these properties:

- `flow`: Defines the structure and behavior of the multi-step form.
- `inputs`: Defines values that can be used within the flow (optional).
- `params`: Defines values that can be used when rendering each form (optional).
- `history`: A boolean to track previous states for back navigation (defaults to true).
- `initialState`: The initial state of the multi-step form (optional).
- `onYield`: A callback function that is triggered when values are yielded (optional).
- `onReturn`: A callback function that is triggered when the form completes (optional).

It also receives a generic type parameter with these properties:

- `render`: The type that represents what each form step renders.
- `struct`: The structure and values handled at each step of the flow.
- `inputs`: The shape of the values passed via the `inputs` prop.
- `params`: The shape of the values passed via the `params` prop.

```tsx
type Schema = {
  render: { step: number; form: ReactNode };
  struct: [s.Form<{ name: string }>, s.Return<{ name: string }>];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    form: {
      fields: () => ({
        name: ["", []],
      }),
      render: ({ fields, onNext }) => ({
        step: 0,
        form: <AboutMeForm fields={fields} onNext={onNext} />,
      }),
    },
  },
  {
    return: ({ name }) => ({ name }),
  },
];

const { step, form } = useFormity<Schema>({ flow, onReturn });
```
