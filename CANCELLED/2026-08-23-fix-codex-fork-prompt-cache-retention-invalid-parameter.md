# Fix Codex fork prompt_cache_retention invalid parameter

## Goal
Fork child launch with project configuration `openai-codex/gpt-5.6-luna` and thinking `xhigh` fails before implementation on the first LLM request. Current preserved diagnostic: `[invalid_parameter/invalid_request_error/prompt_cache_retention]`. Earlier four Luna forks from session 41 failed with the same generic invalid_request_error, but historical records stripped the parameter, so they are likely related but not proven identical. Investigate why `prompt_cache_retention` is sent to the Codex backend/model when unsupported, preserve privacy-safe provider code/type/param/message diagnostics, and make model/provider options capability-correct. Temporary operational workaround: project fork model switched to `deepseek/deepseek-v4-flash`, thinking `high`.

## Acceptance criteria
- A Luna fork no longer sends unsupported `prompt_cache_retention`, or the option is capability-gated correctly for supported endpoints/models.
- Codex websocket/provider errors retain bounded privacy-safe code, type, param, and useful message without prompts, tool output, secrets, or full request bodies.
- Focused deterministic tests reproduce the invalid parameter route and protect option filtering.
- Focused live provider validation confirms a Luna fork reaches its first response when credentials/provider are available.
- No arbitrary waits, soft assertions, or tests exceeding 10 seconds.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-23T21:56:05+00:00

## Task workflow update - 2026-08-28T13:53:04.426Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled after current manual validation: an explicit openai-codex/gpt-5.6-luna fork with xhigh reached and returned its response without reproducing invalid_parameter/invalid_request_error/prompt_cache_retention. The historical incident is no longer observable in the current stack; reopen with a fresh artifact/error if it recurs.
