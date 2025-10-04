# dev-kit

A developer-friendly library of reusable `.cursor/commands` markdown files for Cursor. Each file is a focused, production-minded coding recipe (e.g., UI components, API endpoints, database entities) with clear intent, style rules, and examples—helping LLMs produce consistent code while keeping token usage low.

## What’s inside

- Reusable command recipes in markdown, designed to be triggered from Cursor’s Chat with `/`.
- Each recipe typically includes: purpose, when to use, required inputs, step-by-step approach, code skeletons, and style conventions (naming, typing, control flow, comments, formatting).

Example layout when installed in a project:

```text
.cursor/
  commands/
    react-date-picker.md
    nestjs-crud-endpoint.md
    ...
```

## Install

You can copy the recipes directly into your project’s `.cursor/commands` folder, or reference this repo as a submodule.

### Option A — Copy files (recommended)

1) Clone this repository locally.
2) Copy the contents of `.cursor/commands` into your project’s `.cursor/commands`.

Bash (macOS/Linux/Git Bash on Windows):

```bash
git clone <REPO_URL> dev-kit
mkdir -p /path/to/your-project/.cursor/commands
cp -R dev-kit/.cursor/commands/* /path/to/your-project/.cursor/commands/
```

PowerShell (Windows):

```powershell
git clone <REPO_URL> dev-kit
New-Item -ItemType Directory -Force -Path .\your-project\.cursor\commands | Out-Null
Copy-Item -Path .\dev-kit\.cursor\commands\* -Destination .\your-project\.cursor\commands -Recurse -Force
```

### Option B — Git submodule

From your project root:

```bash
git submodule add <REPO_URL> .cursor/commands/dev-kit
```

You can then use the recipes under `.cursor/commands/dev-kit`. Optionally copy or symlink selected files up one level if you want them to appear directly in the root `commands` directory.

### Option C — Symlink (advanced)

```bash
ln -s /path/to/dev-kit/.cursor/commands /path/to/your-project/.cursor/commands
```

## Use in Cursor

1) Open your project in Cursor.
2) In Chat, type `/` to open the command palette and start typing a recipe name.
3) Provide any requested inputs the recipe asks for (e.g., entity name, fields).
4) The recipe guides the model with structure and style rules to generate consistent code.

## Examples

### `react-date-picker.md`

- Purpose: Scaffold a typed, accessible React date picker component with props, state, and formatting helpers.
- Includes: prop contract, keyboard interactions, controlled/uncontrolled usage guidance, and styling hooks.
- Example snippet (from a typical recipe):

```md
# React Date Picker
## When to use
For forms that require date selection with keyboard and locale support.

## Inputs
- Component name
- Date library preference (native/`date-fns`/`dayjs`)

## Steps
1. Create typed props with `value`, `onChange`, `min`, `max`.
2. Support keyboard navigation and proper ARIA labels.
3. Expose formatting helpers and test cases.
```

### `nestjs-crud-endpoint.md`

- Purpose: Generate a CRUD HTTP endpoint in NestJS with DTOs, validation, and service wiring.
- Includes: entity schema, DTOs, pipes/guards, repository/service/controller, and e2e test outline.
- Example snippet (from a typical recipe):

```md
# NestJS CRUD Endpoint
## Inputs
- Entity name (singular/plural)
- Fields (name:type:constraints)

## Steps
1. Define entity + repository.
2. Create `CreateDto`/`UpdateDto` with `class-validator`.
3. Implement service with typed methods and error handling.
4. Add controller routes: `POST`, `GET`, `GET/:id`, `PATCH/:id`, `DELETE/:id`.
5. Add tests and example requests.
```

## Conventions

- Strong typing for public APIs and component props.
- Clear naming; avoid abbreviations; prefer early returns; handle errors first.
- Comments explain “why,” not “how.” Keep formatting consistent across files.

## Contributing

Contributions are welcome. Add new `.md` recipes under `.cursor/commands`, following the conventions above. Keep each recipe self-contained, with purpose, inputs, steps, and end-to-end examples.

## License

See `LICENSE` in this repository.
