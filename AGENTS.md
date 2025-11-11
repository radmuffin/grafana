# AGENTS.md

<!-- docs-ai-begin -->

# AGENTS.md

 <!-- docs-ai-begin -->

 <!-- version: 1.1.0 -->

## Documentation

Instructions for documentation authoring in Markdown files.

DOCS.md contains all the Docs AI toolkit docs in one file.

## Role

Act as an experienced software engineer and technical writer for Grafana Labs.

Write for software developers and engineers who understand general programming concepts.

Focus on practical implementation and clear problem-solving guidance.

### Grafana

Use full product names on first mention, then short names:

- Grafana Alloy (full), Alloy (short)
- Grafana Beyla (full), Beyla (short)

Use "OpenTelemetry Collector" on first mention, then "Collector" for subsequent references.
Keep full name for distributions, headings, and links.

Always use "Grafana Cloud" in full.

Use complete terms:

- "OpenTelemetry" (not "OTel")
- "Kubernetes" (not "K8s")

Present observability signals in order: metrics, logs, traces, and profiles.

Focus content on Grafana solutions when discussing integrations or migrations.

## Style

### Structure

Structure articles into sections with headings.

Leave Markdown front matter content between two triple dashes `---`.

The front matter YAML `title` and the content h1 (#) heading should be the same.
Make sure there's an h1 heading in the content; this redundancy is required.

Always include copy after a heading or between headings, for example:

```markdown
## Heading

Immediately followed by copy and not another heading.

## Sub heading
```

The immediate copy after a heading should introduce and provide an overview of what's covered in the section.

Start articles with an introduction that covers the goal of the article. Example goals:

- Learn concepts
- Set up or install something
- Configure something
- Use a product to solve a business problem
- Troubleshoot a problem
- Integrate with other software or systems
- Migrate from one thing to another
- Refer to APIs or reference documentation

Follow the goal with a list of prerequisites, for example:

```markdown
Before you begin, ensure you have the following:

- <Prerequisite 1>
- <Prerequisite 2>
- ...
```

Suggest and link to next steps and related resources at the end of the article, for example:

- Learn more about A, B, C
- Configure X
- Use X to achieve Y
- Use X to achieve Z
- Project homepage or documentation
- Project repository (for example, GitHub, GitLab)
- Project package (for example, pip or NPM)

You don't need to use the "Refer to..." syntax for next steps; use the link text directly.

### Copy

Write simple, direct copy with short sentences and paragraphs.

Use contractions:

- it's, isn't, that's, you're, don't

Choose simple words:

- use (not utilize)
- help (not assist)
- show (not demonstrate)

Write with verbs and nouns. Use minimal adjectives except when describing Grafana Labs products.

## Tense

Write in present simple tense.

Avoid present continuous tense.

Only write in future tense to show future actions.

### Voice

Always write in an active voice.

Change passive voice to active voice.

### Perspective

Address users as "you".

Use second person perspective consistently.

### Wordlist

Use allowlist/blocklist instead of whitelist/blacklist.

Use primary/secondary instead of master/slave.

Use "refer to" instead of "see", "consult", "check out", and other phrases.

### Formatting

Use sentence case for titles and headings.

Use inline Markdown links: [Link text](https://example.com).

Link to other sections using descriptive phrases that include the section name:
"For setup details, refer to the [Lists](#lists) section."

Bold text with two asterisks: **bold**

Emphasize text with one underscore: _italics_

Format UI elements using sentence case as they appear:

- Click **Submit**.
- Navigate to **User settings**.
- Configure **Alerting rules**.

### Lists

Write complete sentences for lists:

- Works with all languages and frameworks (correct)
- All languages and frameworks (incorrect)

Use dashes for unordered lists.

Bold keywords at list start and follow with a colon.

### Images

Include descriptive alt text that conveys the essential information or purpose.

Write alt text without "Image of..." or "Picture of..." prefixes.

### Code

Use single code backticks for:

- user input
- placeholders in markdown, for example _`<PLACEHOLDER_NAME>`_
- files and directories, for example `/opt/file.md`
- source code keywords and identifiers,
  for example variables, function and class names
- configuration options and values, for example `PORT` and `80`
- status codes, for example `404`

Use triple code backticks followed by the syntax for code blocks, for example:

```javascript
console.log('Hello World!');
```

Introduce each code block with a short description.
End the introduction with a colon if the code sample follows it, for example:

```markdown
The code sample outputs "Hello World!" to the browser console:

<CODE_BLOCK>
```

Use descriptive placeholder names in code samples.
Use uppercase letters with underscores to separate words in placeholders,
for example:

```sh
OTEL_RESOURCE_ATTRIBUTES="service.name=<SERVICE_NAME>
OTEL_EXPORTER_OTLP_ENDPOINT=<OTLP_ENDPOINT>
```

The placeholder includes the name and the less than and greater than symbols,
for example <PLACEHOLDER_NAME>.

If the placeholder is markdown emphasize it with underscores,
for example _`<PLACEHOLDER_NAME>`_.

In code blocks use the placeholder without additional backticks or emphasis,
for example <PLACEHOLDER_NAME>.

Provide an explanation for each placeholder,
typically in the text following the code block or in a configuration section.

Follow code samples with an explanation
and configuration options for placeholders, for example:

```markdown
<CODE_BLOCK>

This code sets required environment variables
to send OTLP data to an OTLP endpoint.
To configure the code refer to the configuration section.

<CONFIGURATION>
```

Put configuration for a code block after the code block.

## APIs

When documenting API endpoints specify the HTTP method,
for example `GET`, `POST`, `PUT`, `DELETE`.

Provide the full request path, using backticks.

Use backticks for parameter names and example values.

Use placeholders like `{userId}` for path parameters, for example:

- To retrieve user details, make a `GET` request to `/api/v1/users/{userId}`.

### CLI commands

When presenting CLI commands and their output,
introduce the command with a brief explanation of its purpose.
Clearly distinguish the command from its output.

For commands, use `sh` to specify the code block language.

For output, use a generic specifier like `text`, `console`,
or `json`/`yaml` if the output is structured.

For example:

```markdown
To list all running pods in the `default` namespace, use the following command:

<CODE_BLOCK>
```

The output will resemble the following:

```text
NAME                               READY   STATUS    RESTARTS   AGE
my-app-deployment-7fdb6c5f65-abcde   1/1     Running   0          2d1h
another-service-pod-xyz123           2/2     Running   0          5h30m
```

### Shortcodes

Leave Hugo shortcodes in the content when editing.

Use our custom admonition Hugo shortcode for notes, cautions, or warnings,
with `<TYPE>` as "note", "caution", or "warning":

```markdown
{{< admonition type="<TYPE>" >}}
...
{{< /admonition >}}
```

Use admonitions sparingly.
Only include exceptional information in admonitions.

 <!-- docs-ai-end -->

## Build agent instructions

These instructions explain how an automated build agent (CI runner, GitHub Action, or local developer) should prepare the environment, run builds, tests and linters, and produce artifacts for this repository.

### Contract (inputs / outputs / success criteria)

- Inputs: repository checkout at a commit, environment variables described below, network access to fetch modules/packages.
- Outputs: built artifacts in `./bin/` (backend binaries), frontend build output (public assets), test results, and lint reports. Optionally a Docker image if commissioned.
- Success: `make build` completes with exit code 0, `make test` passes, and `make lint-go` returns 0. Artifacts are uploaded or stored as CI job artifacts.

### Environment (recommended)

- OS: Linux (Ubuntu/CentOS) — CI images normally use Ubuntu runners.
- Go: use the repository Makefile's `GO_VERSION` (see `Makefile` top for exact version). At time of writing Makefile requests Go 1.25.3; prefer a matching or newer patch in the same minor line.
- Node: Node 18+ (LTS) for frontend builds and tooling. `yarn` is used for JS tasks; the repo contains `yarn.lock`.
- Yarn: Yarn 1.x classic (the Makefile uses `yarn install --immutable`). Ensure `yarn` is available in PATH.
- Docker: Optional, required if you build container images.
- Required tools available on PATH: git, make, bash, unzip, tar, and common CI utilities.

### Key repository targets used by agents

Use the project's Makefile targets so local and CI builds behave the same. Common targets:

- `make deps` — install all dependencies (calls `deps-js` which installs frontend deps with yarn).
- `make build` — builds backend and frontend (`build-go` + `build-js`). Produces backend binaries under `./bin/`.
- `make test` — runs backend and frontend tests (`test-go` + `test-js`).
- `make lint-go` — runs golangci-lint checks for backend.
- `make run` / `make run-go` / `make run-frontend` — useful for local development / smoke runs.

These targets are canonical and used by CI; prefer them over ad-hoc commands.

### Minimal setup sequence for a CI job

1.  Checkout code (shallow clone OK depending on CI needs).
2.  Ensure correct Go and Node versions are installed (toolcache or setup actions in CI).
3.  (Optional) set ENV flags used by the Makefile, e.g.: `GO_RACE=1` to enable race detector where appropriate, or `SHARD`/`SHARDS` for sharded tests.
4.  Install dependencies and build:

```sh
# Install system deps if needed (done in CI image)
make deps

# Build everything (backend + frontend)
make build
```

5.  Run tests and linters (separate steps recommended so CI can cache artifacts):

```sh
make test
make lint-go
```

6.  Collect artifacts: `./bin/` (binaries), frontend build outputs (packaged in the repo), and test reports.

### Running focused builds and tests

- Backend only: `make build-backend` or `make build-go`.
- Frontend only: `make build-js` (requires `deps-js`).
- Single backend package tests: use `make test-go-unit FILES=./pkg/yourpkg` or run `go test` directly against the package.

### CI tips and reliability

- Use `make deps` as the first Makefile step so node modules and Go workspace are prepared.
- Cache `~/.cache/go-build`, `GOMODCACHE` and `node_modules` / Yarn cache between CI runs to speed builds.
- Tests are sharded in CI using `SHARD` and `SHARDS` environment variables; replicate CI sharding locally when running a subset of tests.
- For integration tests that require services (Postgres, Redis, Alertmanager), use the `devenv/` scripts or the CI-provided service containers. Many `make` test targets (e.g., `test-go-integration-postgres`) call `devenv-*` targets.

### Common environment variables

- `GO_RACE` — when set enables race detector flags used by tests/build.
- `GO_BUILD_TAGS` — to pass build tags (enterprise/pro) when needed.
- `SHARD`, `SHARDS` — control test sharding.
- `GRAFANA_TEST_DB` — used for integration test databases (postgres/mysql).

### Producing Docker images (optional)

If your CI needs to produce Docker images, follow this pattern:

1.  Run `make build` to produce binaries.
2.  Use a Dockerfile (repo contains packaging assets) to copy `./bin/grafana` and other assets into an image.
3.  Tag and push the image from CI.

Note: Grafana has packaging tooling and several image variants; prefer existing build scripts in `packaging/` if present.

### Troubleshooting common failures

- Node/yarn errors: ensure Yarn 1.x (classic) and Node LTS; delete `node_modules` and the yarn cache and retry `yarn install --immutable`.
- Go module errors: ensure `GOPROXY` is set (or network access), run `make deps-go` which runs `go run build.go setup` to prepare workspace.
- Linter failures: run `make lint-go` locally and fix reported issues or run the linters with the same configuration used in CI.

### Local developer quickstart (short)

1.  Install Go and Node matching the environment.
2.  From repo root:

```sh
make deps      # install node modules + go setup
make build     # build backend and frontend
make run       # run server in dev mode (optional)
```

### Next steps for agent authors / CI integrators

- Add cache layers for `node_modules` and Go module cache to speed CI.
- Split steps so `deps` runs once and artifacts (frontend build, Go build caches) can be reused between jobs.
- Add artifact upload/retention for `./bin/`, test-report JUnit XML (frontend tests) and linter reports.

### Notes and assumptions

- These instructions intentionally use the repository's Makefile targets (example: `make build`, `make test`, `make deps`) so they remain correct if the internal build commands change.
- If your CI uses a non-Linux runner, adjust system package commands accordingly.

 <!-- docs-ai-end -->

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- You have access to the Nx MCP server and its tools, use them to help the user
- When answering questions about the repository, use the `nx_workspace` tool first to gain an understanding of the workspace architecture where applicable.
- When working in individual projects, use the `nx_project_details` mcp tool to analyze and understand the specific project structure and dependencies
- For questions around nx configuration, best practices or if you're unsure, use the `nx_docs` tool to get relevant, up-to-date docs. Always use this instead of assuming things about nx configuration
- If the user needs help with an Nx configuration or project graph error, use the `nx_workspace` tool to get any errors

<!-- nx configuration end-->
