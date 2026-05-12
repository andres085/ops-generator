---
name: min-prompt-tool-use
description: Minimize permission prompts from using tools. The directives are designed to make the commands fit better to allow patterns defined in `settings.json`.
---

# Minimizing permission prompts when using tools

Directives and patterns when using tools to reduce the number of permission prompts to the user.

## When to Activate
- When using tools

## Temp files
- NEVER use `/tmp` or any system temp directory. Write and Edit permissions only cover the project folder tree, so files written to `/tmp` will trigger permission prompts or fail.
- Always use `generated/tmp` as the temp folder. Create it with `mkdir -p generated/tmp` if needed.

## Built-in tools

### Specialized built-in tools
- Prefer specialized built-in tools over Bash; built-in tools avoid permission issues with special characters in paths.
- Use `Read` instead of `cat`/`head`/`tail`
- Use `Glob` instead of `ls`/`find`
- Use `Grep` instead of `grep`/`rg`
- Always use project-root-relative paths (e.g., `generated/my-service/internal/config/config.go`). NEVER use `..` in paths.

### Bash built-in command
- Avoid unnecessary quotes in bash commands. Use `echo ---` not `echo "---"`.
- Avoid chaining `cd` with `&&` when the tool supports path arguments directly (e.g., use `venv/bin/pytest generated/my-service/test_integration/ -v` instead of `cd generated/my-service/test_integration && venv/bin/pytest -v`).
- Do not use brace expansion `{}` in bash commands — it conflicts with the permission system's glob matching and will trigger unnecessary permission prompts. Use separate commands or spell out full paths instead.

## Stack-specific notes

### Go
- Use `go build`, `go test`, `go vet`, `go generate` directly (allowed globally).
- DO NOT prefix commands with `GOPRIVATE=...` or `GOPROXY=...` unless they are NOT set in the project environment.
- Avoid command substitution `$(...)` inline — run commands separately and use resolved paths.

### Python
- Use `venv/bin/python`, `venv/bin/pip`, `venv/bin/pytest` (never system `python`/`pip`/`pytest`).

### Node.js
- Use `npm run <script>` and `npx` (never bare package binaries that aren't in PATH).
