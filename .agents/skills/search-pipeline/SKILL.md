---
name: search-pipeline
description: Build and improve the NeuroAI Evidence Explorer V0 paper search pipeline using the Semantic Scholar Academic Graph API. Use this skill for research-question input, Semantic Scholar requests, top-5 retrieval, response parsing, paper metadata normalization, search error handling, JSON persistence, and retrieval observation. Do not continue into evidence packet generation, analysis validation, report building, or later V0 stages.
---

# Search Pipeline

## Purpose

Use this skill to build, inspect, test, or improve the paper retrieval stage of NeuroAI Evidence Explorer V0.

This skill owns the pipeline:

Research Question  
→ Semantic Scholar Search  
→ API Response  
→ Paper Metadata Normalization  
→ Paper Validation Boundary  
→ `papers.json`

Its goal is not to build a sophisticated academic search engine.

Its goal is to create the smallest reliable retrieval pipeline needed for V0 while exposing the user to important AI Engineering concepts such as:

- HTTP requests
- query parameters
- API contracts
- external-service failure
- timeout
- HTTP status codes
- rate limiting
- missing data
- response parsing
- schema boundaries
- retrieval relevance
- Precision@5
- reproducible search observations

The pipeline should remain simple enough that the user can understand the full request-to-file flow.

---

## 1. Scope

This skill is responsible for:

- accepting a NeuroAI research question
- converting the question into the query sent to Semantic Scholar
- calling the Semantic Scholar Academic Graph paper search endpoint
- requesting only V0-required metadata
- retrieving up to 5 papers
- parsing the API response
- normalizing external field names into the project's internal representation
- preserving missing optional fields
- handling expected external-service failures
- producing data suitable for the `Paper` schema
- saving validated or normalized search results to `papers.json`
- exposing search latency when required
- helping the user inspect retrieval quality
- preparing results for later Precision@5 evaluation

This skill stops after the search result artifact has been produced and observed.

---

## 2. Out of Scope

Do NOT implement any of the following while using this skill:

- `evidence_packet.md`
- ChatGPT analysis
- `analysis.json`
- analysis schema validation
- Markdown report generation
- final evidence brief
- Evidence / Interpretation / Hypothesis synthesis
- full Baseline vs V0 evaluation
- PDF download
- PDF parsing
- full-text search
- embeddings
- vector database
- RAG
- reranking
- LLM API
- database storage
- FastAPI
- Streamlit
- React
- web UI
- Docker
- CI/CD
- cloud deployment
- multi-agent architecture

If the requested task naturally belongs to a later pipeline stage, stop and explain which stage should handle it.

Do not silently expand the search skill into the rest of the application.

---

## 3. Stage Boundary

This skill begins with:

### Input

A plain-text NeuroAI research question.

Example:

> How does hippocampal replay differ from experience replay in reinforcement learning?

This skill ends with:

### Output

A search result artifact containing up to five papers.

Target file:

`data/<question-slug>/papers.json`

The result should contain normalized paper metadata that can later be consumed by the evidence-packet stage.

This skill does NOT decide the final scientific answer to the research question.

Retrieval results are candidate evidence, not conclusions.

---

## 4. Core Retrieval Principle

Always preserve the distinction:

Search Result ≠ Scientific Evidence ≠ Final Answer

Semantic Scholar relevance ranking provides candidate papers that may be useful for the research question.

A paper appearing in the top five does not prove that:

- it directly answers the question
- its abstract contains all needed information
- its methodology is comparable to another paper
- its findings support a later interpretation
- it should automatically appear in a final scientific conclusion

The search pipeline retrieves candidates.

Human evaluation later determines relevance.

Do not treat ranking position as scientific confidence.

---

## 5. V0 Search Contract

V0 uses the Semantic Scholar Academic Graph API paper search endpoint.

Conceptual request:

`GET /graph/v1/paper/search`

The request should contain:

- a plain-text `query`
- `limit=5`
- only fields required by V0

Target fields include:

- `paperId`
- `title`
- `abstract`
- `authors`
- `year`
- `url`
- `citationCount`

If the implementation also requests a field already justified by the V0 plan, keep the request minimal.

Do not request large amounts of unused metadata.

Do not introduce advanced Semantic Scholar search syntax unless explicitly required.

The research question should initially be treated as a plain-text relevance query.

---

## 6. Result Limit

The default V0 result limit is:

`5`

Do not increase the result count merely because retrieval quality is disappointing.

The purpose of the initial V0 is to observe whether Semantic Scholar top-5 relevance search is sufficient.

If Precision@5 is poor, record that result before changing the search strategy.

Changing the result count changes the experiment.

Do not silently alter evaluation conditions.

---

## 7. External API Boundary

Treat Semantic Scholar as an external and unreliable system boundary.

Never assume:

- the service is always available
- HTTP 200 is always returned
- every field exists
- every paper has an abstract
- every response matches expectations
- every query produces results
- the response arrives before the timeout
- rate limits will never be reached

The application must fail intentionally and understandably.

External-service uncertainty is part of the system design.

---

## 8. Request Lifecycle

When implementing or reviewing the search flow, reason about the request in this order:

Research Question  
→ Query String  
→ HTTP Request  
→ Status Code  
→ Response Body  
→ Response Parsing  
→ Metadata Normalization  
→ Paper Contract  
→ Persistence

