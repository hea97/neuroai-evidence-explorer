---
name: debug-learning
description: Investigate meaningful failures using observation, user hypothesis, concept learning, minimal fixes, and before/after measurement. Use for errors, failed tests, unexpected outputs, API failures, schema errors, retrieval problems, timeout, HTTP 429, null data, or suspicious scientific outputs.
---

# Debug Learning

## Purpose

Use this skill when something in NeuroAI Evidence Explorer fails, behaves unexpectedly, or produces suspicious results.

This skill must NOT behave like an automatic bug fixer.

Its purpose is to turn failures into AI Engineering learning opportunities.

The required loop is:

Failure
→ Observation
→ Failing Stage
→ Input / Output Contract
→ User Hypothesis
→ Engineering Concept
→ Success Criterion
→ Minimal Fix
→ Same Input Re-run
→ Before / After Comparison
→ Decision Log

The user should understand why the failure occurred, not merely receive working code.


---

# 1. When to Use This Skill

Use this workflow for meaningful failures such as:

- Python exceptions
- failed pytest tests
- unexpected CLI output
- Semantic Scholar API failures
- HTTP 429
- timeout
- zero search results
- malformed API responses
- missing fields
- `abstract = None`
- Pydantic validation errors
- invalid `analysis.json`
- unknown `paper_id`
- evidence packet problems
- report generation failures
- low Precision@5
- unsupported scientific claims
- citation traceability failures
- results that differ from expectations

Also use this skill when the code technically runs but the result appears wrong.

Examples:

```text
The API returned five papers, but only one appears relevant.
The report was generated, but a method appears that is not stated in the abstract.
analysis.json passes JSON parsing but fails Pydantic validation.
```

# 2. Do Not Immediately Fix Meaningful Failures

When encountering a new meaningful failure, STOP before applying a non-trivial fix.

Do not silently:

add retries
change schemas
change query logic
rewrite functions
add fallback behavior
suppress exceptions
modify prompts
change evaluation rules
introduce dependencies

First investigate the failure.

The only exception is an obviously trivial mechanical problem when the user explicitly requests a direct fix.

Examples of trivial issues may include:

typo in a filename
missing import already implied by existing code
formatting-only mistake
obvious syntax error introduced in the immediately previous edit

When uncertain, use the learning workflow.

# 3. Step 1 — Define the Failure

Start by describing the failure in one sentence.

Use this format:

Observed:
<what actually happened>

Then identify the pipeline stage.

Valid stages include:

CLI Input
Semantic Scholar Request
API Response Parsing
Paper Validation
Paper Storage
Evidence Packet
Human GPT Analysis
Analysis Validation
Paper Reference Validation
Report Generation
Evaluation
Testing

Example:

Observed:
Semantic Scholar returned HTTP 429 instead of the expected paper list.

Failing Stage:
Semantic Scholar Request
# 4. Step 2 — State the Input / Output Contract

Before discussing the cause, define what the stage was supposed to receive and produce.

Use:

Input:
...

Expected Output:
...

Actual Output:
...

Example:

Input:
Research question string

Expected Output:
HTTP 200 response containing up to five paper records

Actual Output:
HTTP 429 response

The purpose is to separate:

what we expected

from:

what actually happened

Do not begin debugging from assumptions.

# 5. Step 3 — Ask the User for a Hypothesis

For a new meaningful failure, ask the user to propose 1–2 possible causes.

Do not reveal the complete diagnosis first.

Use a short question such as:

Before we change the code, what do you think could cause this?
Give me one or two hypotheses.

If useful, provide categories without giving away the answer.

Example:

Think about whether the problem is more likely related to:

- our request
- the external API
- our parsing logic

Then WAIT.

Do not continue into implementation until the user has answered, unless the user explicitly asks to skip this learning step.

# 6. Step 4 — Evaluate the Hypothesis

After the user proposes a hypothesis:

acknowledge what part is plausible
distinguish observation from assumption
identify what evidence would confirm or reject the hypothesis
avoid immediately declaring an answer when evidence is available to test

Use the structure:

Hypothesis:
...

Why it is plausible:
...

What would confirm it:
...

What would reject it:
...

If several hypotheses exist, prefer testing the cheapest and most informative one first.

# 7. Step 5 — Explain the Engineering Concept

Before changing code, explain the smallest relevant concept needed to understand the failure.

Keep the explanation focused.

Possible concepts include:

HTTP status codes
rate limiting
timeouts
retry
backoff
API contracts
optional fields
null handling
Pydantic validation
JSON contracts
schema boundaries
exception handling
mocking
retrieval relevance
Precision@5
grounding
provenance
unsupported claims
citation traceability

Explain:

What is it?
Why does it exist?
Why does it matter here?

Prefer simple examples from the current failure.

Do not turn the response into a broad tutorial unless the user asks for one.

# 8. Step 6 — Define the Success Criterion

Before making the change, define how we will know whether it worked.

Avoid vague criteria such as:

It should work better.

Prefer measurable or observable criteria.

Examples:

A timeout should produce a readable application error instead of an uncaught exception.
abstract=None should pass Paper validation and remain null in papers.json.
The same query should return at least one additional directly relevant paper.
The unsupported field should become not_reported instead of a fabricated value.

Write:

Success Criterion:
...
# 9. Step 7 — Apply the Smallest Fix

Use:

One failure → One hypothesis → One minimal change

Change only what is required to test the current hypothesis.

Do not:

perform unrelated refactoring
rename unrelated modules
reorganize directories
introduce abstractions without need
install new packages unless necessary
solve adjacent problems at the same time

