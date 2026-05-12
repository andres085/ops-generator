Analyze the shared library and document its public API so the implementer knows what is available and what MUST be used instead of reimplemented.

Study the library at: `<shared_library>`

The goal is to produce a reference guide that answers one question: **what does this library provide, and how do you use it?**

Read the library source thoroughly. Focus on public APIs, types, and initialization patterns. Skip internal/private packages.

**IMPORTANT**: Skip generated files (`*.pb.go`, `*.generated.*`). Read `.proto` files instead of generated protobuf code. Skip test files unless they are the only documentation of how to use an API.

Produce a library guide saved as `generated/<artifact>/research/library-guide.md` covering:

1. **Library overview**: One paragraph — what problem does this library solve? What does it provide to consumers?

2. **Dependency declaration**: The exact line to add to `go.mod`, `package.json`, `requirements.txt`, or equivalent.

3. **Key packages / modules**: List each importable package or module with a one-line description of its responsibility.

4. **Initialization pattern**: How to set up the library at application startup. Show the exact call sequence — what to construct, what configuration it needs, what it returns.

5. **Core APIs**: For each key type or function that consumers are expected to use, document:
   - Full import path
   - Function/method signature
   - What it does and when to use it
   - A minimal usage example

6. **Configuration**: What environment variables, config structs, or parameters the library reads. Show the struct definition or config schema.

7. **What NOT to reimplement**: Explicit list of functionality the library already provides. These are things the implementer must use from the library instead of writing themselves.

8. **Error handling contract**: How errors from the library are returned. Are there sentinel errors to check? Error types to unwrap?

9. **Shared types / interfaces**: Types defined in the library that the consumer must use (e.g., request/response types, interfaces to implement, proto message types).

Include exact import paths, type names, and minimal code snippets. The goal is that the implementer can read this guide and know exactly what the library offers without reading its source code.
