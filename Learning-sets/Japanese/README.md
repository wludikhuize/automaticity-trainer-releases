# Learning-set format

A learning set is a JSON file containing exercises and the choices available while answering them. Learning sets contain only data; no JavaScript is required.

## Compatibility guarantee

From application version 1.1.0 onward, the learning-set JSON format is permanently backward compatible. Any file accepted by an earlier version—including the legacy top-level exercise array—will continue to load with the same meaning. Future fields will be optional and additive. Existing fields will not be removed, renamed, narrowed, made required, or reinterpreted, and choice inheritance will not change.

The in-app **Create and add a learning set** section is a copy-ready prompt for generating this format and contains the current upload and folder instructions. It must be updated together with this document whenever the accepted JSON structure or loading workflow changes.

For the website, any valid `.json` file placed in this repository directory is discovered when the server starts or the application refreshes. The Electron app instead uses `Documents/Automaticity Trainer/learning-sets/` (or the `learning-sets/` child of `TRAINER_HOME`). Desktop packages deliberately contain no initial learning sets.

Uploading a learning set with the file picker—or dragging and dropping one JSON file anywhere on the application—writes a formatted copy into the active runtime directory. A higher-version import with the same stable root `id` replaces the existing file at its actual saved filename. Use **Export JSON** in the saved set's action menu to download a clean shareable copy without finding this directory.

During practice, **Discard sentence** can remove an unwanted generated exercise from its learning-set JSON file. The server rewrites the file with the remaining valid exercises. It never allows a learning set to become empty.

## Basic structure

```json
{
  "id": "irregular-verbs",
  "name": "Irregular verbs",
  "version": "1.0.0",
  "sourceLanguage": "en",
  "targetLanguage": "nl",
  "choices": ["writes", "writing", "sees", "seeing", "goes", "going"],
  "exercises": [
    {
      "sentence": "Yesterday I _ a letter.",
      "answers": ["wrote"]
    }
  ]
}
```

The root value must be a JSON object. Comments and trailing commas are not valid JSON.

## Root fields

### `id`

- Optional string
- Identifies the learning set and connects versioned revisions
- Should be stable, short, and unique
- Use lowercase words separated by hyphens, such as `irregular-verbs`
- Exercise progress does not depend on this ID; each exercise receives its own content-based ID
- The application uses this ID as the saved-library key. When `version` is present, only a higher version with this same ID may replace it

### `name`

- Optional non-empty string
- Display name shown by the application
- If omitted, the uploaded filename is used

### `version`

- Optional string in numeric `MAJOR.MINOR.PATCH` format, such as `1.0.0` or `2.3.1`
- Describes the learning-set revision, independently of the application version
- Keep `id` unchanged and increase `version` when sharing a corrected or expanded revision
- Importing a higher version with the same ID overwrites the existing set's actual file, even if the imported filename differs
- Equal versions, older versions, and an unversioned replacement of a versioned set are refused
- A versioned set can upgrade a legacy unversioned set. Two unversioned copies retain the legacy filename/import behavior
- Version does not affect exercise identity; progress is retained for exercises whose sentence and ordered answers are unchanged
- An invalid root version is reported and ignored while otherwise valid exercises load

### `sourceLanguage` and `targetLanguage`

