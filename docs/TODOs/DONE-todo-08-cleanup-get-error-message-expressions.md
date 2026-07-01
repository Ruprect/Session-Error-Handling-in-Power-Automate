# TODO 08 — Clean up redundant expressions in Get Error Message helper

**Priority:** LOW / OPTIONAL — these expressions work at runtime but are needlessly convoluted.
Skip this TODO entirely if unsure; nothing is broken.
**File to change:** `.claude/skills/power-automate-error-handling/references/flow-get-error-message.json`
**Type of change:** Two expression simplifications. Both were translated from an Azure Logic App
Bicep template and carried over redundancy from the original.

## Change 1 — `Select_-_Get_Error_Messages` union expression

The `from` expression currently unions the coalesced XPath result with the SAME fallback XPath
three more times (duplicated arguments):

- old (the full `from` value):
  `@union(coalesce(xpath(outputs('Compose_-_Convert_to_XML'), '//error/message[not(preceding-sibling::message)]/text()'), xpath(outputs('Compose_-_Convert_to_XML'), '//message[not(preceding-sibling::message)]/text()'), xpath(outputs('Compose_-_Convert_to_XML'), '//message[not(preceding-sibling::message)]/text()')), xpath(outputs('Compose_-_Convert_to_XML'), '//message[not(preceding-sibling::message)]/text()'), xpath(outputs('Compose_-_Convert_to_XML'), '//message[not(preceding-sibling::message)]/text()'))`

- new (semantically identical — union of the two distinct XPath results, deduplicated):
  `@union(coalesce(xpath(outputs('Compose_-_Convert_to_XML'), '//error/message[not(preceding-sibling::message)]/text()'), xpath(outputs('Compose_-_Convert_to_XML'), '//message[not(preceding-sibling::message)]/text()')), xpath(outputs('Compose_-_Convert_to_XML'), '//message[not(preceding-sibling::message)]/text()'))`

Rationale: `union(coalesce(a, b, b), b, b)` ≡ `union(coalesce(a, b), b)`. Duplicate arguments to
`union` add nothing (union deduplicates).

## Change 2 — `Filter_Array_-_Get_failed_step` where clause

The second disjunct is fully subsumed by the first (`or(X, and(X, Y))` ≡ `X`), and
`item()?['error/message']` is not a valid property path anyway (it looks up a literal key named
`error/message`):

- old (the full `where` value):
  `@or(equals(item()['status'], 'Failed'), and(equals(item()['status'], 'Failed'), equals(item()?['error/message'], null)))`

- new:
  `@equals(item()?['status'], 'Failed')`

(Also switches `item()['status']` to the null-safe `item()?['status']`, matching the expression
documented in CLAUDE.md's Key Expressions Reference.)

## Verification

1. The file must remain valid JSON.
2. Search for `error/message'], null` — must have ZERO matches after the change.
3. Do NOT touch any other action. In particular do NOT change
   `Compose_-_Get_error_message_using_XPath` — its coalesce of two XPath calls is correct as-is.
