---
type: workflow
title: Code Sample Lifecycle
description: How runnable documentation samples are executed, extracted into generated MDX, optionally published with public trace links, and maintained in CI. Includes the MCP structured-content sample as a source-to-snippet example.
tags: [code-samples, documentation, testing, ci, tracing]
verified:
  - by: openwiki/0.4.3
    at: 2026-09-16T08:21:31.094Z
sources:
  - id: openwiki-source-ddbddbe474c8dc57119458d7
    resource: repo://.agents/skills/docs-code-samples/SKILL.md
  - id: openwiki-source-751a704f6f25787856371177
    resource: repo://.github/workflows/test-code-samples-linear.yml
  - id: openwiki-source-97746d8f3662d803e625550e
    resource: repo://.github/workflows/test-code-samples.yml
  - id: openwiki-source-ea70eb6c045047448e446296
    resource: repo://.gitignore
  - id: openwiki-source-012f2c78e3b1446dfc35803f
    resource: repo://Makefile
  - id: openwiki-source-2654e40275744504b4ca7e2b
    resource: repo://scripts/code_sample_tracing.py
  - id: openwiki-source-fd0cb9d6fca56bf4963559e9
    resource: repo://scripts/extract_code_snippets.py
  - id: openwiki-source-560bf24db9566b97ee19e383
    resource: repo://scripts/generate_code_snippet_mdx.py
  - id: openwiki-source-2b15ecffacad911ef9db112f
    resource: repo://scripts/test_code_samples.py
  - id: openwiki-source-71e977f60add0174c28f3e6b
    resource: repo://src/code-samples/deepagents/skills-approval.ts
  - id: openwiki-source-057f6d66b4febbf885983b22
    resource: repo://src/code-samples/deepagents/skills-compose-sources.ts
  - id: openwiki-source-6fcb16d581e331b5e3cdb5f9
    resource: repo://src/code-samples/deepagents/skills-writable.ts
  - id: openwiki-source-b0deb1022f38d6591d1ee3af
    resource: repo://src/code-samples/deepagents/skills.py
  - id: openwiki-source-23c78d0acfd59de3b9fa258b
    resource: repo://src/code-samples/langchain/mcp-structured-content.py
  - id: openwiki-source-c131291505f6c5b8e4a3eb29
    resource: repo://src/code-samples/langgraph/langgraph-graph-api-multiple-schemas.ts
  - id: openwiki-source-4676455906eb0588a9444974
    resource: repo://src/code-samples/trace-links.json
  - id: openwiki-source-6f3dd78552e4a8bf387dd731
    resource: repo://src/snippets/code-samples/langgraph-graph-api-multiple-schemas-js.mdx
  - id: openwiki-source-f314d338ff437b2eb5cc03a3
    resource: repo://src/snippets/code-samples/mcp-structured-content-py.mdx
generated: { by: "openwiki/0.4.3", at: "2026-09-16T08:21:31.094Z" }
---

## Purpose and ownership

Runnable documentation examples are authored in supported-language files below `src/code-samples/`. The runnable file is the editable source of truth; `src/code-samples-generated/` is a gitignored extraction intermediate, and MDX below `src/snippets/code-samples/` is a derivative artifact for documentation consumption. Do not hand-edit generated snippet MDX: change the source, run the appropriate checks, regenerate, and review the resulting MDX diff.

```mermaid
flowchart TD
  Source["Editable runnable source"] --> Run["Run sample"]
  Source --> Extract["Extract marked regions"]
  Run --> Result{"Execution succeeds"}
  Extract --> Intermediate["Gitignored intermediate"]
  Intermediate --> Generate["Generate MDX"]
  Trace{"Tracing enabled"} --> Collect["Collect eligible trace"]
  Result --> Trace
  Collect --> Manifest["Trace manifest"]
  Manifest --> Generate
  Generate --> MDX["Committed generated MDX"]
  MDX --> Refresh["Trusted CI refresh PR"]
```

This is an ownership and publication flow, not a proof chain: extraction and MDX generation transform marked text only. A successful extraction does **not** demonstrate that the sample ran, that its imports resolve, or that its provider and other external dependencies are valid. Run the source separately; trace sharing is a further, explicit public-publication operation.

## Author source regions

