# anySQL: Sandbox Thesis

*Short written thesis for an AAIF Sandbox application, as the
[Project Lifecycle Policy](https://github.com/aaif/technical-committee/blob/main/governance/project-lifecycle-policy.md)
requires: who would use the project, what would justify graduation to Growth,
and why it matters before adoption.*

## The problem

Agents fail in ways that span layers. A wrong answer can come from a bad
retrieval, a bad generation, a failed tool call, a loop, or a prompt injection
that arrived in a tool output and steered a later action. The evidence for
each of these sits in a different place: provider logs, framework callbacks,
vector store results, eval harness output and ad hoc JSON files, each with its
own schema.

Existing observability platforms collect this as span trees and show it
through dashboards. They are built to operate production systems, and they
answer the questions their designers anticipated. What they don't give an
engineer, researcher or security analyst is **one relational schema over all
layers of an agent session, queried with plain SQL**, so that a question like
"did this RAG answer fail at retrieval or at generation?" or "was an
identifier first seen in a tool output later passed to a state-changing tool
call?" is one `JOIN`, not a one-off parser.

## What anySQL is

anySQL is an Apache 2.0 Python library, released on PyPI as `anysql-sdk`
(current version 0.3.1). Transparent wrappers for the OpenAI and Anthropic
clients, an agent tracer and a RAG tracer normalise telemetry into six
canonical Apache Arrow tables (`llm_responses`, `eval_results`,
`pipeline_runs`, `agent_tool_calls`, `agent_trace`, `rag_chunks`). The tables
share join keys: `session_id` and a per-session `step_order` put model calls,
retrievals and tool calls into one ordered sequence, and `query_id` links
retrieved chunks to the evaluation of the answer they produced. DuckDB
queries them in-process; SQLite persists them to one file. There is no
server, account or network call.

The repository also contains `anysql-proxy`, a local proxy that logs the
model calls of AI coding agents (Claude Code, Cursor and others) to DuckDB,
so their use can be studied with the same SQL.

## Who would use it

1. **Agent and RAG engineers** debugging a pipeline locally: comparing models,
   catching prompt regressions, attributing cost, finding loops and failing
   tools.
2. **Researchers evaluating agents**, who need analyses that are versioned,
   reproducible and portable across experiments. A fixed schema means a query
   written for one experiment runs unchanged on another, and queries can be
   frozen by hash before a held-out split is evaluated.
3. **Security teams hunting agent compromises** in recorded telemetry, where
   detection rules are SQL over the step sequence.
4. **Teams studying AI coding assistants**, through `anysql-proxy`.

## Why it matters before adoption

- **It has already carried a peer-reviewed result.** anySQL is the
  measurement substrate of *Untraced by Design: Hunting Prompt-Injection
  Compromises in LLM Agent Telemetry with SQL* (IEEE CSTA 2026, accepted). The
  study replays 11,652 recorded AgentDojo runs into anySQL through its public
  tracer API and evaluates six detection queries, written on one model,
  frozen by hash and run unchanged on four held-out models. The cross-layer
  step sequence was added in 0.2.0 to make that analysis possible. The
  artefacts are public at
  [sadayamuthu/untraced-by-design](https://github.com/sadayamuthu/untraced-by-design).
  A software paper is prepared for JOSS, and the code is archived on Zenodo
  ([10.5281/zenodo.23980548](https://doi.org/10.5281/zenodo.23980548)).
- **A shared relational schema for agent telemetry is a commons question.**
  The value of a schema grows with the number of tools that read and write
  it. That is best decided under neutral governance early, before
  incompatible schemas settle, which is the case the Sandbox stage exists
  for. It also fits the work of AAIF's Observability and Traceability working
  group.
- **It complements, rather than competes with, existing standards.** The
  roadmap adds import and export for the OpenTelemetry GenAI semantic
  conventions and tracing for MCP tool calls, so anySQL becomes an SQL
  analysis layer over telemetry other tools already collect, including
  agents built on AAIF projects.

## What would justify graduation to Growth

Within twelve months of admission, anySQL would apply for Growth when it has:

1. **Adoption:** documented production or research use by at least two
   organizations unaffiliated with the maintainer.
2. **Community:** commits from at least two organizations over the prior six
   months, at least one maintainer from a second organization, and the
   committer-acceptance process in [GOVERNANCE.md](../../GOVERNANCE.md) in
   use.
3. **Technical:** OpenTelemetry GenAI import and export and MCP tool-call
   tracing released, a published docs site, and a documented schema
   stability and migration policy ahead of 1.0
   (see [ROADMAP.md](../../ROADMAP.md)).
4. **Plan:** a written growth plan accepted by a TC sponsor, including the
   project's own definition of Impact.

If those signals don't appear in twelve months, archival is a reasonable
outcome, and the schema and queries remain available under Apache 2.0.

## Risks

- **Single maintainer.** Mitigated by the governance and maintainer process
  in GOVERNANCE.md, `help wanted` areas in MAINTAINERS.md, and neutral hosting,
  which lowers the barrier for other organizations to contribute.
- **No external adoption yet.** The case for Sandbox rests on the research
  use above and on the value of an early neutral schema.
- **Overlap with observability platforms.** anySQL is deliberately an
  embedded analysis layer, and plans to interoperate with OpenTelemetry
  rather than replace it.