Do not jump directly from the research question to `papers.json` conceptually.

The user should be able to identify where a failure occurred within this lifecycle.

For meaningful failures, use `$debug-learning`.

---

## 9. Query Handling

The initial V0 should prefer the simplest possible query behavior.

Start with the research question or an intentionally simple transformation of it.

Do not automatically add:

- complex Boolean expressions
- citation filters
- year filters
- venue filters
- field-of-study filters
- manually curated synonyms
- embedding-based expansion
- LLM-generated query expansion

unless a measured retrieval failure creates a reason to test one of them.

The initial system is a baseline search pipeline.

Complex retrieval logic should be earned through observed failure.

---

## 10. Query Normalization

Query normalization is optional during the initial V0 search implementation.

One known experiment may involve terms such as:

`self-supervised`

versus:

`self supervised`

Do not implement broad normalization before observing a relevant failure.

If normalization is tested:

1. preserve the original query
2. record the normalized query
3. run both under comparable conditions
4. compare retrieval results
5. compare Precision@5 where appropriate
6. document whether normalization actually improved retrieval

Do not declare normalization useful without comparison.

---

## 11. Paper Metadata Contract

The search pipeline should produce data compatible with the project's conceptual `Paper` contract:

`paper_id: str`

`title: str`

`abstract: str | None`

`authors: list[str]`

`year: int | None`

`url: str | None`

`citation_count: int | None`

`source: Literal["semantic_scholar"]`

Semantic Scholar external field names may differ from internal project names.

For example:

`paperId`  
→ `paper_id`

`citationCount`  
→ `citation_count`

Normalize external data at the boundary.

Do not allow Semantic Scholar-specific naming conventions to leak unnecessarily throughout the application.

---

## 12. Authors

Semantic Scholar may return author objects rather than simple strings.

The internal V0 representation should remain simple:

`authors: list[str]`

Extract only the author names required by the project contract.

Do not build an author entity system.

Do not add:

- author IDs
- affiliations
- profiles
- citation networks

unless the project scope explicitly changes.

---

## 13. Missing Abstracts

A missing abstract is expected external-data behavior.

If Semantic Scholar returns:

`abstract = null`

preserve it as:

`None`

or its valid JSON equivalent:

`null`

Do not:

- invent an abstract
- use outside knowledge to fill it
- silently remove the paper
- replace it with a fabricated summary
- make the whole search fail solely because one abstract is missing

The missing value should remain visible to downstream stages.

This is an important V0 learning case for optional fields and incomplete external data.

---

## 14. Missing Optional Metadata

The same principle applies to optional fields such as:

- year
- URL
- citation count

If the project schema allows the field to be optional, preserve the absence.

Do not manufacture default scientific metadata.

Avoid misleading defaults such as:

`year = 0`

or:

`citation_count = 0`

unless zero is explicitly known to be the real value.

Missing data and zero are not the same thing.

---

## 15. Required Metadata

Fields required by the internal contract should be handled deliberately.

At minimum, a usable search result should have enough identity to represent a paper in the pipeline.

If a required field such as `paper_id` or `title` is missing or malformed:

- do not fabricate it
- do not silently pass invalid data downstream
- allow schema validation or explicit handling to reveal the contract failure

The search pipeline should make bad external data visible rather than hide it.

---

## 16. Source Provenance

Every normalized paper must preserve its source as:

`semantic_scholar`

Source provenance is part of the reliability design.

Do not remove source identity merely because the project currently uses only one academic API.

The later evidence pipeline must be able to identify where the metadata came from.

---

## 17. Search Output Artifact

The target search artifact is:

`data/<question-slug>/papers.json`

The artifact should be:

- valid JSON
- human-readable
- deterministic in structure
- compatible with the Paper schema
- sufficient for later evidence-packet generation

Do not add unrelated derived analysis to `papers.json`.

For example, do not insert fields such as:

- `scientific_strength`
- `paper_quality`
- `supports_question`
- `importance_score`
- `AI_summary`
- `method_quality`

unless the V0 contract explicitly defines them.

Search output should remain retrieval data, not scientific judgment.

---

## 18. Slug Handling

If the research question is used to generate an output directory or filename slug:

- keep the implementation simple
- make the result filesystem-safe
- keep behavior deterministic
- do not spend excessive time designing perfect slug rules

CLI and filename polish are not the objective of V0.

If slug behavior becomes a blocker, use the smallest understandable solution.

---

## 19. Search Success Definition

A successful initial search stage means:

- the CLI or search function receives the research question
- Semantic Scholar is called correctly
- up to five paper records are returned
- external metadata is normalized
- optional missing values are preserved
- invalid required data is not fabricated
- results are saved to `papers.json`
- the user can inspect what was retrieved

Success does NOT mean:

- all five papers are relevant
- all five papers have abstracts
- the system answered the research question
- retrieval quality is already optimal

Retrieval quality is something to measure after the pipeline works.

# 2/3 — Failure Handling, Testing, and Search Observation

## 20. Zero Search Results

A successful HTTP request may legitimately return zero papers.

Zero results are not automatically an application error.

The search pipeline must distinguish between:

- the HTTP request failed
- the API returned malformed data
- the API successfully returned zero results

These are different situations.

If Semantic Scholar returns zero results:

- do not crash
- do not fabricate papers
- do not silently replace the query
- do not automatically broaden the search
- clearly expose that zero papers were returned
- still produce predictable application behavior

The user should be able to observe:

`0 papers retrieved`

before deciding whether query design should be investigated.

If zero results appear unexpected, use `$debug-learning`.

Possible investigation areas include:

- query wording
- terminology mismatch
- hyphenation
- interdisciplinary terminology
- actual request parameters
- Semantic Scholar search behavior

Do not modify query strategy before establishing a hypothesis.

---

## 21. HTTP Status Handling

Always inspect the HTTP status code before parsing the response as successful paper data.

Do not assume that every response body contains the expected Semantic Scholar schema.

Handle at least the V0-relevant cases:

- successful response
- HTTP 429
- other non-success HTTP responses when encountered

Do not silently convert failed HTTP responses into empty paper lists.

For example:

HTTP 429

must not become:

`papers = []`

because these two states mean different things.

One means:

> The search succeeded and found no papers.

The other means:

> The external service rejected or limited the request.

Preserve this distinction.

---

## 22. HTTP 429 — Rate Limiting

HTTP 429 is an expected external-service failure mode.

The V0 search pipeline must handle it intentionally.

When HTTP 429 occurs:

- detect the status explicitly
- do not attempt to parse it as a normal paper response
- provide a human-readable error
- preserve enough context to understand that rate limiting occurred
- avoid infinite retry loops

Example conceptual behavior:

> Semantic Scholar rate limit reached. Please retry later.

The exact wording may vary.

The important behavior is that the user understands:

- the API request reached the service
- the service rejected it because of request-limit behavior
- this is not the same as invalid JSON or zero results

Do not hide HTTP 429 behind a generic message such as:

> Something went wrong.

---

## 23. Retry Policy

Retry behavior should remain minimal in V0.

Do not build a sophisticated resilience framework.

If retry is introduced, it must be justified by an observed failure or the explicit V0 acceptance criteria.

Possible retry targets include transient conditions such as:

- rate limiting
- temporary network failure

Avoid retrying failures that are unlikely to improve through repetition.

Examples:

- invalid query construction caused by our own code
- invalid schema
- malformed local data
- deterministic parsing bugs

A retry should not be used to hide application bugs.

---

## 24. Retry Boundaries

If retry behavior exists:

- use a finite retry count
- keep behavior understandable
- avoid recursive retry logic
- avoid infinite loops
- avoid excessive waiting
- surface failure after retries are exhausted

The user should still know that retries occurred.

Do not make the external API appear perfectly reliable by hiding repeated failures.

The reliability lesson is that external systems may fail and our system should respond predictably.

---

## 25. Backoff

Backoff may be introduced if HTTP 429 or transient failure makes it necessary.

Keep the implementation minimal.

The conceptual purpose of backoff is:

> Do not immediately repeat a request at the same rate after the service has already indicated that requests should slow down.

Do not implement advanced distributed-systems retry infrastructure.

For V0, prioritize understanding over sophistication.

If backoff is added because of an observed failure:

1. record the original behavior
2. define the hypothesis
3. define the retry/backoff change
4. re-run under comparable conditions when practical
5. record whether behavior improved

Use `$debug-learning` for this process.

---

## 26. Timeout Handling

Every external HTTP request should have an intentional timeout.

Do not allow a request to wait indefinitely.

Timeout behavior should be understandable to the user.

If a timeout occurs:

- detect it explicitly
- do not convert it into zero results
- provide a readable message
- preserve the distinction between network/service failure and valid search output

Conceptually:

`Research Question`
→ request sent
→ no response within allowed time
→ timeout failure

A timeout is an external boundary failure, not a retrieval-quality score.

Do not increase the timeout indefinitely merely to make the error disappear.

---

## 27. Timeout Configuration

Use a reasonable and explicit timeout value.

Do not optimize the exact value prematurely.

The important V0 principle is:

> External requests must have bounded waiting time.

If the timeout value later becomes a problem:

- observe actual latency
- form a hypothesis
- measure before changing it

Avoid magic values scattered across multiple files.

Keep request configuration understandable.

---

## 28. Network Failure

Network-related failures should not be treated as valid empty search responses.

Examples may include:

- connection errors
- DNS-related failure
- connection interruption
- request timeout

Handle them at the request boundary.

Do not fabricate API results.

Do not continue downstream with misleading data.

Prefer an explicit failure message that allows the user to understand which stage failed.

If the root cause is unclear, invoke `$debug-learning`.

---

## 29. Malformed or Unexpected Response

Even an external API may return data that does not match our expectations.

Possible cases include:

- missing top-level `data`
- `data` not being the expected type
- missing required paper identity fields
- unexpected field types
- partially malformed records

Do not assume response shape only because HTTP status is successful.

The search pipeline should intentionally cross this boundary:

External JSON  
→ parsing  
→ normalization  
→ schema validation

Do not silently coerce arbitrary malformed data into valid-looking paper records.

---

## 30. Parsing vs Validation

Keep these concepts distinct.

### Parsing

Converts the external response into Python data structures.

### Normalization

Maps external names and structures into the project's internal representation.

Example:

`paperId`
→ `paper_id`

### Validation

Checks whether the normalized data satisfies the project contract.