Supported source extensions are `.py`, `.ts`, `.java`, `.kt`, `.go`, and `.sh`. Put a matched `:snippet-start: <id>` / `:snippet-end:` pair in a language-correct comment: `#` for Python and shell, and `//` for TypeScript, Java, Kotlin, and Go. The line-based extractor permits indented markers, strips matched `:remove-start:` / `:remove-end:` regions from a snippet body, dedents and normalizes the extracted body, and fails on an unclosed snippet or remove region. It deliberately recognizes only comment-line markers rather than parsing TypeScript, avoiding parser failures from comment-like content in strings.

A snippet ID must end in the emitted-language suffix: `-py`, `-js`, `-java`, `-kt`, `-go`, or `-sh`. The MDX generator derives its fence language from that suffix and silently skips an intermediate whose ID has the wrong suffix. Use unique, descriptive kebab-case IDs, since each becomes the MDX filename.

Use trailing remove blocks for runnable harness code, assertions, credentials-only setup, or a blocking tail that readers should not copy. They are executed as part of the source but excluded from the generated snippet. Do not terminate before the visible region with `SystemExit`, `process.exit`, or `exit 0`: such a pass can establish only parsing, not that the shown imports, construction, or calls actually executed.

### Scope and layout

One file can hold related regions and one execution harness. TypeScript samples execute as a single module, so visible regions and remove blocks share module scope. Split independently runnable TypeScript examples when they would duplicate imports or top-level bindings. Python can commonly keep related setup and regions in one file; however, a file with multiple snippet markers is ineligible for a public trace link, so split it when independently traceable examples are wanted.

The existing Deep Agents skills examples illustrate this tradeoff: `skills.py` has multiple markers and is recorded as a multi-snippet trace exclusion, whereas the approval, source-composition, and writable TypeScript sources each have one visible snippet and a trailing hidden assertion. The single-snippet LangGraph multiple-schemas TypeScript sample has a manifest trace entry and its generated MDX renders the corresponding `View example trace` card.

### Presentation controls

The optional first line in a snippet may be `:codegroup-tab:`; `:codegroup-fence-mods:` can follow it or be the first line by itself. Generation consumes these directives rather than emitting them and builds the Mintlify fence. For Python and TypeScript, recognized model strings can expand into configured provider CodeGroups; placing `# KEEP MODEL` or `// KEEP MODEL` immediately before a model occurrence preserves it. These presentation transformations do not execute or validate source code.

## MCP structured-content sample

`src/code-samples/langchain/mcp-structured-content.py` is a single-snippet Python source, identified as `mcp-structured-content-py`. Its visible region opens an `MCPAdapter`, lists its tools, constructs a `create_agent("claude-sonnet-5", tools)` agent, invokes it asynchronously, and prints `message.artifact["structured_content"]` only for `ToolMessage` values with a non-`None` artifact.

The source's remove block supplies the runnable harness that the reader does not see: it creates a local FastMCP `get_user` tool returning a record for Alice, runs the async example, and asserts that a tool artifact exists and has `structured_content["name"] == "Alice"`. Thus the committed generated MDX contains only the reusable agent-facing function while the source retains the local server setup and outcome check. The MDX is evidence that the extractor and generator produced the expected visible text; it is not evidence of a successful execution.

The sample has one marker, satisfying the structural trace eligibility rule, but `trace-links.json` has no entry for `mcp-structured-content-py`; its generated MDX accordingly has no trace card. Eligibility is not publication: it still needs a successful traced run that produces a qualifying agent root before the collector shares a public URL.

## Execute and diagnose

During authoring, use a focused run before a wider suite:

```bash
make test-code-samples FILES="src/code-samples/langchain/mcp-structured-content.py"
make test-code-samples
make code-snippets
```

`FILES` accepts a space-separated explicit list. Missing, unsupported, and out-of-tree paths are warned about and skipped; with no list, the runner recursively selects eligible files below `src/code-samples/` in Python, TypeScript, Java, Kotlin, Go, then shell order. It invokes Python via `uv run python`, TypeScript via `npx tsx`, Go via `go run`, shell via `bash`, and Java/Kotlin through JBang on Java 21. TypeScript, Go, and shell run from `src/code-samples/` to resolve shared dependencies.

