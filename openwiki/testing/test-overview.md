---
type: validation guide
title: Testing Overview
description: Change-oriented validation guidance for socket-isolated unit tests, documentation and generated-file gates, package and upstream version checks, and credentialed live code samples. It identifies what each check proves, its intentional boundaries, and how to triage failures.
tags: [testing, pytest, ci, documentation, code-samples, versioning]
verified:
  - by: openwiki/0.4.3
    at: 2026-09-16T08:21:31.094Z
sources:
  - id: openwiki-source-21617d8a6b2b570989a7c900
    resource: repo://.github/workflows/check-version-claims.yml
  - id: openwiki-source-164e2da859b5277df81c7d94
    resource: repo://.github/workflows/ci.yml
  - id: openwiki-source-0976291f8216a4c7151f20a7
    resource: repo://.github/workflows/refresh-external-versions.yml
  - id: openwiki-source-751a704f6f25787856371177
    resource: repo://.github/workflows/test-code-samples-linear.yml
  - id: openwiki-source-97746d8f3662d803e625550e
    resource: repo://.github/workflows/test-code-samples.yml
  - id: openwiki-source-71ee7a4afbd2d6aa7b29f3d1
    resource: repo://htmltest-mint-export.yml
  - id: openwiki-source-012f2c78e3b1446dfc35803f
    resource: repo://Makefile
  - id: openwiki-source-05ccef8d4cf1698187f20464
    resource: repo://pyproject.toml
  - id: openwiki-source-0a0a6c8d7a88288e6b6b9b5b
    resource: repo://scripts/check_cross_refs.py
  - id: openwiki-source-6b3ad04031a04803eb901844
    resource: repo://scripts/check_external_versions.py
  - id: openwiki-source-99b53585619b83f258314f8b
    resource: repo://scripts/check_version_claims.py
  - id: openwiki-source-2654e40275744504b4ca7e2b
    resource: repo://scripts/code_sample_tracing.py
  - id: openwiki-source-bd35b3b527f9ad0799d45497
    resource: repo://scripts/data/external_versions.yaml
  - id: openwiki-source-560bf24db9566b97ee19e383
    resource: repo://scripts/generate_code_snippet_mdx.py
  - id: openwiki-source-2b15ecffacad911ef9db112f
    resource: repo://scripts/test_code_samples.py
  - id: openwiki-source-6a4f3df816b7f7f45b6ac5b1
    resource: repo://src/code-samples/conftest.py
  - id: openwiki-source-a10b62517b8302a8d4cf3b31
    resource: repo://tests/unit_tests/test_check_external_versions.py
  - id: openwiki-source-607673c5c40214b511f9e0a7
    resource: repo://tests/unit_tests/test_check_version_claims.py
  - id: openwiki-source-71e085db64c5296fd9b80141
    resource: repo://tests/unit_tests/test_otel_endpoints.py
  - id: openwiki-source-1695beda93a0ca504f038424
    resource: repo://tests/unit_tests/test_skills.py
generated: { by: "openwiki/0.4.3", at: "2026-09-16T08:21:31.094Z" }
---

## Choose validation by boundary

The repository deliberately separates deterministic, socket-isolated unit tests from checks that intentionally query package registries or GitHub, generated-document checks, and executable samples that contact real services. Select the narrowest check that covers the change. A pass in one boundary does not establish a pass in another.

