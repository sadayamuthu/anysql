# Roadmap

What anySQL plans to work on next. It's a plan, not a promise: priorities
change with what users report. Discuss any item in an issue before starting
on it.

## Now: 0.4

- **Docs site** at docs.anysql.org: the schema reference, the UC1 to UC5
  methods, and the query library in [docs/QUERIES.md](docs/QUERIES.md).
- **LangChain callback handler**: a real `BaseCallbackHandler` subclass, so
  `AgentTracer` can be passed in `callbacks=[...]`.
- **Publish `anysql-proxy` to PyPI**, so it no longer needs a git install.
- **Contributor onboarding**: `good first issue` labels, and issue and pull
  request templates.

## Next: 0.5 to 0.6

- **OpenTelemetry GenAI import and export.** Read spans that follow the
  OpenTelemetry semantic conventions for generative AI into the six tables,
  and write them back out, so anySQL can be an SQL layer over telemetry that
  other tools collect.
- **MCP tool-call tracing.** Record Model Context Protocol tool calls, with
  the server and tool names, into `agent_tool_calls` and the session's step
  sequence.
- **More adapters**: the OpenAI Responses API, Google Gemini, and AWS Bedrock.
- **Agent security queries**: ship the provenance queries from the
  "Untraced by Design" study (a value first seen in a tool output and later
  used in a state-changing call) as built-in methods.

## Later: 1.0

- **Schema stability.** Freeze the six canonical tables, document a
  versioning and migration policy, and only change them in a major release.
- **Parquet export** of all tables, for use with other analysis tools.
- **Stable public API** for adapters and tracers, so third parties can write
  their own.

## Community goals

- Maintainers from at least two organizations (see
  [GOVERNANCE.md](GOVERNANCE.md)).
- Documented production or research use by at least two unaffiliated
  organizations.
