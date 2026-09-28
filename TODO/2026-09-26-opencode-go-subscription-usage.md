# Add OpenCode Go subscription limits to /usage

## Goal
Deferred by user during 2026-09-26 OpenCode Go provider work. OpenCode Go model API key cannot fetch subscription windows. The Pi usage extension at https://github.com/Dakai/pi-opencode-go-usage uses GET https://opencode.ai/console/api/go/status with `x-org-id: wrk_...` and the browser's `__Host-console_session` cookie, then reads `access.meters` fiveHour, week, and month. This is an undocumented console API and requires separate sensitive browser-session credentials. Obtain explicit user approval of storage and refresh UX before implementation; do not silently reuse the model API key or persist a browser cookie. Existing /usage is ProviderQuotaProbeService; virtual proof via TuiUsageCommandVirtualTest.

## Acceptance criteria
- Confirm supported authentication and credential handling with the user before implementing.
- If approved, show five-hour, weekly, and monthly limits for configured Go account; fail safely on expired cookie, missing subscription, malformed API, or transport failure.
- Never log or render browser cookies or API keys; validate at the lowest correct layer with Castor.

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
- Created: 2026-09-26T15:00:13+00:00