These responsibilities may exist near each other in a small V0 implementation, but their conceptual differences should remain clear.

When a failure occurs, identify which responsibility failed.

Do not describe every data problem simply as:

> API error.

---

## 31. Error Messages

Errors should help a human understand what failed.

Prefer messages that expose the failing boundary.

Useful categories include:

- Semantic Scholar request failed
- Semantic Scholar rate limit reached
- Semantic Scholar request timed out
- unexpected API response
- paper validation failed
- no papers found

Avoid exposing unnecessary implementation internals to the CLI user.

Also avoid overly generic messages.

Bad:

> Error.

Better:

> Semantic Scholar request timed out before search results were returned.

Error handling is part of system design.

---

## 32. Do Not Swallow Exceptions Silently

Never use broad exception handling merely to keep the program running.

Avoid patterns conceptually equivalent to:

`except Exception: return []`

when that would erase the difference between:

- API failure
- programming bug
- zero search results

Catch exceptions when the application can:

- explain the problem
- recover intentionally
- convert external-library behavior into a clearer project-level error

Do not suppress information needed for debugging.

---

## 33. Search Latency

V0 evaluation includes Machine Latency.

The search pipeline should allow Semantic Scholar search and file-generation latency to be observed.

Do not build a performance-monitoring system.

A simple measurement is sufficient.

Measure the relevant operation using an appropriate monotonic timer.

Record latency in seconds.

The purpose is not micro-optimization.

The purpose is to establish observable system behavior.

---

## 34. What Latency Means

Machine Latency should answer a simple question:

> How long did the machine-controlled search and artifact-generation stage take?

Do not include human ChatGPT analysis time in machine latency.

Human-in-the-loop work is a separate stage.

Keep measurement boundaries consistent when comparing runs.

Do not compare values measured using different definitions without clearly stating the difference.

---

## 35. Do Not Optimize Latency Prematurely

Do not change architecture merely because one request appears slow.

First determine:

- what was measured
- whether the delay was external or local
- whether the result is reproducible
- whether performance matters for the V0 acceptance criteria

The project prioritizes reliability and learning over micro-optimization.

A slightly slow but understandable pipeline is preferable to unnecessary complexity.

---

## 36. Search Testing Philosophy

Tests should protect the search contract.

They should not prove that Semantic Scholar itself works.

The test suite should focus on our behavior around the external boundary.

Important V0 search cases include:

- successful search response
- up to five papers returned
- field normalization
- author-name extraction
- `abstract = None`
- optional metadata missing
- zero search results
- HTTP 429
- timeout
- malformed or unexpected response
- JSON persistence

Tests should be small and understandable.

---

## 37. Do Not Depend on Live Semantic Scholar in Tests

Automated tests should not require the live Semantic Scholar API.

Live API tests can fail because of:

- network availability
- external outages
- rate limits
- changing search results
- service behavior outside our control

This would make tests nondeterministic.

Use mocks or fixtures for core automated tests.

The live API should be used during manual integration runs.

Keep these two purposes separate.

---

## 38. Mock Responses

Use small mock responses that represent meaningful external behavior.

Useful fixtures may represent:

### Successful Response

Five or fewer realistic paper records.

### Missing Abstract

A valid paper with:

`abstract = null`

### Zero Results

An empty `data` list.

### HTTP 429

A simulated rate-limit response.

### Timeout

A simulated request timeout.

### Malformed Response

A response that violates expected API shape.

Do not create enormous fixtures.

Each fixture should exist because it protects a specific contract.

---

## 39. Mock Data Integrity

Mock data should not create fake scientific claims for presentation as real research.

Mocks are engineering test artifacts.

Make their purpose obvious.

If paper metadata is fictional or modified for a unit test, do not present it as real scientific evidence in sample portfolio outputs.

Keep:

engineering fixture

separate from:

real retrieval result.

This distinction matters in a scientific evidence project.

---

## 40. Live API Verification

After core tests pass, perform a live Semantic Scholar search for manual integration verification.

A live run should answer:

- Did the request reach Semantic Scholar?
- Did we receive up to five papers?
- Did metadata normalize correctly?
- Did `papers.json` get created?
- Are missing fields preserved correctly?
- Is the output understandable?

Do not use one successful live call as proof that error handling works.

Failure behavior belongs in controlled tests.

---

## 41. Test Naming

Name tests after behavior, not implementation details.

Prefer names conceptually similar to:

`test_search_returns_normalized_papers`

`test_search_preserves_missing_abstract`

`test_search_handles_zero_results`

`test_search_handles_rate_limit`

`test_search_handles_timeout`

Avoid tests tied unnecessarily to private helper-function structure.

Refactoring should not break tests when observable behavior remains correct.

---

## 42. Search Test Priority

Prioritize tests in roughly this order:

1. successful response
2. normalized paper fields
3. missing abstract
4. zero results
5. HTTP 429
6. timeout
7. invalid or malformed response
8. JSON output

Do not spend excessive time testing every internal line of code.

The project timebox is 6–8 hours.

Protect the highest-risk boundaries first.

---

## 43. API Blocking Stop Condition

Do not allow Semantic Scholar connectivity problems to consume the entire project.

If live API integration blocks progress for approximately 45 minutes:

1. record the failure
2. preserve the current debugging evidence
3. create or use a small mock Semantic Scholar response
4. continue building the downstream search contract
5. return to live API integration later

