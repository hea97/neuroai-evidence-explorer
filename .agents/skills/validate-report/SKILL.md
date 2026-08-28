---
name: validate-report
description: Validate manually-created analysis.json against NeuroAI Evidence Explorer V0 contracts, verify paper_id provenance, and build the final Markdown evidence brief. Use for analysis validation, reference validation, report generation, evidence/interpretation/hypothesis separation, uncertainty handling, and source traceability.
---

# Validate Report

## Purpose

Use this skill to validate Human-in-the-loop AI analysis and convert valid structured analysis into the final NeuroAI Evidence Explorer V0 Markdown brief.

Core flow:

`papers.json`  
+ `analysis.json`  
→ Schema Validation  
→ Paper Reference Validation  
→ Evidence / Interpretation / Uncertainty Separation  
→ Markdown Brief  
→ Source Traceability

This stage does not call an LLM.

The analysis has already been created manually through ChatGPT.

The purpose of this skill is to determine whether that analysis is safe enough to enter the final report.

---

## 1. Scope

This skill owns:

- loading `analysis.json`
- validating analysis against Pydantic contracts
- validating allowed confidence values
- preserving `not_reported`
- verifying `paper_id` references
- linking analysis back to retrieved papers
- generating the final Markdown brief
- separating scientific evidence from interpretation
- preserving open uncertainty
- presenting testable hypotheses separately
- including traceable source links
- report-focused tests

Typical files:

- `src/neuroai_explorer/report.py`
- `src/neuroai_explorer/schemas.py`
- `tests/test_report.py`
- `tests/test_schemas.py`
- `examples/sample_brief.md`
- `docs/decisions.md`

---

## 2. Out of Scope

Do NOT use this skill to implement:

- Semantic Scholar retrieval
- query optimization
- evidence packet generation
- automated ChatGPT calls
- LLM APIs
- PDF parsing
- RAG
- embeddings
- vector databases
- reranking
- full evaluation pipeline
- web UI
- database infrastructure
- deployment

This skill begins after `papers.json` and `analysis.json` exist.

---

## 3. Inputs

The report stage consumes two trusted-or-to-be-validated artifacts.

### Retrieved Papers

`papers.json`

Conceptual contract:

```text
Paper
- paper_id: str
- title: str
- abstract: str | None
- authors: list[str]
- year: int | None
- url: str | None
- citation_count: int | None
- source: Literal["semantic_scholar"]
```

### Human AI Analysis

analysis.json

Conceptual contract:
```
PaperAnalysis
- paper_id: str
- research_question: str | "not_reported"
- method: str | "not_reported"
- dataset: str | "not_reported"
- main_finding: str | "not_reported"
- limitation: str | "not_reported"
- evidence_quote_or_paraphrase: str
- confidence: Literal["high", "medium", "low"]
```
Do not trust analysis.json merely because it is valid JSON.

It must also satisfy the project contract.

## 4. Output

The target artifact is:
```
outputs/<question-slug>_brief.md
```
The report should allow a reader to quickly understand:
```
the research question
which papers were retrieved
what each paper reports
what information was not reported
what is evidence
what is interpretation
what remains uncertain
what hypothesis could be tested next
where each source can be traced
```
The brief should not present itself as a complete literature review.

## 5. Validation Order

Validate in this order:
```
JSON can be loaded.
Analysis structure matches the Pydantic contract.
Required fields exist.
Restricted values are valid.
not_reported is preserved where used.
Every analysis paper_id exists in retrieved papers.
Source metadata can be joined back to the paper.
Only then generate the report.
```
Do not build the Markdown report from invalid analysis.

## 6. Schema Validation

The report stage must validate analysis.json before using it.

Check:
```
required fields
field types
confidence enum
analysis object structure
allowed literal values
```
If validation fails:
```
do not silently repair arbitrary fields
do not discard the invalid record without explanation
identify the failing field
provide a readable validation failure
```
For meaningful failures, use:
```
$debug-learning
```
## 7. not_reported Preservation

not_reported is meaningful data.

Do not replace it with:
```
guessed scientific information
empty strings
generic text such as "unknown"
outside knowledge
inferred methodology
```
If the analysis reports:
```
dataset = "not_reported"
```
the final report should preserve that uncertainty.

Do not make the report appear more complete than the evidence.

## 8. Paper Reference Validation

Every analysis record must reference a retrieved paper.

Conceptual rule:
```
analysis.paper_id ∈ retrieved_paper_ids
```
If not:
```
reject or clearly fail validation
identify the invalid ID
do not create a placeholder paper
do not search the web to repair it
do not silently drop the record
```
Example:
```
Unknown paper_id:
abc123
```
```
Valid retrieved paper IDs:
def456
ghi789
```
This is a provenance failure.

