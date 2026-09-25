# Evidence Policy

## Purpose

Evidence must make each test result understandable and independently reviewable.

## General Rules

Evidence must be:

- produced by an actual tool invocation;
- directly related to the tested behavior;
- associated with exactly one test run and test case;
- stored using a stable relative path or URI;
- referenced through `evidence_references`;
- free of secrets and unnecessary personal data.

Agents must never invent evidence references.

## Minimum Evidence

### Passed

A passed result requires:

- a verified observable expected outcome;
- a concise description of the observed state.

Capture a screenshot when the visual state is important or when required by the scenario.

### Failed

A failed result requires:

- a screenshot of the failure state when possible;
- the expected outcome;
- the actual observed outcome;
- the failed step or assertion;
- the relevant page URL without sensitive query values.

### Errored

An errored result requires:

- the tool or environment error message;
- the operation being attempted;
- a screenshot when a browser page was available;
- trace, log, or diagnostic output when supported.

### Skipped

A skipped result requires:

- the unmet precondition;
- the reason execution could not proceed;
- evidence of that condition when it can be obtained safely.

## Healing Evidence

Every healing attempt must preserve:

- the failed locator or interaction;
- the observed page state;
- the replacement locator candidate;
- the reason the candidate was selected;
- the validation result;
- screenshots before and after healing when available.

A healed test must not be reported as passed unless the original business outcome is subsequently verified.

## Artifact Organization

Use this conceptual structure:

```text
artifacts/
└── runs/
    └── <test-run-id>/
        └── <test-case-id>/
            ├── failure.png
            ├── healed.png
            ├── trace.zip
            └── execution.log
```

File names must:

- contain no secrets;
- avoid absolute machine-specific paths in reports where possible;
- be deterministic and understandable;
- avoid overwriting unrelated evidence.

## Failure Messages

Failure messages must state:

1. what was attempted;
2. what was expected;
3. what was observed;
4. whether healing was attempted;
5. why the final status was selected.

Avoid vague messages such as “test failed” or “element not found”.

## Evidence Integrity

Agents must not:

- alter screenshots to hide failures;
- reuse evidence from another test case;
- reference nonexistent artifacts;
- claim that a trace or screenshot was captured when the tool did not confirm it;
- expose local credentials or private application data.

If required evidence cannot be captured, record that limitation explicitly.