| Change | Run locally | What a pass establishes | Important limit |
| --- | --- | --- | --- |
| Pipeline, parser, preprocessor, watcher, skill, authored OTel contract, or version-checker logic | `make test` | Isolated behavior and repository structural contracts | Network sockets are disabled; mocked transports do not establish a live registry or GitHub response. |
| Built documentation, internal links, or anchors | `make broken-links-with-anchors` | A freshly built tree passes Mint's filtered link and anchor check | This is not a source-reference or live URL check. |
| Source `@[ref]` references | `make check-cross-refs` | Each eligible reference resolves in every rendering scope | Generated code-sample snippets are excluded. |
| Literal package version in MDX | `uv run python scripts/check_version_claims.py [--files <page> ...]` | A successfully queried PyPI or npm release set contains the named `>=` or `==` version | It intentionally queries registries, and publication does not prove that a feature needs that version. Lookup failures are unresolved, not proof of a bad version. |
| Upstream-owned mirror requirement | `uv run python scripts/check_external_versions.py [--only <id>]` | A registered page's uniquely captured version equals its GitHub file or latest-release source | It intentionally queries GitHub; it applies only to equality claims, not feature floors, and does not validate surrounding requirement prose. |
| Refresh an upstream-owned mirror requirement | `uv run python scripts/check_external_versions.py --write` | Only the captured version digits are synchronized for readable drifted entries | Review the resulting prose; write mode reports unreadable entries without failing. |
| Provider overview or external integration metadata | Run its generator or `uv run python scripts/refresh_integration_downloads.py --check-docs-urls` | Generated output is current, or URL schemes are safe | URL-scheme validation makes neither requests nor writes. |
| Mint export external resources | `make export-htmltest` | Configured external resources in an export pass htmltest | Internal paths and hashes are intentionally disabled. |
| Runnable example | `make test-code-samples [FILES="..."]` | The selected program exits successfully in its actual toolchain and environment | It may need credentials, PostgreSQL, and live providers. |

```mermaid
flowchart TD
  Change["Documentation or code change"] --> Unit["make test"]
  Unit --> Offline["Socket-isolated unit contracts"]
  Change --> Build["Build and Mint checks"]
  Build --> Links["Built links and anchors"]
  Change --> RefCheck["make check-cross-refs"]
  RefCheck --> Maps["Source link-map scopes"]
  Change --> Package["Package-version checker"]
  Package --> Registry["Intentional PyPI or npm lookup"]
  Change --> Mirror["External-version checker"]
  Mirror --> GitHub["Intentional GitHub lookup"]
  Change --> Samples["make test-code-samples"]
  Samples --> Live["Live providers credentials and PostgreSQL"]
  Live --> Trace["Optional trace manifest and snippet links"]
```

This boundary diagram separates offline assertions from deliberate external lookups and live sample execution.

## Socket-isolated pytest suite

Run the core suite with:

```bash
make test
```

`TEST_FILE` defaults to `tests/unit_tests`; for example, run `make test TEST_FILE=tests/unit_tests/test_skills.py` while changing agent-skill contracts. The target runs `uv run pytest --disable-socket --allow-unix-socket $(TEST_FILE) -vv`. Pytest discovers `test_*.py` and `test_*`, uses asyncio auto mode with function-scoped fixture loops, reports extra outcomes, and shows the five slowest tests. Install the test group with `uv sync --group test`.

Socket isolation is an invariant: unit tests must use mocks, temporary files, or permitted Unix sockets rather than opening a network connection. The `file_system` context manager creates disposable `src/` and `build/` trees for filesystem tests. In particular, the focused version-tool tests monkeypatch their HTTP seams; they test parsing, allowlists, error handling, and rewrite behavior without testing external availability. Run the tools themselves separately when the change requires their intentional registry or GitHub boundary.

### Focused contracts

- **Builder:** `test_builder.py` covers copying supported source files, ignoring unsupported extensions, preserving directory structure, preprocessing, and Python/JavaScript builds. See [Builder Tests](/openwiki/testing/builder-tests.md).
- **Parser, rendering, autolinks, and watcher:** retain coverage for Markdown AST and emitted syntax, front matter, headings, code blocks, admonitions, tabs, conditionals, source lines, language-scoped `@[Reference]` replacement, and ignored editor backup/temporary files. Fenced and escaped text must remain protected.
- **Cross-reference checker:** source scanning skips code-sample snippets and `node_modules`, ignores fenced code and escaped references, and requires an unfenced shared OSS reference to resolve for every applicable scope.
- **Agent skills:** structural tests require every `.agents/skills/` directory to have valid matching frontmatter, real referenced repository paths and Make targets, and catalogue rows that agree with the tree. See [Agent Authoring Skills](/openwiki/operations/agent-skills.md).
- **Version tools:** `test_check_version_claims.py` fixes ecosystem-routing precedence, truncated-series matching, ignore parsing, safe package lookup, and outage classification. `test_check_external_versions.py` fixes unique page matching, digit-only rewrites, source/target allowlists, GitHub failure handling, and the difference between check and write exits. It also requires the committed mirror registry to point to existing pages with exactly one match.
- **Integration metadata:** the issue-form parser maps `###` sections without evaluating values, validates required and language-specific package metadata, and returns a nonzero CLI exit for invalid input. `docs_url` tests preserve safe HTTP(S) and site-relative URL handling.