This is not giving up.

It is a scope-control decision.

The goal is to keep the V0 learning pipeline moving.

---

## 44. Mock-First Continuation During API Failure

When switching temporarily to mock data:

Do not pretend the live API works.

Clearly mark:

> Live Semantic Scholar integration not verified yet.

Use the mock only to validate:

- parsing
- normalization
- schema interaction
- JSON persistence
- downstream file flow

Do not claim Semantic Scholar integration success until a live request has actually succeeded.

---

## 45. Retrieval Observation

After a successful live search, the user should inspect the retrieved papers.

Do not automatically judge relevance on behalf of the user as part of the search implementation.

Present enough information to inspect:

- title
- year
- authors where useful
- abstract availability
- source URL

The user will later assign relevance judgments.

Search implementation and retrieval evaluation are related but separate responsibilities.

---

## 46. Human Relevance Judgment

V0 uses a human relevance scale:

`0 = not relevant`

`1 = partially relevant`

`2 = directly relevant`

The search pipeline should preserve enough information for this judgment.

Do not automatically generate these scores using an LLM.

Do not invent relevance labels.

Human judgment is intentional in V0.

---

## 47. Precision@5 Preparation

Precision@5 conceptually asks:

> Of the top five retrieved papers, how many are relevant to the research question?

For the V0 project, human relevance judgments are used to support this evaluation.

Do not change the search implementation merely to maximize Precision@5 before establishing the baseline.

The sequence should be:

Initial Search  
→ Observe Top 5  
→ Human Relevance Judgment  
→ Measure  
→ Form Hypothesis  
→ Minimal Search Change  
→ Re-run  
→ Compare

This is an engineering experiment.

---

## 48. Poor Retrieval Is Not Automatic Failure

If only two of five retrieved papers are relevant, do not hide or replace the result.

That may reveal a meaningful limitation:

> Semantic Scholar top-5 relevance search is insufficient for this research question.

This can be a valid V0 finding.

The project values measured limitations.

Do not optimize the portfolio by deleting weak examples.

---

## 49. Search Experiments

When testing a retrieval improvement, change one meaningful variable at a time.

Possible future V0 experiment:

Original:

`self-supervised learning`

Modified:

`self supervised learning`

Compare:

- result set
- relevance judgments
- Precision@5

Avoid simultaneously changing:

- query wording
- result count
- filters
- endpoint
- evaluation question

Otherwise the cause of improvement becomes unclear.

---

## 50. Before / After Retrieval Comparison

Whenever a search change is described as an improvement, record:

### Before

- query
- top-five results
- human relevance judgments
- Precision@5

### Change

One specific retrieval modification.

### After

- resulting query
- top-five results
- human relevance judgments
- Precision@5

### Conclusion

- improved
- worsened
- unchanged
- inconclusive

Never rely solely on:

> These papers look better.

Measurement is required for meaningful retrieval claims.

---

## 51. Search Failure and `$debug-learning`

Use `$debug-learning` when encountering:

- HTTP 429
- timeout
- unexpected zero results
- malformed API data
- failed search tests
- surprising missing fields
- low Precision@5
- unexpected query behavior
- unexplained latency
- search results that appear unrelated

The search skill defines the retrieval workflow.

The debug-learning skill defines how meaningful failures should be investigated.

Do not duplicate a full debugging process inside arbitrary code changes.

---

## 52. Search Pipeline Completion Check

Before declaring the search stage complete, verify:

- a research question can enter the search flow
- the query sent to Semantic Scholar is understandable
- the result limit is five
- required metadata fields are requested
- successful responses are parsed
- external fields are normalized
- author names are converted appropriately
- missing abstracts remain missing
- optional missing metadata remains missing
- zero results are handled
- HTTP 429 is handled
- timeout is handled
- tests do not depend on the live API
- normalized paper data can be persisted
- `papers.json` can be inspected by the user

Do not continue to the evidence-packet stage until the search stage has been run or explicitly approved.

# 3/3 — Codex Execution Rules, Acceptance Criteria, and Completion

## 53. Codex Working Mode

When this skill is invoked, Codex should behave as a stage-focused implementation partner.

Do not attempt to solve the entire repository.

Work only on the search pipeline.

At the beginning of the task, briefly state:

### Current Stage

Search Pipeline

### Goal

What specific retrieval capability is being implemented or inspected?

### Files Likely to Change

List only the files that are likely required for the current task.

Typical files may include:

- `src/neuroai_explorer/search.py`
- `src/neuroai_explorer/cli.py`
- `src/neuroai_explorer/schemas.py`
- `tests/test_search.py`
- `tests/test_schemas.py`
- `docs/decisions.md`

Do not list unrelated future-stage files.

### Acceptance Criteria

State the observable conditions that must be true before the current task is considered complete.

Keep this opening concise.

---

## 54. Implementation Order

For the initial V0 search pipeline, prefer this order:

1. Define or inspect the Paper data contract.
2. Implement the smallest Semantic Scholar request function.
3. Parse the external response.
4. Normalize external metadata.
5. Validate normalized paper records.
6. Handle missing optional fields.
7. Handle zero results.
8. Handle timeout.
9. Handle HTTP 429.
10. Save results to `papers.json`.
11. Add focused tests.
12. Run one real NeuroAI research question.
13. Let the user inspect the retrieved papers.
14. Record meaningful failure or retrieval findings.

