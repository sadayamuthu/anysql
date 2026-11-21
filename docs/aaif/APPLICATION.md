# AAIF Project Proposal: draft answers

Draft answers for the
[AAIF Project Proposal form](https://github.com/aaif/project-proposals/issues/new?template=project-proposal.yml),
in form order. Paste each section into the matching field. Items marked
**TODO** need your input before submitting.

Links to GOVERNANCE.md, MAINTAINERS.md, ROADMAP.md, CODE_OF_CONDUCT.md and
docs/aaif/THESIS.md only work after this branch is merged to `main`.

---

## Project Name

anySQL

## Project Description

anySQL is an open-source Python library that records the telemetry of LLM
applications, agents and RAG pipelines, and lets you query it with standard
SQL. Transparent wrappers for the OpenAI and Anthropic clients, an agent
tracer and a RAG tracer normalise telemetry into six canonical Apache Arrow
tables: `llm_responses`, `eval_results`, `pipeline_runs`, `agent_tool_calls`,
`agent_trace` and `rag_chunks`. The tables share join keys: `session_id` and
a per-session `step_order` put model calls, retrievals and tool calls into
one ordered sequence, and `query_id` links retrieved chunks to the
evaluation of the answer they produced. Records are buffered in memory,
optionally persisted to a single SQLite file, and queried in-process with
DuckDB. There is no server, account or network call.

**Why it's valuable.** Cross-layer questions about agents, such as "did this
RAG answer fail at retrieval or at generation?" or "was a value first seen
in a tool output later passed to a state-changing tool call?", become a
single SQL `JOIN` rather than a one-off parser over several log formats.
Because the schema is fixed and documented, analyses are portable across
experiments and can be versioned, hashed and published. Built-in analyses
cover multi-model comparison, prompt regression detection, cost attribution,
agent debugging (tool failure rates, loop detection, session diffs) and RAG
failure-mode classification.

The repository also contains `anysql-proxy`, a local HTTP proxy between AI
coding agents (Claude Code, Cursor, Windsurf, Continue, Zed) and the
OpenAI/Anthropic APIs that logs tokens, cost, latency and editor context to
DuckDB.

**Origin and history.** Started by Karthikeyan Sadayamuthu in March 2026
(v0.1.0 released 2026-03-10). v0.2.0 (November 2026) added the cross-layer
step sequence, used as the measurement substrate of a peer-reviewed study of
prompt-injection detection in agent telemetry (IEEE CSTA 2026, accepted).
The current release is v0.3.1 (2026-11-19), on PyPI as `anysql-sdk` and
archived on Zenodo (DOI 10.5281/zenodo.23980548).

## Preferred Lifecycle Stage

Sandbox

## Alignment with AAIF Mission

The AAIF exists to give agentic AI open, neutrally governed infrastructure.
As agents take actions on users' behalf, being able to inspect what an agent
saw, decided and did, across model calls, retrievals and tool calls, is a
precondition for trusting it. anySQL provides an open, embedded, vendor
neutral way to do that: a documented relational schema for agent telemetry
and plain SQL to query it, with no hosted service.

anySQL directly serves the topics of AAIF's **Observability and
Traceability** working group (agent traces, tool-call provenance) and
**Security and Privacy** working group (detecting prompt-injection
compromises from recorded telemetry), and supports **Accuracy and
Reliability** through regression detection and RAG failure analysis. The
value of a shared telemetry schema grows with the number of tools that use
it, which is best settled under neutral governance early.

## Relation to Existing AAIF Projects

anySQL is complementary to existing AAIF projects and does not overlap with
any of them. It is an analysis layer for the behaviour of agents built with
them:

- **Model Context Protocol (MCP):** MCP standardises how agents call tools;
  anySQL records and analyses those tool calls. MCP tool-call tracing (server
  and tool names into `agent_tool_calls` and the session step sequence) is on
  the roadmap.
- **goose and other agents:** anySQL can record and query the sessions of
  agents like goose, and `anysql-proxy` already records the model calls of AI
  coding agents.
- **AGENTS.md:** no direct relation; anySQL could be used to measure how
  agents behave with and without project instructions.

anySQL also plans import and export for the OpenTelemetry GenAI semantic
conventions, so it interoperates with telemetry collected by other tools
rather than replacing them.

## Example Use Cases and Evidence of Adoption

**Use cases**

1. *Agent debugging:* tool failure rates, loop detection, session diffs and
   human intervention points over `agent_tool_calls` and `agent_trace`.
2. *RAG forensics:* joining `rag_chunks` to `eval_results` on `query_id` to
   classify failures as retrieval or generation.
3. *Agent security:* SQL detection rules over the step sequence, such as an
   identifier first seen in a tool output and later used in a
   state-changing call.
4. *Model evaluation and cost:* multi-model comparison, prompt regression
   detection and cost attribution by feature.
5. *Studying AI coding assistants* with `anysql-proxy`.

**Evidence**

anySQL is early stage and does not yet have documented production use
outside the maintainer. Current evidence:

- Measurement substrate of *Untraced by Design: Hunting Prompt-Injection
  Compromises in LLM Agent Telemetry with SQL*, IEEE CSTA 2026 (accepted). The
  study replays 11,652 recorded AgentDojo runs into anySQL through its public
  tracer API and evaluates six detection queries frozen by hash across four
  held-out models. Artefacts:
  https://github.com/sadayamuthu/untraced-by-design
- Software paper prepared for the Journal of Open Source Software (`paper/`).
- Released on PyPI: https://pypi.org/project/anysql-sdk/
- Archived on Zenodo: https://doi.org/10.5281/zenodo.23980548

The Sandbox thesis explains why the project matters before adoption:
https://github.com/sadayamuthu/anySQL/blob/main/docs/aaif/THESIS.md

## Technical Committee Sponsor

**TODO:** none identified yet. Leave blank, or name a TC member if you reach
one first (for example through the AAIF Discord or the Observability and
Traceability working group).

## GitHub Repository URL

https://github.com/sadayamuthu/anySQL

## License

Apache 2.0

## Governance Model

anySQL uses a maintainer-based model adapted from Minimum Viable Governance,
documented in
[GOVERNANCE.md](https://github.com/sadayamuthu/anySQL/blob/main/GOVERNANCE.md).
Roles are users, contributors, maintainers and a lead maintainer. Decisions
are made by lazy consensus in issues and pull requests; schema-breaking
changes, major releases and governance changes need explicit majority
approval of maintainers, with the lead maintainer breaking ties. There is
currently one maintainer; GOVERNANCE.md sets out how contributors become
maintainers and states the goal of maintainers from at least two
organizations. On admission the project would adopt the LF technical charter.

## CI/CD & Release Workflow

- **CI** (GitHub Actions, `.github/workflows/pr-check.yml`) runs on every
  pull request to `main`: pytest on Python 3.10, 3.11 and 3.12 (about 110 SDK
  tests, plus the proxy's own test suite), `ruff check` and
  `ruff format --check`, and a check that the version has been bumped.
- **Release** (`.github/workflows/release.yml`) runs on merge to `main`: if
  the version in `pyproject.toml` is new, it builds a wheel and sdist, and
  publishes to PyPI from a protected `pypi` environment, then tags and
  creates a GitHub release.
- **Dependencies:** Dependabot opens weekly grouped updates for Python
  packages and GitHub Actions.
- **Release frequency:** five releases so far (v0.1.0 in March 2026; v0.1.1,
  v0.2.0, v0.3.0 and v0.3.1 in November 2026). Releases are cut as features
  land, with semantic versioning; the project is pre-1.0.

## Public-Facing Contribution Process for Specifications

anySQL's specification is its six canonical table schemas, defined in
`anysql/schema.py` and documented in the README, with reference queries in
`docs/QUERIES.md`. Changes go through public GitHub issues and pull requests,
as described in
[CONTRIBUTING.md](https://github.com/sadayamuthu/anySQL/blob/main/CONTRIBUTING.md):
larger changes start with an issue to agree the approach, every behaviour
change needs a test, and CI must pass. Under
[GOVERNANCE.md](https://github.com/sadayamuthu/anySQL/blob/main/GOVERNANCE.md),
removing, renaming or retyping a schema column needs explicit majority
approval of maintainers. A documented schema versioning and migration policy
is planned for 1.0 ([ROADMAP.md](https://github.com/sadayamuthu/anySQL/blob/main/ROADMAP.md)).

## Publicly Accessible Issue Tracker

https://github.com/sadayamuthu/anySQL/issues

## External Project Dependencies

**anysql-sdk, runtime**

| Dependency | License |
|------------|---------|
| duckdb | MIT |
| pyarrow | Apache-2.0 |
| pandas | BSD-3-Clause |

**anysql-sdk, optional extras**

| Dependency | License |
|------------|---------|
| openai | Apache-2.0 |
| anthropic | MIT |
| langchain | MIT |

**anysql-proxy, runtime**

| Dependency | License |
|------------|---------|
| aiohttp | Apache-2.0 |
| duckdb | MIT |
| click | BSD-3-Clause |
| cryptography | Apache-2.0 OR BSD-3-Clause |
| tomli-w | MIT |
| tomli (Python < 3.11) | MIT |

**Development only:** pytest (MIT), pytest-asyncio (Apache-2.0), ruff (MIT),
black (MIT), aioresponses (MIT), hatchling (MIT).

## Maintainers & Contributors

| Name | GitHub | Affiliation | Role |
|------|--------|-------------|------|
| Karthikeyan Sadayamuthu | @sadayamuthu | Independent / OpenAstra | Lead maintainer |

Contribution activity:
https://github.com/sadayamuthu/anySQL/graphs/contributors

anySQL currently has one maintainer and no external contributors. The plan to
grow the contributor base (a documented maintainer process, `help wanted`
areas, and the goal of maintainers from two organizations) is in
[GOVERNANCE.md](https://github.com/sadayamuthu/anySQL/blob/main/GOVERNANCE.md)
and [MAINTAINERS.md](https://github.com/sadayamuthu/anySQL/blob/main/MAINTAINERS.md).

## Leadership Team & Decision Process

Leadership: Karthikeyan Sadayamuthu (lead maintainer).

Decisions are made by lazy consensus in public issues and pull requests (at
least 72 hours for anything beyond a small fix). Schema-breaking changes,
major releases, contested maintainer changes and governance changes need
explicit majority approval of maintainers; the lead maintainer breaks ties
and records the reason publicly. While there is one maintainer, that
maintainer decides and records the reason in the issue or pull request. See
[GOVERNANCE.md](https://github.com/sadayamuthu/anySQL/blob/main/GOVERNANCE.md).

## Lifecycle Stage Evidence

Applying for **Sandbox**. Against the Sandbox acceptance criteria:

- **OSI-approved permissive license:** Apache 2.0.
- **Working implementation:** `anysql-sdk` v0.3.1 on PyPI, with CI on three
  Python versions; used in a peer-reviewed study.
- **At least one actively committing maintainer, with documented intent to
  grow the contributor base:** see GOVERNANCE.md and MAINTAINERS.md.
- **Short written thesis:**
  https://github.com/sadayamuthu/anySQL/blob/main/docs/aaif/THESIS.md
- **Trademark and asset transfer, LF technical charter:** agreed, see the
  checkbox below.

Public roadmap:
https://github.com/sadayamuthu/anySQL/blob/main/ROADMAP.md

## Security

No OpenSSF Best Practices badge yet. Current posture:

- Vulnerabilities are reported privately through GitHub security advisories;
  [SECURITY.md](https://github.com/sadayamuthu/anySQL/blob/main/SECURITY.md)
  sets out a precise threat model and what is in and out of scope.
- anySQL runs in-process and makes no network calls of its own. Recorded
  values are written with parameterised SQLite statements, and the adapters
  do not record API keys, headers or client configuration.
- `anysql-proxy` stores provider API keys encrypted (Fernet) in
  `~/.anysql/keys.toml`.
- PyPI publishing runs from a protected GitHub environment, and GitHub
  Actions workflows use read-only default permissions.
- Dependabot keeps Python and Actions dependencies current.

## Website URL

https://anysql.org

## Documented Governance Practices

- https://github.com/sadayamuthu/anySQL/blob/main/GOVERNANCE.md
- https://github.com/sadayamuthu/anySQL/blob/main/MAINTAINERS.md
- https://github.com/sadayamuthu/anySQL/blob/main/CODE_OF_CONDUCT.md
- https://github.com/sadayamuthu/anySQL/blob/main/CONTRIBUTING.md
- https://github.com/sadayamuthu/anySQL/blob/main/SECURITY.md

## Links to Social Media Accounts

**TODO:** add any, or leave blank.

## Trademark and accounts

Tick the checkbox only if you agree to transfer the anySQL name, the
anysql.org domain, the PyPI projects and other project accounts to the Linux
Foundation if accepted.

## Details of Existing Financial Sponsorship

None. The project has received no external funding.

## Infrastructure Needs or Requests

None beyond the standard Sandbox infrastructure (CI). The project currently
runs on GitHub Actions free tier.

## Additional Information

anySQL is pre-1.0 and early stage, and applies to Sandbox because a shared,
neutrally governed relational schema for agent telemetry is most useful if it
is settled before incompatible schemas proliferate. The project plans to
interoperate with OpenTelemetry's GenAI conventions and MCP rather than
compete with them, and would welcome guidance from the Observability and
Traceability working group.

Note: the repository's `docs.anysql.org` documentation link is not live yet;
a docs site is the first item on the roadmap.

---

## Application contact name(s) and email(s)

Karthikeyan Sadayamuthu, sadayamuthu@outlook.com

## Contributing or sponsoring entity signatory information

**TODO:** decide whether you sign as an individual or through OpenAstra
(the README says "anySQL is managed by OpenAstra").

If an individual:

| Name | Country | Email address |
|------|---------|---------------|
| Karthikeyan Sadayamuthu | United States | sadayamuthu@outlook.com |

If OpenAstra signs, fill in the organization table instead: legal name,
address, entity type (for example a Delaware corporation), signatory name and
title, and email.
