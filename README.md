# Webhook Basics Reference

A webhook is a way for one service to notify another the moment something happens, by sending an HTTP request to a URL you provide, instead of you repeatedly asking whether anything has changed. When a payment succeeds, an email bounces, or a form is submitted, the service calls your webhook with the details. Because anyone who finds the URL can call it, the two things that matter most are verifying the request genuinely came from the sender and handling delivery reliably.

This reference covers webhooks in plain language, for someone wiring services together.

## What is inside

- [01-what-a-webhook-is.md](01-what-a-webhook-is.md) push instead of poll, and when to use one
- [02-receiving-and-responding.md](02-receiving-and-responding.md) the endpoint, the payload, and responding fast
- [03-verifying-the-sender.md](03-verifying-the-sender.md) signatures and why an unverified webhook is a hole
- [04-reliability-retries-and-idempotency.md](04-reliability-retries-and-idempotency.md) handling retries and duplicate deliveries
- [05-security-and-common-mistakes.md](05-security-and-common-mistakes.md) the failures that bite, and how to avoid them

## More

Part of a catalog of single-file browser tools and plain-language references, all MIT licensed and dependency-free: [0xelitesystem.github.io](https://0xelitesystem.github.io/). Built by [elitesystem.ai](https://elitesystem.ai).

## License

MIT. Copyright (c) 2026 RBL Industries LLC.