The runner inherits the environment, so a sample can use credentials and live providers or PostgreSQL. Its timeout defaults to 1,200 seconds and is configurable through `CODE_SAMPLE_TIMEOUT_SECONDS`. Ordinary nonzero exits, timeouts, and trace-collection exceptions fail the runner. A detected LangSmith 429 response is retried three total times with 15-second delays; after that it is recorded as skipped rather than failed. A rate-limit skip is not a successful validation and does not collect a trace.

## Extract and generate

Run the transformation independently of execution with:

```bash
make code-snippets
```

The target first invokes `scripts/extract_code_snippets.py`, then `scripts/generate_code_snippet_mdx.py`. Extraction writes `<source-stem>.snippet.<snippet-id>.<extension>` under `src/code-samples-generated/`, preserving the product subdirectory. Generation scans all supported intermediates, emits language fences or applicable CodeGroups, consults the trace manifest, and writes `<snippet-id>.mdx` under `src/snippets/code-samples/`.

For iteration, `CODE_SNIPPET_SOURCES` can restrict extraction to existing supported sources under `src/code-samples/`:

```bash
CODE_SNIPPET_SOURCES="src/code-samples/langchain/mcp-structured-content.py" make code-snippets
```

A full extraction removes supported intermediates before rebuilding. A partial extraction replaces intermediates only for the selected source stems, but generation still scans all remaining intermediates. Therefore a partial run is an iteration optimization, not proof of complete derived state; use a full regeneration before relying on the full output set.

## Trace publication

`make update-code-sample-traces` enables `CODE_SAMPLE_TRACING=1`, defaults `LANGSMITH_PROJECT` to `docs-code-samples`, runs the samples with LangSmith tracing, and then regenerates MDX. It requires `LANGSMITH_API_KEY`. Since the collector calls LangSmith sharing to create or reuse a public URL, run it only with authorized credentials and with example inputs and outputs suitable for public visibility.

After a successful source execution, the collector counts source markers. No marker gets no link. Exactly one marker is eligible: it polls up to six times, with two-second waits and a two-second start-time buffer, for a root run in the configured project. It prefers agent-like roots and can fall back to a chain root with an LLM child, then shares the selected run and records URL, source, run and trace IDs, name, and update time in `src/code-samples/trace-links.json`. More than one marker records the source in `skipped_multi_snippet` and removes stale per-snippet entries.

During generation, a manifest URL adds or replaces the trailing `View example trace` card; no URL removes a prior trace CTA. This keeps trace association in the manifest rather than in hand-authored MDX.

## CI boundaries and maintenance

The **Test Code Samples** workflow runs for relevant pull requests, manual dispatch, and at 00:00 UTC on the first day of each month. It skips fork pull requests because secrets are unavailable. Internal PRs calculate the merge-base diff and test only changed eligible samples; manual and scheduled runs test all samples. The workflow provisions Python/uv, Node 20, Java/JBang, Go, PostgreSQL, and relevant secrets.

Only manual and scheduled full runs enable tracing. If testing succeeds, CI regenerates snippets and preserves the trace manifest and generated snippet directory while it restores a clean checkout. It opens or updates the deterministic `chore/refresh-code-sample-traces` PR only when either publication artifact differs. Pull-request checks can execute changed sources but do not share traces or publish artifact changes.

A separate observer creates a Linear issue only for failed or cancelled scheduled sample runs. When investigating failures, distinguish ordinary execution failure, rate-limit skip, lack of a qualifying trace, trace-collection failure, and publication diff behavior.

## Change checklist

1. Edit the runnable source under `src/code-samples/`, never the generated MDX as the primary edit.
2. Add matched language-correct markers and a unique ID with the correct language suffix.
3. Keep the visible region executable; put only non-display harness or blocking code in trailing remove blocks.
4. Run `make test-code-samples FILES="..."`; do not infer success from extraction.
5. Run `make code-snippets` and review generated MDX, using a full regeneration before relying on complete output.
6. Use one marker per source when a public trace is desired, and treat `make update-code-sample-traces` as intentional public publication.

## Related pages

- [Preprocessing](/openwiki/concepts/preprocessing.md)
- [GitHub Actions and CI/CD](/openwiki/integrations/github-actions.md)
- [Adding and Maintaining Documentation Pages](/openwiki/operations/adding-pages.md)
- [Testing Overview](/openwiki/testing/test-overview.md)