### OpenTelemetry documentation contract

`tests/unit_tests/test_otel_endpoints.py` scans every `.mdx` file below `src`. A generic `OTEL_EXPORTER_OTLP_ENDPOINT` must not carry a `/v1/traces`, `/v1/metrics`, or `/v1/logs` suffix; the HTTP exporter appends its signal path. In documents containing Collector exporters, a full traces URL must use `traces_endpoint`, not generic `endpoint`.

Its runtime cases remain offline: isolated environment dictionaries instantiate `OTLPSpanExporter`, attach it to a `TracerProvider` and `SimpleSpanProcessor`, and mock session `post`. They establish that a trace-specific URL is used unchanged and a generic base gains exactly one `/v1/traces` suffix. Preserve both the authored and mocked-transport contracts when changing [Trace with OpenTelemetry](../../src/langsmith/trace-with-opentelemetry.mdx).

## Version-claim and external-version validation

### Published package versions

`scripts/check_version_claims.py` scans MDX for package specifiers using `>=` or `==`, deduplicates claims while retaining source lines, and queries each distinct package at PyPI or npm. A literal version passes when it was published; a shortened floor such as `1.1` also passes when a published release begins `1.1.`. An ignore file can suppress a known-good unresolved literal specifier. A successful lookup with no matching release is blocking, while an outage, timeout, malformed payload, or unavailable package is reported as unresolved rather than mislabeled unpublished. `--advisory-only` reports but exits zero.

The ecosystem decision is deliberately contextual because a name can exist on both registries with divergent release lines. An npm scope wins first, Python extras next, then the closest same-line Python/JavaScript label, a `:::python` or `:::js` fence, page path, and finally the PyPI default. Before constructing a URL, the checker validates the package name against the relevant allowlist. The command performs up to eight concurrent registry lookups with a 30-second timeout. These are intentional network operations and must not be placed inside the socket-isolated test path.

The read-only **Check version claims** workflow is a 10-minute pull-request gate for changes to `src/**/*.mdx`, the checker, its ignore list, or its workflow. It checks out full history, computes the merge base with the PR base, and invokes `--files` only for changed MDX files below `src`; no changed eligible page skips Python setup and the checker. This scope means the gate proves publishability only for changed documentation, not the whole tree or correctness of an old-but-real feature floor. The scheduled sweep in `refresh-external-versions.yml` fills that coverage gap by checking all pages in advisory mode and publishing its result to the job summary.

### Mirrored upstream requirements

`scripts/check_external_versions.py` addresses a different fact: selected pages repeat a version requirement that another project owns. `scripts/data/external_versions.yaml` registers each true mirror claim with a stable ID, page under `src/`, a page regex with a named `version` group, and either a GitHub-file source with its own named group or a latest GitHub release. The page regex must match exactly once. The loader rejects pages outside `src/`, unsafe repository slugs or upstream paths, and unsupported source types, which protects both URL construction and `--write` edits.

Normal mode fails when a registered entry drifted or cannot be read. `--write` replaces only the captured version span and exits zero even when entries are unreadable, allowing other resolved updates to reach review. Neither mode establishes that an upstream-owned requirement has no changed flags, peer dependencies, or prerequisites; a reviewer must inspect that context. Do not register feature floors: upstream changes cannot determine when the documented feature was introduced.

