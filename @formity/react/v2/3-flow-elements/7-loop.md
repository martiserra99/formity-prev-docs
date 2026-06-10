# Loop

Learn how the loop element is used in the flow.

## Usage

The **loop** element repeats a set of flow elements while a condition is `true`.

```tsx
type Schema = {
  render: React.ReactNode;
  struct: [
    s.Variables<{
      i: number;
      questions: {
        question: string;
        options: { value: string; label: string }[];
        default: string;
        correct: string;
      }[];
      right: number;
      wrong: number;
    }>,
    s.Loop<
      [
        s.Variables<{
          question: {
            question: string;
            options: { value: string; label: string }[];
            default: string;
            correct: string;
          };
        }>,
        s.Form<{ answer: string }>,
        s.Variables<{ i: number; right: number; wrong: number }>,
      ]
    >,
    s.Return<{ right: number; wrong: number }>,
  ];
  inputs: Record<never, never>;
  params: Record<never, never>;
};

const flow: Flow<Schema> = [
  {
    variables: () => ({
      i: 0,
      questions: [
        {
          question: "What is the capital of Australia?",
          options: [
            { value: "sydney", label: "Sydney" },
            { value: "melbourne", label: "Melbourne" },
            { value: "canberra", label: "Canberra" },
          ],
          default: "sydney",
          correct: "canberra",
        },
        {
          question: "Who painted 'The Starry Night'?",
          options: [
            { value: "picasso", label: "Pablo Picasso" },
            { value: "vangogh", label: "Vincent van Gogh" },
            { value: "dali", label: "Salvador Dalí" },
          ],
          default: "picasso",
          correct: "vangogh",
        },
      ],
      right: 0,
      wrong: 0,
    }),
  },
  {
    loop: {
      while: ({ i, questions }) => i < questions.length,
      do: [
        {
          variables: ({ i, questions }) => ({
            question: questions[i],
          }),
        },
        {
          form: {
            fields: ({ question, i }) => ({
              answer: [question.default, [i]],
            }),
            render: ({ fields, values, onBack, onNext }) => (
              <QuestionForm
                key={values.i}
                fields={fields}
                question={values.question}
                step={values.i}
                onBack={onBack}
                onNext={onNext}
              />
            ),
          },
        },
        {
          variables: ({ i, question, answer, right, wrong }) => ({
            i: i + 1,
            right: question.correct === answer ? right + 1 : right,
            wrong: question.correct === answer ? wrong : wrong + 1,
          }),
        },
      ],
    },
  },
  {
    return: ({ right, wrong }) => ({ right, wrong }),
  },
];
```

The `while` function receives the flow values and returns a boolean. While it is `true`, the elements in `do` are executed. The `do` array accepts any flow elements.

> The form inside the loop needs a dynamic key to ensure uniqueness across iterations. Additionally, the array of the field must also change between iterations to reset the default value.
