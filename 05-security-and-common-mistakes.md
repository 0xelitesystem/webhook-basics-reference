# Security and common mistakes

Pulling the pieces together, a safe and reliable webhook receiver verifies the sender, handles deliveries idempotently, responds quickly, and is bounded against abuse. The common mistakes are the inverses of these, and they tend to be discovered the hard way, through fraud, corrupted data, or missed events. Knowing them in advance is cheaper than learning them in production.

## Acting on unverified input

The most serious mistake is processing webhook payloads without verifying the sender, covered earlier. An endpoint that trusts whatever it receives can be fed forged events by anyone who finds the URL, leading to fraud and corrupted data. The fix is non-negotiable: verify the signature against the shared secret before acting on any payload, and reject anything that does not verify. Relying only on the URL being secret is not enough, because URLs leak, through logs, referrers, and shared configuration. Treat verification as the gate every delivery must pass before its contents are trusted.

## Not being idempotent

The second common mistake is assuming each event arrives exactly once and building handling that breaks on duplicates. Because retries are normal, duplicates happen, and a non-idempotent receiver eventually double-processes an event, with consequences that depend on what the event triggers. The fix is to make processing idempotent using a stable event identifier, so duplicates are recognized and skipped. This is easy to design in and painful to retrofit after a duplicate has already caused a bad outcome, so it belongs in the receiver from the start.

## Slow receivers and missed acknowledgments

A receiver that does heavy work before responding causes the sender to time out and retry events it actually delivered, producing duplicate deliveries and, if combined with non-idempotent handling, duplicate effects. The fix, from the receiving page, is to acknowledge quickly and do the slow work separately, keeping the path the sender waits on thin and fast. Many duplicate-delivery problems trace back to a slow receiver rather than genuine delivery failures, so keeping acknowledgment fast prevents a whole class of issues.

## Leaking or hardcoding the secret

The signing secret is a credential, and the mistakes around it are the same as for any secret: hardcoding it in source, committing it to a repository, logging it, or putting it in a frontend. A leaked webhook secret lets an attacker sign forged requests that pass verification, defeating the protection entirely. The fix is to store the secret as a secret, out of code and logs, and to rotate it immediately if it is ever exposed, treating any exposure as compromise. The secrets-management reference in this portfolio covers this in full; webhooks are simply one more place a secret has to be handled correctly.

## Forgetting abuse and exposure controls

Two further mistakes round out the list. Leaving the endpoint without rate limiting means a flood of requests, forged or replayed, can overwhelm it or run up the cost of whatever it triggers; the rate-limiting reference covers bounding this. And exposing more than necessary in the endpoint, detailed error messages, internal information in responses, or actions that do more than the event needs, widens what a successful forgery or abuse could accomplish. The safe receiver does the least it needs to, reveals little, is rate limited, verifies every sender, handles duplicates, and acknowledges fast. Each of these is one of the mistakes above, avoided. A webhook integration built with all of them in place is dependable and safe to act on; one missing any of them has a known, predictable way to fail.
