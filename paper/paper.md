---
title: 'anySQL: SQL analytics over LLM, agent and RAG telemetry'
tags:
  - Python
  - large language models
  - LLM agents
  - retrieval-augmented generation
  - observability
  - DuckDB
  - SQL
authors:
  - name: Karthikeyan Sadayamuthu
    orcid: 0009-0008-3539-6048
    affiliation: 1
affiliations:
  - name: Independent Researcher, United States
    index: 1
date: 19 November 2026
bibliography: paper.bib
---

<!-- DRAFT. Not for submission before JOSS's six-month public-development gate is met
     (target: May 2027). Update version, dates and the research-impact section before
     submitting. -->

# Summary

anySQL is a Python library that records the telemetry of large language model (LLM)
applications and lets researchers query it with standard SQL. It wraps the OpenAI and
Anthropic Python clients, and provides an agent tracer and a retrieval-augmented
generation (RAG) tracer. Each source is normalised into one of six canonical Apache Arrow
tables: `llm_responses`, `eval_results`, `pipeline_runs`, `agent_tool_calls`,
`agent_trace` and `rag_chunks`. Records are buffered in memory, optionally persisted to
SQLite, and exposed to DuckDB [@raasveldt2019duckdb] as views at query time, so a session
can be analysed with a single SQL statement and no server.

The tables share join keys. `session_id` and a per-session `step_order` link model calls,
retrievals and tool calls made in one agent session into one ordered sequence, and
`query_id` links retrieved chunks to the evaluation results of the answer they produced.
These keys make cross-layer questions answerable in SQL, for example whether a RAG answer
failed at retrieval or at generation, or whether a value that first appeared in a tool
output was later used as an argument to a state-changing tool call.

# Statement of need

Researchers who study LLM agents and RAG pipelines need to ask questions that span layers
of the system: what the model was asked, what it retrieved, which tools it called, and in
what order. In most setups this evidence is split across provider logs, framework
callbacks and ad hoc JSON files, each with its own schema. Answering a cross-layer
question then means writing a one-off parser for each experiment, which is slow, hard to
review and hard to reproduce.

anySQL targets researchers and engineers who evaluate LLM systems and want that evidence
in one relational form. Its design choices follow from that use:

- **A fixed, documented schema.** Six tables with typed Arrow schemas mean that an analysis
  written for one experiment runs unchanged on another, and that queries can be frozen
  before a test split is evaluated.
- **SQL as the analysis language.** Detection rules, metrics and comparisons are plain SQL
  files that can be versioned, hashed and published alongside results.
- **Local and embedded.** DuckDB runs in-process and SQLite persistence is a single file,
  so a study's full telemetry can be archived and replayed without a hosted service.
- **Ingestion through a public tracer API.** Recorded runs from existing benchmarks can be
  replayed into anySQL, not only live traffic.

anySQL also ships built-in analyses for five common tasks: multi-model comparison, prompt
regression detection, cost attribution, agent debugging (tool failure rates, loop
detection, session diffs) and RAG failure-mode classification.

# State of the field

Hosted and self-hosted LLM observability platforms such as Langfuse [@langfuse] and
Arize Phoenix [@phoenix] collect traces and present them through dashboards and their own
query interfaces. The OpenTelemetry semantic conventions for generative AI [@otelgenai]
standardise span attributes for model calls. These tools are aimed at operating
production systems. anySQL differs in three ways that matter for research use: it needs
no server or account; its unit of analysis is a relational table rather than a span tree,
so cross-layer joins are ordinary SQL; and its schema includes evaluation results and
retrieved chunks as first-class tables keyed to the responses they belong to. anySQL can
complement these platforms by serving as an offline analysis layer over exported or
replayed telemetry.

# Software design

anySQL has four layers. *Adapters* wrap provider clients transparently, so an existing
`chat.completions.create` or `messages.create` call is logged without code changes.
*Tracers* record agent steps, tool calls and retrievals; the RAG tracer accepts LangChain
and LlamaIndex document objects as well as plain dictionaries. A *context* layer based on
Python `contextvars` attaches feature names, tags and session identifiers to every record
in a scope, in both synchronous and asynchronous code. The *engine* validates records
against the Arrow schemas, assigns step numbers to sequenced tables under a lock, persists
records to SQLite as JSON, and registers each table with DuckDB as a view. Validation
happens when tables are materialised, so storage stays schema-tolerant while queries see
typed columns.

The repository also contains `anysql-proxy`, a separate package that runs as a local HTTP
proxy between an AI coding assistant and the OpenAI or Anthropic API. It forwards
requests unchanged and logs token counts, cost, latency and editor context to a local
DuckDB file, so the use of coding assistants can be studied with the same SQL approach.

The library is tested with pytest, including tests that check that model calls,
retrievals and tool calls in one session share a single step sequence. It is released on
PyPI as `anysql-sdk` under the Apache 2.0 licence, and archived on Zenodo
[@anysql_zenodo].

# Research impact statement

anySQL is the measurement substrate of @sadayamuthu2026untraced, which studies what a
security team can detect with SQL over recorded LLM agent sessions. That study replays
11,652 recorded runs from the AgentDojo benchmark [@debenedetti2024agentdojo] into anySQL
through its public tracer API, and evaluates six detection queries that were written on
one model, frozen by hash, and run unchanged on four held-out models. The cross-layer step
sequence in anySQL 0.2.0 was added to make that analysis possible: it lets a query test
whether an identifier first seen in a tool output was later used in a state-changing call.
The study's queries, loaders and results are public [@untraced_artifacts].

<!-- TODO before submission: add any further use (other papers, preprints, external
     users, issues or PRs from non-authors). -->

# AI usage disclosure

<!-- TODO: complete accurately before submission. JOSS requires: tool names and versions,
     where they were used (code, tests, documentation, this paper), the scope of the
     assistance, and confirmation of human review. Example wording below; edit to match
     what actually happened. -->

Generative AI tools were used in developing this software and in preparing this paper.
Anthropic's Claude (via Claude Code; models: TODO) was used to assist with
TODO: [e.g. implementation of adapters and tracers, writing tests, documentation, and
drafting this manuscript]. The author defined the problem, the table schema, the join-key
design and the cross-layer step sequence, and made all architectural decisions. All
AI-generated code, tests, documentation and text were reviewed, modified where needed, and
validated by the author, who takes full responsibility for the content.

# Acknowledgements

<!-- TODO: funding statement, or "This work received no external funding." -->

# References