Do not implement all steps blindly in one uncontrolled change.

When a meaningful stage is completed, provide a command for the user to run and stop when user observation is required.

---

## 55. Small-Step Development

Prefer small implementation steps.

A good task may be:

> Implement the Semantic Scholar request and return raw search data.

A second task may be:

> Normalize the returned paper records into the Paper schema.

A third task may be:

> Add timeout and HTTP 429 handling.

Avoid a task such as:

> Build the full research search system, evaluation pipeline, report generator, and README.

The user must be able to understand what changed at each step.

---

## 56. File Responsibility

Keep responsibilities simple.

### `search.py`

Owns behavior related to:

- Semantic Scholar request construction
- HTTP request execution
- response-status handling
- parsing
- normalization
- search-related exceptions or result handling

Do not place report-building logic here.

### `schemas.py`

Owns Pydantic data contracts.

The search pipeline may use the Paper schema but should not redesign unrelated later-stage schemas unless required.

### `cli.py`

Owns command-line interaction.

Keep CLI logic thin.

The CLI should call application functions rather than contain all search implementation details.

### `tests/test_search.py`

Owns search behavior tests.

Prefer mocked external responses.

### `tests/test_schemas.py`

Owns Paper schema boundary tests when appropriate.

Do not duplicate identical validation tests across multiple files without need.

---

## 57. Thin CLI Rule

The CLI is an entry point, not the search architecture.

Avoid putting all HTTP, parsing, validation, and persistence logic directly inside a CLI command.

Conceptually prefer:

CLI  
→ Search Function  
→ Normalize / Validate  
→ Persist

This allows search behavior to be tested independently from command-line input handling.

Do not overengineer the CLI with a framework unless already justified by the repository.

---

## 58. Function Design

Prefer functions with clear responsibilities.

Examples of reasonable responsibilities include:

- perform Semantic Scholar search
- normalize one external paper record
- save validated papers
- generate a safe question slug

Avoid unnecessary class hierarchies.

Do not create interfaces, repositories, service containers, or factories solely because they are common enterprise patterns.

V0 should remain easy to read from top to bottom.

---

## 59. Dependency Rule

Use the smallest dependency set required by the current repository design.

Before adding a package, ask:

1. Is this required for the current V0 stage?
2. Can the existing standard library or installed project dependency handle it clearly?
3. Does the new package meaningfully simplify reliability or readability?
4. Does it increase project setup cost?

Do not add dependencies for speculative future features.

If a new dependency is genuinely needed, explain:

### Dependency

Package name.

### Why

What concrete V0 problem it solves.

### Alternative

What would be required without it.

Do not install first and justify later.

---

## 60. Do Not Refactor Ahead

Do not refactor working search code merely because a more sophisticated architecture is possible.

Do not introduce:

- service layers
- repositories
- dependency injection
- plugin systems
- generic API clients
- abstract base classes

unless a concrete V0 problem requires them.

Prefer explicit code over premature extensibility.

The project currently has one paper source.

Do not design for ten hypothetical sources before a second source exists.

---

## 61. Schema Boundary Rule

External Semantic Scholar data should cross a clear validation boundary before it becomes trusted internal data.

Conceptually:

Semantic Scholar JSON  
→ Normalization  
→ Paper Schema  
→ Internal Paper Data

Do not treat arbitrary external dictionaries as trusted objects throughout the application.

The schema exists to make assumptions explicit.

When validation fails, investigate the contract rather than bypassing it.

---

## 62. Persistence Rule

`papers.json` should represent the actual retrieval result for the research question.

Do not write the file before data has passed the intended normalization and validation boundary.

The file should remain useful for:

- manual inspection
- later evidence packet generation
- reproducibility
- debugging
- evaluation

Avoid embedding temporary debug logs inside the JSON artifact.

Logs and data are different concerns.

---

## 63. Output Directory Rule

Store research-question-specific results under a predictable path.

Target pattern:

`data/<question-slug>/papers.json`

If the parent directory does not exist, create it intentionally.

Do not scatter generated files across:

- repository root
- source package
- test directories

Generated research data belongs under the designated data directory.

Follow existing `.gitignore` policy.

---

## 64. Sample Data vs Runtime Data

Keep real runtime data and public sample artifacts conceptually separate.

Runtime search output:

`data/...`

Public portfolio sample:

`examples/...`

Do not automatically copy every research run into the examples directory.

Only curated and verified examples should become public sample artifacts.

Do not expose test fixtures as if they were real Semantic Scholar results.

---

## 65. Logging and Console Output

Console output should help the user observe the current stage.

Useful information may include:

- research question
- number of papers retrieved
- output file path
- search latency
- readable failure message

Avoid excessive internal debug noise during normal successful execution.

Do not print full large API responses unless explicitly debugging.

The user should be able to understand:

> What was searched, what happened, and where the result was saved.

---

## 66. Do Not Hide the Query

The user should be able to determine what query was actually sent to Semantic Scholar.

This becomes important when debugging retrieval quality.

If the input question is transformed, preserve enough visibility to compare:

### Original Question

User-supplied research question.

### Search Query

Actual string sent to Semantic Scholar.

Do not perform hidden query rewriting.

