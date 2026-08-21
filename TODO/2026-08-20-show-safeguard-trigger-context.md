# Show and highlight the exact Safeguard approval trigger

## Goal
Safeguard approval prompts currently identify an operation as destructive or otherwise guarded without clearly showing the exact input that triggered the decision. Before approving or denying, the user should see both what will run and why Safeguard intercepted it.

For guarded tool operations, show the relevant triggering input in the approval UI—for example, the complete shell command for a guarded `bash` call. When the guard was triggered by one or more regex matches, visually emphasize the exact matched substring(s) inside that input using the existing theme (for example warning/error color and bold), while keeping the surrounding content readable.

The display must be derived from Safeguard's actual decision evidence, not by independently re-running or approximating policy matching in the TUI. Preserve original input exactly apart from presentation styling, safely escape markup/control content, wrap long input without truncation, and retain the existing approve/deny behavior.

Keep the change minimal: extend the existing Safeguard approval presentation and decision contract only as needed. Do not add policy configuration, expose unrelated/raw tool payload fields, or build a general-purpose syntax highlighter.

## Acceptance criteria
- Every Safeguard approval prompt shows the exact relevant operation input that caused interception, such as the complete command for a guarded shell operation.
- When Safeguard's decision includes regex match evidence, every actual matched span is distinctly highlighted within the displayed input using existing theme styles; surrounding text remains legible.
- The UI consumes match evidence produced by the Safeguard decision path and does not duplicate policy evaluation or rerun regex rules in presentation code.
- Long or multiline triggering input wraps without ellipsis/truncation, preserves meaningful whitespace, and remains associated with the approval reason.
- Overlapping, repeated, Unicode, and multiple-rule match spans render deterministically without corrupting text or ANSI styling.
- Untrusted command/tool text is escaped so it cannot inject Symfony Console markup or terminal control sequences through the approval display.
- Approval, denial, cancellation, tool execution, and audit/logging behavior remain unchanged; no sensitive unrelated payload data is displayed or logged.
- Automated tests cover decision evidence propagation and virtual TUI rendering for plain, matched, multiple-match, multiline, and escaped trigger input; required focused Castor validation passes.

## Workflow metadata
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-20T16:39:29+00:00
