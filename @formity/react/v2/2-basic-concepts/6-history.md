# History

Learn how the history prop controls back navigation in the form.

## History

The `history` prop is a boolean to control whether Formity tracks previous states for back navigation. It defaults to `true`, which is the right choice when the user can only go forward and backward.

Set it to `false` when navigation is handled by jumping instead, such as a form with a sidebar that lets users jump directly to any step.

```tsx
return <Formity<Schema> flow={flow} history={false} onReturn={onReturn} />;
```