Retrieval experiments require reproducible inputs.

---

## 67. Search Reproducibility

External search ranking may change over time.

Do not claim that a live Semantic Scholar query will always return the same five papers.

Instead, improve reproducibility by preserving:

- research question
- actual query
- retrieval timestamp if already part of evaluation design
- retrieved paper IDs
- saved `papers.json`

Do not overengineer versioning during V0.

The saved artifact is the primary record of what the system observed at that time.

---

## 68. Live Result Honesty

Never claim a live Semantic Scholar search succeeded unless it was actually executed successfully.

If only mocks were tested, say:

> Search behavior verified with mocks; live Semantic Scholar integration not verified yet.

If the live API works but a specific failure branch was only mocked, say so.

Distinguish:

- unit-tested behavior
- integration-tested behavior
- manually observed behavior

Do not collapse all three into:

> Everything works.

---

## 69. Testing Before Claiming Completion

Before declaring a code change complete:

1. run the relevant focused tests
2. inspect the result
3. report what passed
4. report anything not verified

Prefer focused tests first.

Example:

`pytest tests/test_search.py -q`

Then, when appropriate, run the wider suite.

Do not repeatedly run the entire test suite after every tiny edit unless needed.

---

## 70. Search Stage Manual Run

Once core search behavior is ready, provide an exact manual command using a representative NeuroAI question.

Recommended representative question:

`How does hippocampal replay differ from experience replay in reinforcement learning?`

Example conceptual command:

`python -m neuroai_explorer search "How does hippocampal replay differ from experience replay in reinforcement learning?"`

Adapt the exact command to the actual CLI implementation.

Do not document commands that do not match the repository.

---

## 71. What the User Should Observe

After the manual search, ask the user to inspect:

1. Did the command complete?
2. How many papers were retrieved?
3. Was `papers.json` created?
4. Do the paper IDs and titles look valid?
5. How many papers contain abstracts?
6. Do the papers appear relevant to the research question?
7. Are any fields unexpectedly missing?
8. Was the latency reasonable and recorded if required?

Do not immediately continue to evidence-packet generation.

The user's observation is part of the stage gate.

---

## 72. Stage Gate

After the first successful search run:

STOP.

Report:

### Changed

What implementation was completed?

### Files

Which files changed?

### Run

Exact command the user should execute.

### Expected

What should happen?

### Observe

What should the user inspect?

Then wait for the user's result before moving to the next major stage unless the user explicitly approves continuation.

Do not build ahead.

---

## 73. Failure Stage Gate

If the manual run fails:

Do not continue to the next pipeline stage.

Invoke or recommend:

`$debug-learning`

Then investigate:

Failure  
→ Contract  
→ Hypothesis  
→ Concept  
→ Minimal Change  
→ Same Input Re-run

Search failure must be understood before downstream functionality is built on top of it.

---

## 74. First Retrieval Evaluation

After the first successful live search, perform or prepare a simple manual relevance observation.

For each retrieved paper, the user may assign:

`0 = not relevant`

`1 = partially relevant`

`2 = directly relevant`

Do not automate this judgment in V0.

The purpose is to expose the relationship between:

Semantic Ranking

and:

Human Research Relevance

These are not necessarily the same.

---

## 75. Precision@5 Interpretation

When Precision@5 is calculated, report the actual result honestly.

If the relevance definition treats both partially and directly relevant papers as relevant, document that rule explicitly.

Do not quietly change the relevance threshold between experiments.

The evaluation rule must remain stable for before/after comparison.

If the exact Precision@5 labeling convention has not yet been defined in the project evaluation rubric, do not invent one silently.

Flag it for explicit definition.

---

## 76. Retrieval Decision Log

Create a Decision Log entry when a search observation changes engineering understanding.

Examples:

- hyphenated terminology reduced retrieval quality
- Semantic Scholar top-5 search returned weak cross-disciplinary matches
- missing abstracts materially reduced downstream evidence coverage
- rate limiting required explicit retry behavior
- query normalization improved Precision@5
- live API reliability forced temporary mock-based development

Do not write a Decision Log entry simply because a normal search succeeded.

Focus on decisions and learning.

---

## 77. Scope Escalation Rule

If retrieval quality appears poor, do not immediately suggest RAG, embeddings, or reranking as the implementation answer.

First ask:

> What is the smallest search-level experiment that could explain the failure?

Possible V0 experiments may include:

- query wording
- hyphen normalization
- terminology simplification

Only after the V0 evaluation should larger retrieval architecture be discussed as future work.

Preserve the roadmap:

V0  
→ simple Semantic Scholar retrieval

Future versions  
→ richer retrieval architecture

Do not collapse future roadmap into current implementation.

---

## 78. Scientific Search Integrity

Search code must not manipulate results to support a preferred scientific conclusion.

Do not:

- remove papers because their findings seem inconvenient
- prioritize papers based on desired conclusions
- fabricate relevance
- insert papers from memory
- replace API results with hand-selected evidence without clearly labeling the change

The retrieval system should expose what it actually retrieved.

Scientific interpretation belongs later.

---

## 79. Citation Count

`citation_count` is metadata.

Do not treat citation count as:

- scientific truth
- methodological quality
- evidence strength
- relevance score

Do not sort or filter by citation count unless an explicit experiment requires it.

