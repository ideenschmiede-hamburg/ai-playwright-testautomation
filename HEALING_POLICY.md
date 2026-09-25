# Healing Policy

## Purpose

Healing may repair automation mechanics. It must never change the intended business behavior of a test.

## When Healing Is Allowed

Healing is allowed only when:

- the intended target is unambiguous;
- the failure is likely caused by an unstable locator or interaction;
- the expected business outcome remains unchanged;
- the page is in the expected application state;
- the replacement can be verified through browser observations.

## When Healing Is Forbidden

Do not heal by:

- removing or weakening assertions;
- changing expected values to match actual failures;
- skipping required steps;
- accepting a different business outcome;
- bypassing authentication or authorization;
- injecting application state directly;
- selecting an element only because it makes the test pass;
- changing application code or production data;
- hiding a genuine application defect with retries.

## Failure Classification

Before healing, classify the failure as one of:

- `application_defect`: The application violates the expected behavior.
- `locator_drift`: The intended element exists but its locator changed.
- `timing_instability`: The expected state appears asynchronously.
- `environment_failure`: The application or required dependency is unavailable.
- `test_design_error`: The scenario is ambiguous or invalid.
- `unknown`: There is insufficient evidence.

Automatic healing is permitted only for `locator_drift` and bounded `timing_instability`.

## Locator Selection

Prefer stable, user-oriented locators in this order:

1. semantic role combined with accessible name;
2. stable test identifier;
3. associated label or other accessibility relationship;
4. stable visible text when unique;
5. stable structural locator as a last resort.

Avoid:

- generated CSS classes;
- deeply nested CSS selectors;
- positional selectors;
- dynamic IDs;
- XPath tied to document structure;
- text that changes frequently.

A locator must uniquely identify the intended element before it is accepted.

## Healing Procedure

For each healing attempt:

1. Capture the current browser state.
2. Confirm that the correct page and workflow state are active.
3. Record the failed locator or interaction.
4. Inspect available semantic and accessibility information.
5. Generate the smallest reasonable set of replacement candidates.
6. Select the highest-priority unique candidate.
7. retry the failed interaction once with that candidate.
8. Continue the original scenario.
9. Verify the original expected business outcome.
10. Record the attempt and its evidence.

## Limits

- Allow at most two healing attempts per failed interaction.
- Do not repeatedly refresh or restart until a failure disappears.
- Do not apply a candidate that matches multiple elements.
- Do not continue healing after detecting an application defect.
- Do not silently persist an unverified locator change.

When the limit is reached, submit an `errored` result with diagnostic evidence.

## Acceptance of a Healing Proposal

A healing proposal is acceptable only when:

- the replacement target is semantically equivalent;
- uniqueness was verified;
- the original interaction succeeds;
- the original expected outcome succeeds;
- evidence supports the decision;
- the old and proposed locators are documented.

A successful runtime recovery is still a proposal until reviewed or persisted through an explicitly authorized workflow.

## Reporting

The final result must state:

- whether healing was attempted;
- the failure classification;
- the number of attempts;
- the selected replacement, if any;
- whether the original expected outcome was verified.

Healing must remain transparent and auditable.