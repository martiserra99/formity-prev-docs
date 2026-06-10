# State

Learn what the state is about and how it can be used.

## State

In the form element's `render` function, there are the following two functions:

- `getState`: Takes the current form values and returns the current state of the multi-step form.
- `setState`: Updates the state of the multi-step form to a new value.

The `Formity` component also accepts an `initialState` prop to restore a previously saved state so the user can resume where they left off.

### Saving the state

To save the state as the user fills out the form, call `getState` inside your form component and persist it whenever the values change. Using React Hook Form, it could look like this:

```tsx
const onSaveState = useEffectEvent(({ values }) => {
  const state = getState(values);
  saveState(state);
});

useEffect(() => {
  const unsubscribe = form.subscribe({
    formState: { values: true },
    callback: onSaveState,
  });
  onSaveState({ values: form.getValues() });
  return () => unsubscribe();
}, [form]);
```

### Resuming the state

Pass the saved state to the `initialState` prop to resume the form from where the user left off.

```tsx
return (
  <Formity<Schema> flow={flow} initialState={savedState} onReturn={onReturn} />
);
```