- Optional [BCP 47 language tags](https://www.rfc-editor.org/rfc/bcp/bcp47.txt)
- `sourceLanguage` describes the language of `sentence`, `answers`, and `choices`
- `targetLanguage` describes the language of `translation` and normally `explanation`
- Common examples are `en`, `nl`, `ja`, `de`, `pt-BR`, and `zh-Hant`
- The application canonicalizes valid tags, so for example `EN-us` becomes `en-US`
- These values set HTML language metadata for the displayed source text, answer controls, typed input, translation, feedback, discard preview, and tutor reports
- They do not translate text or restrict which Unicode characters can be used
- Omit either field when its language is unknown or not applicable; existing learning sets therefore keep their previous neutral behavior
- An exercise can override either root tag for multilingual or mixed-language sets

### `practiceMode`

- Optional string
- Forces this learning set to use one existing client mode: `adaptive`, `sequential`, `random`, `mistakes`, or `slow`
- `sequential` performs one complete pass in the order of objects in the root `exercises` array, regardless of the normal session length, and does not insert adaptive review questions; it is suitable for a longer story or passage presented sentence by sentence
- While the set is active, the mode selector displays the required mode and cannot be changed
- Omit this field to use the learner's saved mode selection, preserving the behavior of older learning sets
- The value does not affect exercise identity or progress

```json
{
  "id": "short-story",
  "name": "A short story",
  "practiceMode": "sequential",
  "exercises": [
    { "sentence": "First, the traveller _ home.", "answers": ["left"] },
    { "sentence": "Then she _ the river.", "answers": ["crossed"] }
  ]
}
```

### `choices`

- Optional array of non-empty strings
- Defines one flat default distractor list available to every gap in every exercise
- Duplicate values are removed when the file is loaded
- Values can be words, symbols, dates, formulas, or any other missing text
- An exercise can replace this list by defining its own `choices`
- Correct answers do not need to be listed: the application inserts the accepted answers for the active gap and shuffles the displayed buttons at runtime

### `exercises`

- Required, non-empty array
- Contains the exercise objects described below

## Exercise fields

### `sourceLanguage` and `targetLanguage`

- Optional BCP 47 tags with the same meaning as the root fields
- When omitted, the corresponding root language is inherited
- When supplied, the exercise value replaces the root value for that exercise
- A supplied invalid or empty tag makes that exercise invalid; an invalid root tag is reported and ignored while otherwise valid exercises can still load

### `sentence`

- Required string
- Must contain at least one underscore (`_`)
- Every underscore represents one answer slot
- Avoid using underscores as ordinary punctuation because they will become blanks

Example:

```json
"sentence": "Yesterday I _ a letter."
```

This produces one visible empty slot (represented as `___` below):

```text
Yesterday I ___ a letter.
```

### `answers`

- Required array containing one entry for each underscore
- Each entry can be a non-empty string or a non-empty array of accepted strings
- Answers must be in the same order as the blanks
- Answers are compared exactly, including spelling and characters

```json
{
  "sentence": "Yesterday I _ a letter.",
  "answers": ["wrote"]
}
```

Here, `wrote` belongs to the only blank.

To accept multiple answers for one blank, use a nested array:

```json
{
  "sentence": "The path leads _ the river.",
  "answers": [["toward", "towards"]]
}
```

Both `toward` and `towards` are correct for that blank. With multiple blanks, every outer-array position still belongs to one blank:

```json
{
  "sentence": "Yesterday _ _ a letter.",
  "answers": [["I", "we"], ["wrote", "sent"]]
}
```

### `translation`

- Optional non-empty string
- Clarifies the intended meaning of the complete sentence
- Can disambiguate the context or meaning expected by an exercise
- Displayed below the source sentence only when the learner enables translations in the interface
- If omitted, no translation or translation toggle is shown for that exercise
- May contain line breaks

```json
{
  "sentence": "La capitale de la France est _.",
  "answers": ["Paris"],
  "translation": "Name the capital city of France."
}
```

### `explanation`

- Optional non-empty string
- Displayed after the answer is checked, for both correct and incorrect responses
- Never displayed before checking
- Intended for short context about why the accepted answers work
- May contain line breaks

```json
{
  "sentence": "The path leads _ the river.",
  "answers": [["toward", "towards"]],
  "explanation": "Both spellings are accepted."
}
```

### `choices`

- Optional flat array of non-empty strings, or nested array containing one string array per gap
- Replaces the root `choices` list for this exercise
- Does not merge with the root list
- A flat array is the backward-compatible form and supplies the same distractors to every gap
- A nested array must contain exactly one list per underscore, in the same order as `answers`, and supplies different distractors to each gap
- Do not add accepted answers merely to make them visible. The app inserts all accepted alternatives for the active gap automatically and shuffles their positions with the distractors
- Choice order never identifies the correct answer

```json
{
  "sentence": "Yesterday I _ a letter.",
  "answers": ["wrote"],
  "choices": ["write", "written"]
}
```

For multiple gaps, use one nested list per gap:

```json
{
  "sentence": "Yesterday _ _ a letter.",
  "answers": ["we", "wrote"],
  "choices": [
    ["I", "they"],
    ["write", "written"]
  ]
}
```

Choice selection follows this order:

1. Use the active gap's nested exercise-level choice list when present.
2. Otherwise, use a flat exercise-level `choices` list for every gap.
3. Otherwise, use the flat root-level `choices` list.
4. Insert every accepted answer for the active gap, remove duplicates, and shuffle the button order.

In Advanced mode, buttons are hidden and the learner types the answer, so configured choices do not limit accepted input.

## Complete example

```json
{
  "id": "irregular-verbs",
  "name": "Irregular verbs",
  "sourceLanguage": "en",
  "targetLanguage": "nl",
  "choices": [
    "writes",
    "writing",
    "sees",
    "seeing",
    "goes",
    "going"
  ],
  "exercises": [
    {
      "sentence": "Yesterday I _ a letter.",
      "answers": ["wrote"]
    },
    {
      "sentence": "I have _ that film twice.",
      "answers": ["seen"],
      "choices": ["see", "saw"]
    },
    {
      "sentence": "Last week we _ to the museum.",
      "answers": ["went"]
    }
  ]
}
```

## Exercise identity and progress

The application creates an internal deterministic ID from the sentence and ordered answer structure. Conceptually:

```text
sentence + "|" + canonical accepted answers
```

Consequences:

- Reordering exercises does not lose their progress.
- Changing root or exercise choices does not reset exercise progress.
- Changing language tags does not reset exercise progress.
- Changing a sentence or any correct answer creates a new exercise with fresh progress.
- Adding or removing an accepted alternative creates a new exercise with fresh progress.
- Existing exercises whose answers are all strings keep their previous identity.
- No exercise ID needs to be written into the JSON file.

## Legacy minimal format

The older array-only format is still accepted:

```json
[
  {
    "sentence": "Yesterday I _ a letter.",
    "answers": ["wrote"]
  }
]
```

It has no root-level choices. The correct answers are still available as buttons, but the object format is recommended because it supports an explicit learning-set name and default choices.

## Validation checklist

Before adding or uploading a learning set, check that:

- The file is valid JSON and normally uses the `.json` extension.
- The root object contains a non-empty `exercises` array.
- Every exercise has a string `sentence` and an array of `answers`.
- Every sentence contains at least one underscore.
- Every sentence has the same number of underscores and answers.
- Every answer entry is a non-empty string or a non-empty array of non-empty strings.
- No choice is an empty string.
- Exercise choices contain either only strings or only per-gap arrays, never a mixture.
- Nested exercise choices contain exactly one array for every underscore in the sentence.
- Every supplied `sourceLanguage` or `targetLanguage` is a non-empty valid BCP 47 language tag.
- A supplied root `version` consists of exactly three numeric `MAJOR.MINOR.PATCH` components without leading zeroes.
- A supplied root `practiceMode` is exactly one of `adaptive`, `sequential`, `random`, `mistakes`, or `slow`.
- Every explanation, when present, is a non-empty string.
- Every translation, when present, is a non-empty string.
- Exercise-level choices contain useful distractors for the relevant gap and do not duplicate accepted answers unnecessarily.

Exercises are validated independently. If one or more exercises fail these checks, the valid exercises are still loaded and a report lists each skipped exercise by its original one-based position and reason.

The complete file is rejected only when:

- The JSON syntax cannot be parsed.
- The root has no `exercises` array.
- The exercises array is empty.
- No exercise in the file is valid.

If root-level `choices` is invalid, the application reports it and continues with an empty root choice list. Invalid root `version` and `practiceMode` values are likewise reported and ignored. Ignoring an invalid version does not permit it to replace an installed versioned set. A file uploaded or dropped through the application is cleaned before being written into this directory, so its saved copy contains only accepted exercises. A file placed in this directory manually is not modified; invalid entries in it will be reported again after the next refresh.
