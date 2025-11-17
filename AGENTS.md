# AGENTS.md - Opencode Repository Guidelines

## Build/Lint/Test Commands
- `bun test` - Run all tests
- `bun test <filename>` - Run specific test file
- `bun run check` - Run all Biome checks (format + lint)
- `bun run check:types` - Run TypeScript type checking
- `bun run build` - Build TypeScript project

## Code Style Guidelines
- **TypeScript**: Use strict type checking, prefer explicit types
- **Imports**: Use ES6 imports, organize by type (third-party, local)
- **Formatting**: Biome with tab indentation and double quotes
- **Linting**: Biome recommended rules with style enforcement
- **Naming**: camelCase for variables/functions, PascalCase for types/classes
- **Error Handling**: Use try/catch blocks, proper error types
- **Documentation**: Include JSDoc comments for public APIs

## Agent Development
- Use agent-template.md for new agents
- Follow YAML frontmatter structure: description, mode, model, temperature, tools
- Keep descriptions concise and action-oriented
- Maintain deterministic tone with temperature 0.2 for technical roles

## Critical: Backlog.md Workflow
- **NEVER edit task files directly** - use `backlog task edit` CLI commands only
- **Always use `--plain` flag** for AI-friendly output when viewing tasks
- **Task workflow**: assign yourself with `-a @myself`, set status to "In Progress", add plan, implement, add notes, mark Done
- **Acceptance Criteria**: manage via `--check-ac <index>`, `--ac "new criterion"`, etc.

## Code Standards

- Runtime: Bun with TypeScript 5
- Formatting: Biome with tab indentation and double quotes
- Linting: Biome recommended rules
- Testing: Bun's built-in test runner
- Pre-commit: Husky + lint-staged automatically runs Biome checks before commits
- The pre-commit hook automatically runs biome check --write on staged files to ensure code quality. If linting errors are found, the commit will be blocked until fixed.