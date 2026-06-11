# Verifying the sender

Because a webhook URL can be called by anyone who knows it, the single most important security measure is verifying that each request genuinely came from the service you expect, before you act on its contents. The standard mechanism is a signature: the sender signs the payload with a shared secret, includes the signature in the request, and you recompute it to confirm the request is authentic and unaltered. An endpoint that skips this acts on whatever it receives, including forgeries.

## Why verification is essential

A webhook receiver that trusts its input is a hole. An attacker who learns the URL can send a fabricated event, a fake payment success, a fake order, a fake account change, and an unverified receiver will process it as real. The consequences range from corrupted data to fraud, depending on what the events trigger. Verification closes this by ensuring that only requests genuinely from the sender, proven by something an attacker cannot forge, are acted on. This is the webhook equivalent of never trusting user input: the payload is untrusted until verified, no matter how genuine it looks.

## How signature verification works

The common method is a shared secret known only to the sender and you. When the sender sends a webhook, it computes a signature over the payload using that secret, typically an HMAC, and includes the signature in a request header. On your side, you take the received payload, compute the signature the same way with the same secret, and compare it to the one in the header. If they match, the request genuinely came from someone who holds the secret, which is the sender, and the payload was not altered in transit. If they do not match, the request is forged or tampered, and you reject it without acting. The secret never travels in the request; only the signature does, which is what makes forgery impractical.

## Verify before reading the payload as trusted

The order matters: verify the signature before you treat the payload as real and act on it. A receiver that parses and acts on the event first, then verifies, has already trusted unverified input. The signature check should be a gate at the front of the receiver, rejecting anything that does not verify before any business logic runs. Verifying against the exact raw payload the sender signed is important too, since reformatting or reparsing the body before verifying can break the comparison; compute the signature over what actually arrived.

## Protect and rotate the secret

The signing secret is exactly the kind of credential the secrets-management reference covers: it must be kept out of code, frontends, logs, and screenshots, stored where access is controlled, and rotated. If the secret leaks, an attacker can sign forged requests that pass verification, which defeats the whole protection, so a leaked webhook secret should be rotated immediately and treated as compromised. Each sending service usually provides the secret when you set up the webhook; store it as a secret, not as a constant in your code, and apply the same rotation discipline you would to any key.

## Verification is necessary, not sufficient

Signature verification proves the request came from the sender and was not altered, which is the main thing, but a complete receiver pairs it with the other defenses in this portfolio. Rate limiting bounds how fast even a legitimate-looking flood can hit the endpoint, as the rate-limiting reference covers. Handling retries and duplicates, covered next, ensures verified events are processed correctly even when delivered more than once. Verification answers who sent this; reliability answers did I handle it exactly once; rate limiting answers can this endpoint be overwhelmed. Together they make a webhook receiver safe to act on, where verification alone makes it merely authentic.