The trusted refresh workflow runs at 08:00 UTC Monday and on manual dispatch with contents and pull-request write permissions. It provides `GITHUB_TOKEN`, runs `--write`, and creates no branch when `src/` has no diff. Otherwise it saves the generated patch, restores the checkout, and applies it to an open `chore/refresh-external-versions` pull request or a new branch; a second diff check prevents empty commits. The workflow's PR text explicitly requires review of non-version prose changes. This write path is maintenance automation, not a socket-isolated test or a pull-request validation gate.

For implementation changes, run the focused tests through the isolated suite, then explicitly choose a live command when its boundary is relevant:

```bash
make test TEST_FILE=tests/unit_tests/test_check_version_claims.py
make test TEST_FILE=tests/unit_tests/test_check_external_versions.py
uv run python scripts/check_version_claims.py --files src/langsmith/evaluators.mdx
uv run python scripts/check_external_versions.py --only codex-cli
```

See [Version Claim Validation](/openwiki/operations/version-claim-validation.md) for authoring, registry-entry, and review details.

## Documentation, metadata, and export gates

`make broken-links-with-anchors` builds first, then runs Mint from `build/` with anchors enabled; filtering removes known deployment-generated OpenAPI and standalone-snippet noise, while remaining reported links fail the target. `make broken-links` omits anchors. The reusable CI link job uses Python 3.13 and Node 22, runs this target plus `make check-openapi`, and has a 20-minute limit.

`make check-cross-refs` is instead a source-level map check. It reports unresolved file, line, reference, and scope and exits 1. Generated provider-overview validation is another distinct gate: CI regenerates `src/oss/python/integrations/providers/overview.mdx` and fails on a diff, except for the defined automated-update or `bypass-auto-check` exemptions. Change the generator or `packages.yml`, regenerate, and commit the result rather than editing the overview by hand.

The external integration `docs_url` check is intentionally offline: it accepts HTTP(S) and single-slash site-relative URLs, rejects protocol-relative and unsafe schemes, and performs no requests or writes. Full integration-table generation is different: it merges hosted front matter with third-party rows and retrieves npm/PyPI download data over the network.

`make export-htmltest` makes a Mint export, unpacks it, and runs htmltest. Because an export lacks a complete page set, its configuration checks external URLs but disables internal paths and internal hashes; it limits external HTTP concurrency to four and timeout to 30 seconds. The scheduled htmltest workflow runs Monday at 08:00 UTC and can also be dispatched manually.

## Executable code samples

Run all eligible samples, or a space-separated subset, with:

```bash
make test-code-samples
make test-code-samples FILES="src/code-samples/langchain/return-a-string.py"
```

The runner selects `.py`, `.ts`, `.java`, `.kt`, `.go`, and `.sh` files below `src/code-samples`; an explicit `FILES` list is checked for existence, supported extension, and location, while an unset list recursively selects all eligible files outside `__pycache__` and `node_modules`. It runs Python through `uv`, TypeScript through `npx tsx`, Go through `go run`, shell through `bash`, and Java/Kotlin single-file scripts through JBang on Java 21. TypeScript, Go, and shell run from `src/code-samples` so shared dependencies resolve; Python and JBang run from the repository root. `src/code-samples/package.json` owns the Node dependencies used by that TypeScript execution path.

Every invocation inherits the caller environment and is bounded by `CODE_SAMPLE_TIMEOUT_SECONDS`, which defaults to **1,200 seconds per sample**. A timeout, missing executable, or nonzero exit is normally a sample failure. This runner is intentionally live—not an extension of `make test`—so diagnose provider credentials, service readiness, dependencies, and sample behavior separately from a socket-isolated test failure.

### CI selection, services, and credentials

The code-sample workflow runs for relevant pull requests, manual dispatch, and at 00:00 UTC on the first day of each month. It skips fork PRs because repository secrets are unavailable there. PR runs compute the merge-base diff and test only modified eligible sample files; scheduled and manual runs select all samples. The job allows 60 minutes for a PR and 90 minutes for full runs, independently of the per-sample timeout.