Prefer local, reversible changes.

Before editing, briefly state:

Minimal Change:
...

Example:

Minimal Change:
Add explicit handling for HTTP 429 in the existing Semantic Scholar request function without changing the search architecture.
# 10. Step 8 — Re-run the Same Input

Whenever possible, reproduce the original failure using the same input.

The comparison must isolate the effect of the change.

Prefer:

same research question
same fixture
same JSON
same test
same CLI command

Do not change the input and the implementation simultaneously unless unavoidable.

Provide the exact command to run.

Example:

pytest tests/test_search.py -q

or:

python -m neuroai_explorer search \
  "How does hippocampal replay differ from experience replay in reinforcement learning?"
# 11. Step 9 — Compare Before and After

After re-running, compare the observed result.

Use:

Before:
...

After:
...

Success Criterion:
PASS / FAIL

Do not claim improvement merely because the error disappeared.

Check whether the intended behavior actually improved.

Example:

Before:
HTTP 429 produced an uncaught exception.

After:
HTTP 429 produces a clear rate-limit error message.

Success Criterion:
PASS

If it fails:

Do not pile on additional fixes.

Start another hypothesis cycle.

# 12. Step 10 — Decide What the Result Means

After the experiment, classify the result.

Possible conclusions:

Hypothesis supported
Hypothesis rejected
Partially supported
Insufficient evidence

Then explain what was learned.

Example:

Result:
Hypothesis supported.

Learned:
An external API should be treated as an unreliable system boundary.
HTTP 429 is not a parsing failure; it is a service-level response that requires explicit handling.
# 13. Decision Log

Not every failure belongs in the Decision Log.

Record the result in:

docs/decisions.md

when the failure teaches something meaningful about:

architecture
external APIs
schemas
retrieval
grounding
evaluation
project scope
reliability
dependency decisions
scientific integrity

Use:
```Text
## Decision XXX — <title>

## Problem

...

### Observation

...

### Hypothesis

...

### Change

...

### Before

...

### After

...

### Decision

...

### Learned

...
```

Do not log:

simple typos
formatting fixes
obvious syntax mistakes
insignificant test maintenance
# 14. Special Debugging Cases
HTTP 429

Do not assume the API is broken.

Investigate:
```
request frequency
rate limiting
response status
retry behavior
error handling
```
Relevant concepts:
```
rate limit
retry
backoff
external service reliability
Timeout
```
Distinguish:
```
network failure
slow external service
incorrect timeout configuration
application bug
```
Do not simply increase timeout indefinitely.

abstract = None

This is not automatically an error.

External scientific metadata is incomplete.

Expected principle:

None is valid when the schema allows it.

Do not invent an abstract or remove the paper silently.

Zero Search Results

Do not immediately broaden the query automatically.

First inspect:
```
research question
query string
normalization
API response
```
A zero-result search may be valid information.

Low Precision@5

Do not manipulate the evaluation.

Investigate:
```
query formulation
ambiguous terminology
cross-disciplinary terminology
Semantic Scholar ranking behavior
```
Record the actual score.

Pydantic Validation Error

Do not bypass validation simply to make the pipeline run.

Validation failure indicates a broken contract.

Identify:
```
which field
expected type/value
actual type/value
source of the invalid data
Unsupported Scientific Claim
```
Treat this as a reliability failure.

Do not rewrite the claim using general scientific knowledge.

Check whether the supplied abstract actually supports it.

If not supported:

not_reported

or move the statement into:

Interpretation

or:
```
Hypothesis
```
when appropriate.

# 15. Scientific Integrity During Debugging

Never use outside scientific knowledge to silently repair missing evidence.

The pipeline must remain grounded in the supplied evidence.

Keep these categories distinct:

Evidence
Interpretation
Uncertainty
Hypothesis

If a claim cannot be traced to the supplied paper information, say so.

A correct-looking scientific statement can still be a pipeline failure if it is unsupported by the evidence packet.

# 16. Debugging Output Format

When starting a debugging cycle, keep the response concise and structured.

Use:
```
Observed
Failing Stage
Input
Expected Output
Actual Output
Your Hypothesis?
```
After receiving the user's hypothesis, continue with:
```
Hypothesis Assessment
Concept
Success Criterion
Minimal Change
Run
```
After execution:
```
Before
After
Result
Learned
Decision Log
```
Do not flood the user with every possible explanation at once.

# 17. Do Not Hide Failure

Failures are project evidence.

Do not:
```
suppress warnings solely for appearance
delete low evaluation scores
remove failed examples because they look bad
change metrics after seeing results
describe partial success as complete success
fabricate successful execution
```
If a test or experiment has not actually been run, say:

Not verified yet.

If the result is poor, report the result accurately.

# 18. Core Principle

The purpose of debugging in this repository is not:
```
make the error disappear
```
The purpose is:
```
observe
→ understand
→ hypothesize
→ test
→ measure
→ learn
```
A failure that produces a clear engineering lesson is valuable project output.


## 이 Skill이 실제로 어떻게 작동하나

예를 들어 Day 1에서 Semantic Scholar 호출 중 `429`가 발생했다고 하자.

그때 Codex에:

```text
$debug-learning

Semantic Scholar search를 실행했는데 429가 발생했어.

라고 하면 바로 retry 코드를 집어넣는 게 아니라,

Observed:
Semantic Scholar returned HTTP 429.

Failing Stage:
Semantic Scholar Request

Input:
Research question

Expected Output:
HTTP 200 + up to five papers

Actual Output:
HTTP 429

Before changing the code, what do you think could cause this?
Give me 1–2 hypotheses.
```