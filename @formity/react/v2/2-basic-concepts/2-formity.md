# Formity

Everything you need to know about the Formity component.

## Formity

The `Formity` component is the main component of this package. It accepts the same props and generic type parameter as `useFormity`.

The only difference is that the `render` property of the `Schema` type must be `ReactNode`, since the component renders directly to the screen.

```tsx
type Schema = {
  render: ReactNode;
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
      render: ({ fields, onNext }) => (
        <AboutMeForm fields={fields} onNext={onNext} />
      ),
    },
  },
  {
    return: ({ name }) => ({ name }),
  },
];

return <Formity<Schema> flow={flow} onReturn={onReturn} />;
```