High citation count does not automatically mean a paper is more relevant to the research question.

Preserve the field without overinterpreting it.

---

## 80. Publication Year

`year` is metadata.

Do not automatically favor newer or older papers unless the research question or explicit evaluation requires it.

Missing year should remain missing when allowed by the schema.

Do not invent temporal filtering during initial V0 retrieval.

---

## 81. Search URL and Provenance

When Semantic Scholar provides a usable paper URL, preserve it.

The later evidence system depends on source traceability.

Do not remove URLs for cosmetic reasons.

If a URL is absent and the schema allows it:

preserve:

`null`

Do not fabricate a URL based on assumptions about paper titles or identifiers.

---

## 82. Security and Input Handling

Treat user-supplied research questions as data.

Do not execute research-question text as shell code.

When creating CLI or filesystem behavior:

- pass query text as data
- generate filesystem-safe slugs
- avoid unsafe string interpolation into shell commands
- rely on HTTP client parameter handling for query encoding

Do not build manual URL strings in a way that creates unnecessary escaping bugs when the HTTP client can safely manage query parameters.

---

## 83. Secrets

The V0 should begin without requiring a Semantic Scholar API key if the chosen public endpoint supports the intended usage.

Do not hardcode secrets.

If credentials are introduced later:

- use environment variables or the repository's established secret-handling method
- never commit secret values
- never place keys in examples or tests

Do not add secret-management infrastructure unnecessarily during V0.

---

## 84. Code Comments

Use comments to explain decisions that are not obvious from code.

Good comment:

> Preserve missing abstracts because incomplete external metadata is a valid V0 state.

Weak comment:

> Call function.

Do not over-comment straightforward Python.

Prefer readable names and small functions.

Comments should explain why, not narrate every line.

---

## 85. Type Clarity

Use type annotations where they improve boundary clarity.

The most important typed boundaries are:

- research query input
- external response handling
- normalized Paper data
- lists of validated Paper objects
- persistence interfaces

Do not spend excessive time making every local variable maximally typed.

Prioritize contracts over ceremony.

---

## 86. Error Type Clarity

When introducing project-specific errors, keep the number small.

Only create custom exceptions when they make application behavior clearer.

Possible conceptual distinctions include:

- search request failure
- rate-limit failure
- response parsing failure

Do not create an elaborate exception hierarchy for V0.

Readable messages and clear boundaries matter more than class count.

---

## 87. Search Pipeline Acceptance Criteria

The initial search pipeline is accepted when all of the following are true:

### Input

- a research question can be supplied to the search flow

### Request

- the Semantic Scholar paper search endpoint is used
- the query is plain-text and observable
- the result limit is five
- only necessary metadata fields are requested

### Response

- successful responses are parsed
- up to five papers are handled
- zero results are handled intentionally
- HTTP 429 is handled intentionally
- timeout is handled intentionally

### Data

- external metadata is normalized
- `paperId` becomes `paper_id`
- `citationCount` becomes `citation_count`
- author names become `list[str]`
- missing abstracts remain missing
- optional missing values are preserved
- `source` is preserved as `semantic_scholar`

### Validation

- normalized records cross the Paper schema boundary
- invalid required fields are not fabricated

### Persistence

- output is saved to:
  `data/<question-slug>/papers.json`

### Testing

- core search behavior has focused tests
- automated tests do not require the live Semantic Scholar service

### Observation

- at least one representative NeuroAI question has been manually searched or clearly marked as not yet live-verified
- the retrieved papers can be inspected by the user

Do not declare the search stage complete unless these conditions are satisfied or the user explicitly changes the scope.

---

## 88. Definition of Search Stage Done

The search stage is done when the user can explain the following flow:

Research Question  
→ Query Parameter  
→ HTTP Request  
→ Status Code  
→ External JSON  
→ Normalization  
→ Paper Validation  
→ `papers.json`

The user should also be able to explain:

- why `abstract=None` is valid external-data behavior
- why HTTP 429 differs from zero results
- why timeout must be bounded
- why tests should mock external API behavior
- why top-five search results are candidates rather than scientific truth
- why retrieval improvement must be measured

If the code works but these boundaries remain hidden, the learning objective is incomplete.

---

## 89. Completion Response Format

When finishing a search-pipeline task, respond using:

### Changed

Briefly explain what was implemented.

### Files Changed

List only files actually changed.

### Verified

State which tests or executions were actually run.

### Run

Provide the exact command the user should run next.

### Expected

State the expected observable output.

### Observe

Tell the user what to inspect or think about.

### Not Yet Done

Clearly identify anything that belongs to later stages.

Example:

### Not Yet Done

- evidence packet generation
- ChatGPT analysis
- final brief
- full evaluation

Do not imply that later stages are complete.

---

## 90. Final Search Pipeline Rule

The purpose of this skill is not to make Semantic Scholar look reliable.

The purpose is to build a small system that behaves honestly around an unreliable external source.

The desired engineering pattern is:

Research Question  
→ Retrieve  
→ Validate  
→ Preserve Missing Data  
→ Handle Failure  
→ Save  
→ Observe  
→ Measure

Do not build ahead.

Do not hide external uncertainty.

Do not confuse retrieval with evidence.

Do not claim improvement without comparison.

Keep the search pipeline small enough that the user can understand every system boundary.