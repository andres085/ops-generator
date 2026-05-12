Research what needs to be built and document the information the architect needs to design it.

This research is about the **target** — what you are going to build and what it interacts with — not the existing codebase.

Use web search to find official documentation, SDK references, and code examples when the artifact will consume an external API or service. For internal features, analyze the existing system to understand constraints.

Investigate based on the nature of the artifact:

1. **What needs to be built**: Summarize the artifact in one paragraph. What does it do? What does it consume? What does it produce? What problem does it solve?

2. **External interfaces** (if consuming an external API, SDK, or service):
   - Available strategies for the required operation — compare bulk/streaming/recursive/event-based approaches
   - Best SDK or client library for the target language — compare official vs community, pagination support, maintenance
   - Authentication model for a long-running service (not user-interactive) — what credentials, scopes, token refresh
   - Pagination or streaming behavior — cursor vs offset, page size limits, streaming support
   - Available data/metadata — what fields are returned, which require extra API calls
   - Rate limits and throttling — limits, backoff strategy
   - Key gotchas — non-obvious constraints, permission requirements, API quirks

3. **Functional requirements** (if building an internal feature):
   - Inputs and outputs — what data enters and exits
   - Business rules — what transformations, validations, or decisions happen
   - Edge cases — empty input, large volume, concurrent access, partial failures

4. **Dependencies**: Other services, systems, or components this artifact depends on. What contracts must it respect?

5. **Constraints**: Performance requirements, data volume expectations, latency budgets, compliance or security requirements.

Do NOT write implementation code — that is the implementer's job. Focus on the architectural facts needed to make design decisions.

**Deliverable**: Save the research as `generated/<artifact>/research/target-guide.md` with this structure:

```markdown
# Target Guide: <Artifact>

## Summary
<One paragraph: what this artifact does, what it consumes, what it produces.>

## Recommendation
<One-line summary of the recommended approach to the main challenge.>
<Brief explanation of why this approach wins over alternatives.>

## Strategies Compared
<Table or list comparing available approaches. For each: mechanism, pros, cons, when to use.>

## Recommended SDK / Library
<Name, version, reason chosen over alternatives. Include dependency declaration (go.mod line, npm package, etc.).>

## Authentication Model
<Auth method, required credentials/scopes, and how token refresh works. No code — just architectural facts.>

## Available Data / Metadata
<Table mapping source fields → artifact-relevant fields. Note which require extra API calls.>

## Rate Limits / Constraints
<Limits, throttle response format, recommended backoff strategy.>

## Edge Cases and Gotchas
<Numbered list of non-obvious constraints.>

## Sources
<URLs to official docs, SDK repos, API references.>
```
