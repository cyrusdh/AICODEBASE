# AICODEBASE

AICODEBASE is a central archive for AI-assisted projects, reference prompts, and knowledge. The repository stores completed project snapshots alongside reusable prompts so past work can be referenced in future sessions.

## Repository structure

- **[ProjectIndex.md](./ProjectIndex.md)** – high-level summary of every archived project.
- **[Prompts.md](./Prompts.md)** – collection of standard prompts for archiving projects and continuing development.
- **projects/** – directory containing individual project folders, each with a `project_reference.md` file and any extracted code.

## Archiving a project

1. When a project is complete, open `Prompts.md` and use [Prompt 1](./Prompts.md#prompt-1-push-project-to-github).
2. Save the AI's markdown response as `project_reference.md` inside a new folder under `projects/`.
3. Extract the files from the response into that folder and commit them to the repository.

## Continuing development

1. Copy the contents of the archived `project_reference.md`.
2. Use [Prompt 2](./Prompts.md#prompt-2-continue-working-on-existing-project) to resume work or add new features while keeping compatibility with the existing codebase.

For a list of available projects, see [ProjectIndex.md](./ProjectIndex.md).
