# NeuroAI Evidence Explorer — Codex Instructions

## 1. Project Mission

NeuroAI Evidence Explorer V0 is a small AI Engineering learning project.

The goal is NOT to build the largest or most automated research system.

The goal is to learn how to build a trustworthy evidence pipeline by working through:

Research Question
→ Semantic Scholar Search
→ Paper Schema + Validation
→ Evidence Packet
→ Human-in-the-loop ChatGPT Analysis
→ Structured Analysis Validation
→ Evidence Brief
→ Evaluation
→ Decision Log

Optimize for:

1. Learning
2. Reliability
3. Traceability
4. Measurable improvement
5. Small and understandable implementation

Prefer:

Learning > Feature Count  
Reliability > Convenience  
Traceability > Fluent Output  
Evaluation > Impression  
Simple Design > Premature Architecture


---

# 2. V0 Scope

The V0 timebox is approximately 6–8 hours across two days.

The V0 must remain intentionally small.

## Required V0 Capabilities

The finished V0 should be able to:

- accept one NeuroAI research question from the CLI
- search Semantic Scholar for up to 5 relevant papers
- store paper metadata in a consistent schema
- handle missing abstracts
- handle zero results
- handle timeout and HTTP 429 errors
- generate `papers.json`
- generate `evidence_packet.md`
- accept manually-created ChatGPT analysis as `analysis.json`
- validate the analysis with Pydantic
- verify that referenced `paper_id` values exist
- generate a Markdown evidence brief
- separate Evidence, Interpretation, and Hypothesis
- evaluate representative research questions
- record important failures and engineering decisions
- keep README and sample outputs understandable for a public GitHub repository


---

# 3. Strict Scope Guard

Do NOT add the following during V0:

- LLM API calls
- automated ChatGPT calls
- RAG
- embeddings
- vector databases
- rerankers
- PDF full-text parsing
- databases
- SQLite
- PostgreSQL
- FastAPI
- Streamlit
- React
- web UI
- authentication
- multi-agent systems
- fine-tuning
- Docker
- CI/CD
- cloud deployment
- unnecessary infrastructure
- unnecessary frameworks

Do not introduce a new dependency, infrastructure component, or architectural layer unless it is required by the current V0 acceptance criteria.

Before adding anything outside the current design, ask:

> Is this required to satisfy the V0 Definition of Done?

If the answer is no, do not add it.

Do not implement V1 features early.


---

# 4. Learning-First Development Rule

This repository is also a learning environment.

When a meaningful failure occurs, do not immediately hide or automatically repair it.

Use this learning loop:

Failure
→ Identify failing stage
→ Define expected input/output
→ User forms a hypothesis
→ Explain relevant engineering concept
→ Define success condition
→ Apply smallest fix
→ Run the same input again
→ Compare before/after
→ Record important learning

For a new meaningful failure:

1. Show the observed failure clearly.
2. Identify which pipeline stage failed.
3. State the expected input and output contract.
4. Ask the user to propose 1–2 possible causes.
5. Wait for the hypothesis before applying a non-trivial fix.
6. Explain the relevant concept briefly.
7. Propose the smallest reasonable change.
8. Define how success will be measured.
9. Apply the change.
10. Re-run using the same input whenever possible.
11. Compare the result before and after.
12. Record meaningful findings in the Decision Log.

Do not optimize away useful learning moments.

Exception:

If the user explicitly asks Codex to directly fix a trivial issue, the fix may proceed without the full learning loop.


---

# 5. Minimal Change Rule

Use:

> One failure → One hypothesis → One minimal change

Do not perform unrelated refactoring while testing a hypothesis.

Do not modify multiple architectural layers when one local change can test the idea.

Prefer changes that are:

- small
- reversible
- easy to understand
- easy to test
- directly connected to the observed problem

Avoid speculative abstractions.

Do not rewrite working code merely because another design looks cleaner.

Refactor only when:

- duplication causes a concrete problem
- tests are difficult because responsibilities are unclear
- the current structure blocks the next required V0 capability
- the user explicitly requests refactoring


---

# 6. Stage-Gated Development

Do not build the entire project in one pass.

Major development stages are:

Stage 1
Semantic Scholar search client

Stage 2
Paper schema + validation + error handling

Stage 3
Evidence packet generation

Stage 4
Human-in-the-loop analysis input

Stage 5
Analysis validation + report builder

Stage 6
Evaluation

Stage 7
README + sample outputs + cleanup

