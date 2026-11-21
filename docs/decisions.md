# Decisions

Decisions taken on tmail-flutter, oldest first. Append only: a decision that changes gets a new entry that replaces the old one and cites it. Entry format: the `twake-decisions` skill of [linagora/twake-guidelines](https://github.com/linagora/twake-guidelines/blob/main/skills/twake-decisions/SKILL.md).

Categories: UX, UI, Product, Architecture, Infra, Tech.

## 2026-11-21, [Architecture] Read requests are sent again once after a transport error

- Decision: the session request and `Mailbox/get` are sent again once, 500 ms later, when they fail with a socket or connection error; writes (`Email/set`, `EmailSubmission/set`, any `*/set`) are never sent again.
- Decided by: Crash-- (product owner), [fix plan of #5172](https://github.com/linagora/tmail-flutter/issues/5172#issuecomment-7058350249); the 500 ms delay: agent ([Crash--/tmail-flutter#1](https://github.com/Crash--/tmail-flutter/pull/1)), to be confirmed by Crash--
- Why: on iOS the first requests after a long time in background can hit a dead socket and show the generic retry banner, while a new connection works at once.
- Scope: mobile and web, session and folder list loading.

## 2026-11-21, [Architecture] The ecosystem request is retried after a transport error

- Decision: on mobile, when `.well-known/linagora-ecosystem` fails with a network or transport error, it is requested again after 1 s, 4 s and 15 s; any other failure, and any failure on web, is final for the session.
- Decided by: Crash-- (product owner), [fix plan of #5172](https://github.com/linagora/tmail-flutter/issues/5172#issuecomment-7058350249); the delays and leaving web out: agent ([Crash--/tmail-flutter#1](https://github.com/Crash--/tmail-flutter/pull/1)), to be confirmed by Crash--
- Why: the ecosystem carries the Sentry configuration; one failed request at launch kept Sentry paused and hid the "Error reporting" toggle for the whole session. On web a missing or CORS-blocked document fails the same way as a dropped connection, so retrying would only delay every ecosystem consumer.
- Scope: mobile, every ecosystem consumer (Sentry, app grid, premium CTA, Drive attachments, Scribe).

## 2026-11-21, [Product] A folder list that fails to load is reported to Sentry

- Decision: when the folders cannot be loaded, the failure is logged as an error (a Sentry event when reporting is on), except when the device has no network.
- Decided by: Crash-- (product owner), [fix plan of #5172](https://github.com/linagora/tmail-flutter/issues/5172#issuecomment-7058350249); the no-network exception: agent ([Crash--/tmail-flutter#1](https://github.com/Crash--/tmail-flutter/pull/1)), to be confirmed by Crash--
- Why: the exception behind the generic retry banner shows up in Sentry.
- Scope: mobile and web, every screen that loads the folder list.

## 2026-11-21, [Product] Errors raised before Sentry is configured wait in memory

- Decision: Sentry still starts only once the ecosystem gives its configuration and consent is known; the errors raised before that are kept in memory and sent once both the configuration and the user's choice are known and reporting is allowed. They are dropped without being sent when reporting is not allowed, when the ecosystem has no valid Sentry configuration, when the user's choice cannot be read (later errors then follow the instance default), when the session ends, and in any case five minutes after the first one. At most 20 are kept.
- Decided by: Crash-- (product owner), [fix plan of #5172](https://github.com/linagora/tmail-flutter/issues/5172#issuecomment-7058350249); the limits (20 errors, 5 minutes): agent ([Crash--/tmail-flutter#1](https://github.com/Crash--/tmail-flutter/pull/1)), to be confirmed by Crash--
- Why: some customers have Sentry disabled, so nothing may leave the device before the decision, but the errors of the launch are the ones that explain what the user sees first.
- Scope: mobile and web, every error and message reported to Sentry.
