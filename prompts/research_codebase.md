Analyze the reference codebase and document the implementation patterns that the new artifact must follow.

Study the codebase at: `<reference_codebase>`
If a specific reference implementation exists (a file, module, or service to model): `<reference_implementation>`

Read the code thoroughly. Focus on understanding and documenting patterns that new code MUST follow.

**IMPORTANT**: Skip generated files (`*.pb.go`, `*.generated.*`, `dist/`, `build/`). Skip test files (`*.test.*`, `*_test.*`, `*.spec.*`) unless specifically needed to understand testing conventions.

Produce a pattern guide saved as `generated/<artifact>/research/codebase-guide.md` covering:

1. **Tech stack**: Language, frameworks, and key libraries. Include exact versions if visible in `go.mod`, `package.json`, `requirements.txt`, or equivalent.

2. **Project conventions**: Naming conventions (files, functions, types, variables), code style, and formatting rules. Note what would look "wrong" compared to the rest of the codebase.

3. **Project structure**: Directory layout and the role of each main directory. Show a representative example with a short description per folder.

4. **Configuration pattern**: How configuration is loaded (env vars, config files, CLI flags). Show the exact pattern — struct tags, required fields, defaults.

5. **Error handling**: How errors are returned, wrapped, and logged. Show the pattern for three cases:
   - Fatal errors that should abort the operation
   - Recoverable errors that should retry or propagate
   - Per-item errors that should log and skip

6. **Logging**: Logger type used, how it's initialized, what gets logged at each level, and how structured fields are added.

7. **Entry point pattern**: How the application is started. Initialization order, dependency injection, graceful shutdown.

8. **Key shared APIs**: List of shared utilities, base classes, or modules that new code MUST use instead of reimplementing. For each: import path, key method signatures, and when to use it.

9. **Testing conventions**: Testing framework, file naming, how tests are structured (table-driven, describe/it, etc.), how mocks or stubs are done, and the exact command to run tests.

10. **Dependency management**: How dependencies are added (go.mod, package.json, etc.). Any private module setup required.

Include exact import paths, function signatures, and type names where relevant. The goal is that the implementer can read this guide and write code that is indistinguishable from the existing codebase in style and patterns.
