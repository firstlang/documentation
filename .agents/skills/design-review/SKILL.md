---
name: design-review
description: Audit a First specification document or named language feature for every unresolved design ambiguity, propose a concrete resolution for each, and write the complete batch review to an editable Markdown file. Use when the user asks to load or run the design review skill, make a specification handbook-ready, find underspecified behavior, or create an ambiguity review they can approve or revise in one pass.
---

# Design Review

Turn an incomplete First specification into an exhaustive batch review that the user can approve or revise in one Markdown editing pass.

## Establish The Target

1. Resolve the named feature to its primary specification document.
2. If multiple plausible primary documents remain after searching filenames, headings, and references, ask one concise question before continuing.
3. Read the repository's language-facts or overview document when present and all applicable `AGENTS.md` instructions.
4. Do not edit the specification unless the user explicitly asks for edits.

## Audit The Design

Read the primary document completely. Then search the repository broadly enough to find every source that can constrain or contradict it:

- Search the full specification tree for the feature name, terminology, syntax, operators, types, examples, and cross-references.
- Read directly related documents, referenced documents, reference tables, and nearby feature documents whose rules interact with the target.
- Inspect relevant implementation, tests, fixtures, notes, and history when they contain design evidence.
- Treat examples as evidence, not automatically as normative rules.
- Distinguish an unresolved choice from wording that is merely rough. Include an item whenever two reasonable implementations, interpretations, or user-visible behaviors remain possible.
- Include missing behavior, contradictions, unclear boundaries, invalid-input handling, type relationships, inference, evaluation order, mutability, ownership, runtime behavior, syntax, editor behavior, interoperability, diagnostics or notices, and interactions with other features when applicable.
- Split independent decisions into separate items. Combine questions only when they must be decided together.
- Do not invent a resolution during the audit.

## Build The Review

Create a stable ordered ledger before drafting proposals. Order foundational decisions before dependent ones, then draft proposals in that order so later proposals remain consistent with earlier ones.

For every item, provide:

1. A stable number and short title.
2. The unresolved question stated precisely.
3. Why the current material does not determine the answer.
4. The affected source location or interaction when useful.
5. One concrete, handbook-suitable proposed resolution.
6. Brief rationale or consequences inside the proposal when needed to make the choice clear.

Do not invent ambiguities merely to enlarge the review. Do not call a short sample exhaustive. If an unresolved choice exists, do not omit it because its likely resolution seems obvious.

## Validate The Ledger

Before writing the review file, perform a separate evidence pass over every proposed ambiguity. This pass is adversarial: its job is to remove unsupported or hallucinated items, not to preserve the draft.

For each item, verify that the feature, syntax, behavior, or interaction at the center of the ambiguity actually exists in the repository:

- Search the documentation for the exact syntax, term, operator, type, feature name, and nearby concepts used by the item.
- Check related reference tables, examples, tests, fixtures, implementation, and notes when documentation evidence is incomplete.
- Keep the item only when repository evidence shows the underlying feature or interaction is real and the remaining choice is genuinely unresolved.
- Drop the item when the repository explicitly forbids the feature, says it is not supported, or gives a determinate rule.
- Drop the item when no repository evidence can be found that the alleged feature, syntax, or interaction exists.
- Rewrite the item when the evidence supports a narrower real ambiguity than the draft described.

The evidence pass may add a newly noticed ambiguity only when it is discovered while checking repository evidence, not from free association. Any new item must pass the same evidence test before inclusion.

If using helper agents, use one agent to draft the ambiguity ledger and a second agent to validate the ledger against repository evidence. The main agent remains responsible for the final file and must not include an item solely because a helper proposed it.

## Write The Markdown File

Write the complete review to a Markdown file in the repository's documentation root unless the user names another location. Use `<Feature>-Design-Review.md` as the default filename. Save the file in ./+Ambiguity-Ledgers. Do not print the complete review into the conversation; return a concise summary and a link to the file.

Begin the file with:

- A `<Feature> Design Review` title.
- A statement that it is a repository-wide best-effort audit.
- Instructions to leave `agree` unchanged to accept a proposal or replace it with the desired rule.

Use this exact structure for every item:

````markdown
## N. Short title

**Ambiguity:** State the unresolved question and why the current material does not determine it.

**Proposed resolution:** State one concrete rule, with essential rationale, consequences, or examples.

```
Code example explaining the proposal. You may omit this example if you cannot reasonably illustrate the proposal in a code sample.
```

agree
````

Use one blank line between the proposal, `agree`, and the next heading. Verify that the file contains exactly one numbered heading and one standalone `agree` line per ambiguity.

If the output file already exists, inspect it before writing. Preserve user-authored responses and notes. Never replace a response other than an unchanged standalone `agree` without explicit permission.

## Interpret The Completed Review

When the user asks to process their edited review file:

1. Treat an unchanged standalone `agree` as acceptance of that item's proposal.
2. Treat replacement text as the user's resolution or requested change.
3. Treat a missing or blank response as unresolved.
4. Reconcile dependencies in numerical order. A user's replacement takes precedence over proposals and may require revising or skipping later accepted proposals.
5. Identify genuine contradictions or remaining decisions concisely rather than silently guessing.
6. Produce a compact final resolution summary.
7. Do not edit the specification until the user explicitly authorizes applying the decisions.

## Handle Changes And Dependencies

Before writing or updating the review, reconsider every item against the complete ledger. If a foundational proposal would make a later item irrelevant, keep the later stable number, mark its proposal as skipped, and explain the dependency.

If source changes expose a new ambiguity, append it to the end without renumbering existing items. Preserve existing user responses when refreshing an in-progress review.

## Finish The Review

After interpreting the edited file:

1. State that the review is complete.
2. Summarize the final resolutions and skipped items compactly.
3. Identify any ambiguity that remains unresolved.
4. Offer to apply the decisions to the specification, but do not edit it without explicit authorization.

## Examples Of Invocation

- `Use $design-review for the units feature.`
- `Load the design review skill for arrays.`
- `Run a handbook-readiness ambiguity review on 01-Language/03-Data/09-Arrays.md.`
- `Process my edited Arrays-Design-Review.md and summarize the accepted decisions.`