## 9. One Source of Truth

Use papers.json as the source of truth for retrieved paper metadata.

The analysis should not be allowed to redefine:
```
title
authors
year
URL
citation count
```
When building the report:
```
Analysis
→ contributes structured interpretation/extraction

Retrieved Paper
→ contributes identity and provenance
```
Do not duplicate conflicting metadata from AI output.

## 10. Scientific Evidence

The Scientific Evidence section should contain claims grounded in the supplied paper evidence.

Prefer information such as:
```
reported method
reported dataset
reported finding
explicitly reported limitation
evidence quote or faithful paraphrase
```
Do not turn interpretations into evidence.

A plausible scientific statement is not evidence unless supported by the retrieved material.

## 11. Interpretation

Interpretation is allowed only when clearly labeled.

Interpretation may include:
```
comparison between papers
conceptual synthesis
possible relationships
implications suggested by the evidence
```
Do not present interpretation as directly reported by a paper.

Use language that preserves the distinction between:
```
the paper reports
```
and:
```
the evidence may suggest
```
## 12. Open Uncertainty

The report must explicitly preserve unresolved questions.

Open uncertainty may come from:
```
missing abstracts
not_reported fields
incomplete methodology details
disagreement between papers
evidence unavailable at abstract level
unclear cross-disciplinary comparability
```
Do not hide uncertainty to make the report sound confident.

V0 is abstract-only.

That limitation matters.

## 13. Testable Hypothesis

A Testable Hypothesis must be clearly separated from Scientific Evidence.

A hypothesis may extend beyond directly reported findings, but it must be presented as a proposition to investigate.

Do not phrase hypotheses as established facts.

Prefer:
```
A testable hypothesis is that...
```
Avoid:
```
This proves that...
```
The final brief should make the epistemic status obvious.

## 14. Required Brief Structure

The final Markdown brief should contain:
```
Research Question
```
Original research question.

Paper Comparison

A compact comparison of retrieved papers.

Useful columns may include:
```
paper
year
method
dataset
main finding
limitation
confidence
```
Do not force unsupported values into the table.

Use not_reported when appropriate.

Scientific Evidence

Evidence grounded in supplied paper information.

Interpretation

Clearly labeled synthesis or comparison.

Open Uncertainty

What cannot be concluded from current evidence.

Testable Hypothesis

A clearly labeled next hypothesis.

Source Links

Traceable paper sources.

V0 Limitations

At minimum mention:
```
abstract-only evidence
Semantic Scholar as a single source
Human-in-the-loop AI analysis
```
Do not imply full-text verification.

## 15. Paper Comparison Table

The comparison table should help the user compare papers using the same contract.

Prefer consistency over completeness.

Do not add columns merely because they look useful.

If a field is unavailable:
```
not_reported
```
should remain visible.

Do not infer missing table values from other papers.

## 16. Source Links

Every paper with an available URL should remain traceable from the brief.

Prefer displaying:
```
title
paper ID
source URL
```
Do not fabricate missing URLs.

Do not generate citations from memory.

Citation traceability is a V0 evaluation metric.

The report should make later manual verification possible.

## 17. Evidence Traceability

A reader should be able to move conceptually from:
```
Final Brief Claim
→ Paper Analysis
→ paper_id
→ Retrieved Paper
→ Abstract / Source URL
```
Do not generate report claims that cannot participate in this chain.

Traceability is more important than narrative smoothness.

## 18. No Hidden Scientific Enrichment

Do not enrich the report using:
```
Codex model knowledge
general neuroscience knowledge
general machine-learning knowledge
web search
related papers not retrieved
assumptions about standard methods
```
unless the user explicitly changes the task and asks for outside research.

For V0 report generation, use the provided evidence pipeline only.

## 19. Confidence Interpretation

Allowed values:
```
high
medium
low
```
Confidence reflects how clearly the supplied evidence supports the extraction.

Do not treat confidence as:

scientific truth probability
paper quality
replication likelihood
general credibility score

Display or use confidence conservatively.

## 20. Invalid Analysis

If the Human-in-the-loop output is invalid, do not automatically rewrite the scientific content.

Possible invalid cases:
```
malformed JSON
missing required field
invalid confidence
unknown paper ID
wrong field type
unsupported contract value
```
Prefer:
```
Validation Failure
→ Diagnose
→ Correct Input Contract
→ Validate Again
```
Do not create an automated LLM repair loop during V0.

## 21. Report Generation Rule

Report generation should be deterministic where practical.