CI installs Python/uv, Node 20, Java 21/JBang, and the Go version declared by `src/code-samples/go.mod`. It provisions `pgvector/pgvector:pg17`, waits for the local port, and passes `POSTGRES_URI=postgresql://postgres:postgres@127.0.0.1:5432/postgres?sslmode=disable` to children along with Anthropic, LangSmith/gateway, OpenAI, Tavily, Google, and Daytona credentials. For local PostgreSQL-backed samples, `src/code-samples/conftest.py` first honors `POSTGRES_URI`; otherwise it attempts a pgvector testcontainer, then Docker, then the default local URI. Its store-preparation helper drops the shared store and migration tables before a fresh schema setup.

### Rate limits are skips, not executions

The runner recognizes output that combines `429` with a LangSmith rate-limit phrase. It retries up to three total attempts, waiting 15 seconds between attempts. If all attempts are rate-limited, it records the sample as skipped rather than failing the runner; failures for any other reason remain nonzero. Therefore, a green job with persistent rate-limit skips proves neither that those examples ran successfully nor that their output is current.

### Monthly trace collection and generated links

Scheduled and manual full runs set `CODE_SAMPLE_TRACING=1` and `LANGSMITH_PROJECT=docs-code-samples`. Successful samples then enable `LANGSMITH_TRACING`, and the runner attempts trace collection. A trace-collection error fails the run even if the sample command passed.

For a source file with exactly one `:snippet-start:` marker, collection polls LangSmith for a recent agent-like root run, shares the chosen run publicly, and records source path, URL, run/trace IDs, name, and update time under that snippet ID in `src/code-samples/trace-links.json`. Files with no marker or no qualifying agent run receive no link; multi-snippet files are recorded under `skipped_multi_snippet` and are deliberately excluded until split. `make code-snippets` loads that manifest while generating snippet MDX and appends or replaces a `View example trace` Card only when a URL is present.

After a successful full run and snippet regeneration, CI preserves the manifest and generated snippets, switches to `chore/refresh-code-sample-traces`, and opens or updates a pull request with changes to `src/code-samples/trace-links.json` and `src/snippets/code-samples`. This publication step has write and pull-request permissions; ordinary PR sample checks do not perform it. Locally, `make update-code-sample-traces` supplies tracing, runs the samples, and regenerates snippets; it requires `LANGSMITH_API_KEY`.

A separate `workflow_run` workflow watches scheduled **Test Code Samples** completions. If the scheduled run fails or is cancelled, it creates a Linear issue with the run URL using the configured Linear API key and team key. It does not create tickets for pull-request, manual, or successful scheduled runs.

## CI triage

`ci.yml` runs on pull requests, pushes to `main`, and manual dispatch; concurrency cancels an older run for the same workflow/ref. It invokes reusable test, lint, and documentation-link workflows on Python 3.13 and separately checks merge-conflict markers, cross-references, external integration URLs, and generated files. The changed-document version gate is a separate workflow, while scheduled version maintenance lives in the refresh workflow.

Start triage from the relevant row in the matrix. Treat a unit-test socket error as a test-boundary violation. For a version-claim failure, correct a literal unpublished version; an unresolved lookup is an availability signal, not a reason to invent a version. For an external-version check failure, first distinguish real drift from an unreadable upstream or non-unique pattern; after any automated rewrite, review the surrounding requirement. Treat a generated diff as an update-to-source-or-generator task, and a sample failure as a potentially live-environment problem. Treat a rate-limit skip as unexecuted work—not a passing example—and use the workflow run URL for a scheduled-run escalation.

## Related documentation

- [GitHub Actions and CI/CD](/openwiki/integrations/github-actions.md)
- [Version Claim Validation](/openwiki/operations/version-claim-validation.md)
- [Quickstart](/openwiki/quickstart.md)
- [Builder Tests](/openwiki/testing/builder-tests.md)
- [Code Sample Lifecycle](/openwiki/workflows/code-sample-lifecycle.md)
