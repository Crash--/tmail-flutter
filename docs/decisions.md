# Decisions

Decisions taken on tmail-flutter, oldest first. Append only: a decision that changes gets a new entry that replaces the old one and cites it. Entry format: the `twake-decisions` skill of [linagora/twake-guidelines](https://github.com/linagora/twake-guidelines/blob/main/skills/twake-decisions/SKILL.md).

Categories: UX, UI, Product, Architecture, Infra, Tech.

## 2026-11-21, [Architecture] Read requests are sent again once after a transport error

- Decision: the session request and `Mailbox/get` are sent again once, 500 ms later, when they fail with a socket or connection error; writes (`Email/set`, `EmailSubmission/set`, any `*/set`) are never sent again.
- Decided by: Crash-- (product owner), [fix plan of #5172](https://github.com/linagora/tmail-flutter/issues/5172#issuecomment-7058350249); the 500 ms delay: agent ([Crash--/tmail-flutter#1](https://github.com/Crash--/tmail-flutter/pull/1)), to be confirmed by Crash--
- Why: on iOS the first requests after a long time in background can hit a dead socket and show the generic retry banner, while a new connection works at once.
- Scope: mobile and web, session and folder list loading.