Given the same:
```
research question
retrieved papers
validated analysis
```
the Markdown structure should be predictable.

Avoid unnecessary stylistic randomness.

This improves:
```
testing
debugging
before/after comparison
portfolio reproducibility
```
## 22. Testing

Tests should protect meaningful report behavior.

Important cases include:
```
valid analysis produces a brief
unknown paper_id fails
invalid confidence fails
not_reported remains visible
paper metadata comes from retrieved papers
source links are preserved
required report sections exist
missing optional metadata does not crash report generation
```
Do not compare the entire Markdown file byte-for-byte unless exact formatting is part of the contract.

Prefer section and content assertions.

## 23. Provenance Test

Include at least one test where:
```
analysis.json references a valid paper ID
retrieved paper metadata contains the source URL
generated brief preserves that link
```
Also include a failure case where the analysis references an unknown paper ID.

This directly protects citation traceability.

## 24. Workflow

When this skill is invoked:
```
Load retrieved papers.
Load analysis.json.
Validate analysis schema.
Build the set of valid retrieved paper_id values.
Validate all analysis references.
Join analysis records to paper metadata.
Preserve not_reported.
Build the Paper Comparison section.
Build Scientific Evidence.
Build Interpretation separately.
Build Open Uncertainty.
Build Testable Hypothesis separately.
Add Source Links.
Add V0 limitations.
Save Markdown brief.
Run focused tests.
Ask the user to inspect the brief.
Stop before evaluation.
```
For meaningful failures, use:
```
$debug-learning
```
## 25. Critical Rules

Always follow these rules:
```
Validate before report generation.
Never trust arbitrary AI JSON.
Every paper_id must map to a retrieved paper.
Retrieved paper metadata is the provenance source of truth.
Preserve not_reported.
Never invent missing scientific information.
Keep Evidence separate from Interpretation.
Keep Hypothesis separate from Evidence.
Preserve uncertainty.
Preserve source links.
Do not automate LLM repair.
Stop before the evaluation stage.
```
## 26. Acceptance Criteria

The validate-report stage is accepted when:
```
Analysis Validation
valid analysis passes Pydantic validation
malformed analysis fails
confidence accepts only:
high
medium
low
not_reported remains valid
Reference Validation
every analysis paper_id exists in retrieved papers
unknown paper IDs fail clearly
Report
Markdown brief is generated
Research Question exists
Paper Comparison exists
Scientific Evidence exists
Interpretation exists
Open Uncertainty exists
Testable Hypothesis exists
Source Links exist
V0 limitations are stated
Traceability
report paper entries can be traced to retrieved paper IDs
available source URLs are preserved
Scientific Integrity
unsupported information is not silently added
evidence and interpretation remain distinct
hypotheses are clearly labeled
Testing
focused schema/report tests pass
```
Do not declare completion only because Markdown was generated.

Validation and provenance must also succeed.

## 27. Stage Gate

After the first valid brief is generated:
```
STOP.
```
Ask the user to inspect:

Is the research question correct?
Are the same retrieved papers represented?
Are not_reported values preserved?
Does any Evidence statement appear unsupported?
Is Interpretation clearly separated?
Is uncertainty visible?
Is the hypothesis clearly labeled?
Can each paper be traced through its source link?

Do not automatically continue into evaluation.

Evaluation is the next V0 stage.

## 28. Completion Response

When finishing a validate-report task, report:
```
Changed
```
What validation or reporting behavior was implemented?

Files Changed

List only files actually changed.

Validation

State what contracts were checked.

Verified

State which tests were actually run.

Run

Provide the exact command for validation/report generation.

Adapt it to the actual repository implementation.

Conceptual example:
```
python -m neuroai_explorer build data/<question-slug>/analysis.json
```
Expe`cted output:
```
outputs/<question-slug>_brief.md
```
Do not document a CLI command unless it actually exists.

Observe

Tell the user what to inspect in the generated brief.

Not Yet Done

State clearly:
```
evaluation is not complete
Baseline vs V0 comparison is not complete
README portfolio cleanup is not complete unless separately verified
```
## 29. Final Rule

The report stage is not where uncertainty disappears.

It is where uncertainty becomes visible and structured.

The desired pattern is:
```
Human AI Output
→ Validate
→ Verify Provenance
→ Preserve Missing Information
→ Separate Epistemic Categories
→ Generate Traceable Brief
```
Prefer:
```
Traceable + Cautious + Incomplete
```
over:
```
Polished + Confident + Unsupported
```
A final brief is trustworthy only when a reader can distinguish:
```
what the papers report
```
from:

what we interpret

from:

what we still do not know

from:

what we propose to test next.