After completing a major stage:

- summarize what changed
- list the important files changed
- provide the exact command the user should run
- state the expected result
- mention what should be observed
- stop before implementing the next major stage

Do not continue to the next major stage until the current stage has been executed or explicitly approved by the user.

Never respond to a scoped task by silently implementing later stages.


---

# 7. Repository Design

Target repository structure:

```text
neuroai-evidence-explorer/
├─ src/
│  └─ neuroai_explorer/
│     ├─ cli.py
│     ├─ search.py
│     ├─ schemas.py
│     ├─ packet.py
│     ├─ report.py
│     └─ evaluate.py
│
├─ tests/
│  ├─ test_search.py
│  ├─ test_schemas.py
│  └─ test_report.py
│
├─ data/
│
├─ examples/
│  ├─ sample_packet.md
│  └─ sample_brief.md
│
├─ evaluation/
│  ├─ rubric.md
│  └─ results.md
│
├─ docs/
│  └─ decisions.md
│
├─ AGENTS.md
├─ README.md
└─ pyproject.toml
Keep modules focused.

Avoid creating additional modules unless there is a clear V0 responsibility that cannot reasonably live in the existing structure.

8. Data Contract Principles

External data is untrusted and may be incomplete.

Do not assume API fields always exist.

Validate boundaries explicitly.

Paper Schema

Expected conceptual contract:

Paper
- paper_id: str
- title: str
- abstract: str | None
- authors: list[str]
- year: int | None
- url: str | None
- citation_count: int | None
- source: Literal["semantic_scholar"]

Missing optional information must remain missing.

Do not invent replacements.

Analysis Schema

Expected conceptual contract:

PaperAnalysis
- paper_id: str
- research_question: str | "not_reported"
- method: str | "not_reported"
- dataset: str | "not_reported"
- main_finding: str | "not_reported"
- limitation: str | "not_reported"
- evidence_quote_or_paraphrase: str
- confidence: Literal["high", "medium", "low"]

If information is not supported by the supplied abstract or evidence packet:

not_reported

must be preferred over inference.

9. Scientific Integrity Rule

Never fabricate scientific information.

Never fill missing scientific fields using general knowledge.

Never convert an inference into a reported fact.

Preserve provenance whenever possible.

Important identifiers and sources include:

paper_id
paper title
Semantic Scholar URL
source
abstract

The final system must distinguish:

Scientific Evidence
Interpretation
Open Uncertainty
Testable Hypothesis

These categories must not be silently merged.

A fluent answer is not automatically a trustworthy answer.

Traceability is more important than sounding complete.

10. Semantic Scholar Rules

V0 uses Semantic Scholar Academic Graph relevance-ranked paper search.

Expected search behavior:

GET /graph/v1/paper/search

with a plain-text research query.

Target result count:

limit = 5

Request only fields needed by V0.

Handle explicitly:

successful responses
zero results
missing abstract
timeout
HTTP 429
malformed or unexpected response data

External services must be treated as unreliable.

Do not assume every API call succeeds.

If API integration blocks progress for approximately 45 minutes, use a small mock JSON fixture to continue testing downstream stages.

Return to the live API after the downstream pipeline works.

11. Testing Philosophy

Tests protect the V0 contract.

Do not optimize for the number of tests or coverage percentage.

Prioritize tests around important system boundaries.

Important cases include:

successful Semantic Scholar response
abstract = None
zero search results
timeout
HTTP 429
invalid Paper data
invalid analysis schema
unknown paper_id
valid report generation

Tests should be:

deterministic where possible
small
understandable
focused on behavior

Mock network behavior in tests when appropriate.

Do not make the test suite depend on live Semantic Scholar availability.

12. Evaluation Rule

Do not describe a system change as an improvement without measurement.

Representative V0 metrics include:

Precision@5
Abstract Coverage
Schema Validity
Unsupported Field Rate
Citation Traceability
Machine Latency

Evaluation should compare:

Baseline
Research question only

against:

V0
Retrieved abstracts
+ provenance
+ structured analysis contract

The purpose of evaluation is not to obtain high scores.

The purpose is to discover what the current system actually does.

A poor score is a valid engineering result if:

it is measured honestly
the cause is investigated
the limitation is documented

Never manipulate evaluation criteria to make the project look better.

13. Decision Log Rule

Use:

docs/decisions.md

for meaningful engineering decisions.

Do not log every small edit.

Record decisions involving:

important failures
hypotheses
architecture
scope
dependencies
evaluation findings
meaningful before/after changes

Recommended format:

## Decision XXX — Short title

### Problem

What happened?

### Observation

What did we actually observe?

### Hypothesis

What did we think caused it?

### Change

What minimal change did we make?

### Before

What happened before the change?

### After

What happened after the change?

### Decision

What did we decide?

### Learned

What engineering concept or lesson did this reveal?
14. README Rule

README must describe the project that actually exists.

Do not document planned functionality as if it is already implemented.

Keep README synchronized with verified behavior.

The final README should allow a new reader to understand within a few minutes:

Problem
Architecture
Why this design
How to run it
Example output
Evaluation
Known limitations
V1 direction

A useful top-level architecture is:

Search
→ Packet
→ Human GPT
→ Validation
→ Brief
→ Evaluation

Do not prioritize README polish over unfinished evaluation or broken functionality.

15. Stop Conditions

Stop feature development when any of the following applies:

the requested stage is complete
the stage requires user observation before proceeding
implementation is drifting outside V0
a new architecture decision is required
a meaningful failure should be investigated first
evaluation has not been performed for an alleged improvement

Specific V0 boundaries:

do not spend excessive time polishing CLI UX
do not endlessly optimize prompts
do not start UI work
do not start RAG work
do not add infrastructure before evaluation
do not add new features while README and evaluation remain incomplete near the end of V0
16. Definition of Done

V0 is complete only when the following behavior has been demonstrated.

Input
A research question can be supplied through the CLI.

Search
Semantic Scholar returns up to 5 papers and the system handles zero results, timeout, and HTTP 429.

Schema
Retrieved papers are validated using the Paper schema.

Packet
An evidence packet contains paper ID, title, abstract, and source URL.

Human AI
ChatGPT analysis follows the defined JSON contract and unsupported information remains not_reported.

Validation
analysis.json is schema-valid and referenced paper_id values correspond to retrieved papers.

Brief
A Markdown brief separates Evidence, Interpretation, Uncertainty, and Hypothesis.

Evaluation
Representative research questions are used to compare Baseline and V0.

Testing
Core search, schema, and report behavior is tested.

Portfolio
README communicates the problem, architecture, execution, results, limitations, and design decisions clearly.

17. Codex Communication Style

When starting a task, briefly state:

Current Stage
Goal
Files likely to change
Acceptance Criteria

Do not provide a long design document before small implementation tasks.

After implementation, report:

Changed
Why
Run
Expected
Observe

Always provide exact commands where practical.

Example:

pytest tests/test_search.py -q

or:

python -m neuroai_explorer search \
  "How does hippocampal replay differ from experience replay in reinforcement learning?"

Do not claim success solely because code was written.

Success requires an executable or testable result.

18. Core Working Principles

Always follow these three rules.

Do not build ahead.

Do not implement later stages before the current stage has been verified.

Do not fix before thinking.

For meaningful failures, investigate the failure and user hypothesis before applying a non-trivial fix.

Measure meaningful improvement.

Whenever a change is claimed to improve reliability, retrieval, grounding, or output quality, compare before and after using the same or equivalent input.

The purpose of Codex in this repository is not to replace the learner.

The purpose is to help the learner build, observe, debug, measure, and understand the system.


## 이 `AGENTS.md`에서 가장 중요한 부분

이 파일이 들어가면 Codex에게 단순히 **“프로젝트 설명”**만 주는 게 아니라 개발 행동 자체를 제어하게 돼.

특히 아래 흐름을 강제하는 게 핵심이야.

```text
구현
 ↓
희아가 직접 실행
 ↓
관찰
 ↓
문제 발견
 ↓
원인 가설
 ↓
개념 이해
 ↓
최소 수정
 ↓
같은 입력 재실행
 ↓
Before / After
 ↓
Decision Log

프로젝트 계획서에서 정의한 핵심 학습 방식과 그대로 연결된다. 계획서에서도 오류 발생 시 Codex/GPT가 곧바로 정답을 제공하기보다 입력·출력 계약과 원인 가설을 먼저 생각하고, 이후 수정과 재측정을 수행하도록 설계되어 있다.

그리고 의도적으로 Python 코딩 스타일, PEP8, 함수 길이 같은 일반론은 거의 넣지 않았어. 그런 규칙보다 이번 프로젝트에서는 Scope, Stage Gate, Failure Learning, Evidence Integrity, Evaluation이 훨씬 중요하기 때문이야.