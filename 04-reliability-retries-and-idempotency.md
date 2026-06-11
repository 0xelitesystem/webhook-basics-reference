# Reliability, retries, and idempotency

Webhook delivery is not perfectly reliable, and the ways services compensate for that create their own problem. Senders retry deliveries they think failed, which means your endpoint can receive the same event more than once, and events can arrive late or out of order. Handling this correctly, so that a duplicate delivery does not cause duplicate effects, is what separates a webhook integration that quietly corrupts data from one that holds up.

## Why deliveries get retried

A sender cannot always tell whether a delivery reached you. If your endpoint is slow to respond, returns an error, or is briefly unreachable, the sender assumes the delivery failed and retries it later, often several times with increasing delays. This retry behavior is a feature: it means a genuine event is not lost just because your endpoint had a momentary problem. But it has a consequence. Sometimes the original delivery actually did reach you, and you processed it, but your acknowledgment was slow or lost, so the sender retries an event you already handled. Now you receive the same event twice. Retries trade the risk of losing events for the risk of duplicating them, and you have to handle the duplicates.

## Idempotency: handle duplicates safely

The defense against duplicate deliveries is idempotency, which means processing the same event more than once has the same effect as processing it once. If your handling is idempotent, a duplicate delivery is harmless: the second time you see the event, you recognize it as already handled and do nothing further. The common way to achieve this is to give each event a stable identifier, which senders usually provide, and to record which event identifiers you have already processed. When an event arrives, you check whether you have seen its identifier; if so, you acknowledge and skip; if not, you process it and record the identifier. This makes duplicate deliveries safe without losing any genuine event.

## Why idempotency matters so much

Without idempotency, duplicate deliveries cause duplicate effects, and those effects can be serious. A payment webhook processed twice might credit an account twice or send two confirmation emails. An order webhook processed twice might create two orders. Because retries are normal and expected, a webhook receiver that is not idempotent will eventually double-process some event, and the damage depends on what the event does. Building idempotency in from the start, rather than discovering the problem after a duplicate causes harm, is the safe path. It is one of those things that is cheap to design in and expensive to retrofit after it has already caused a bad outcome.

## Order is not guaranteed

Webhooks can also arrive out of order, because retries and network timing mean a later event might be delivered before an earlier one, or an old retry might land after newer events. Your handling should not assume that the order events arrive in matches the order they happened in. Where order matters, rely on information in the events themselves, like timestamps or a sequence the sender provides, rather than on arrival order, and design so that an out-of-order delivery does not corrupt state. For many cases, processing each event idempotently and using the event's own data rather than its arrival position is enough to handle reordering gracefully.

## Plan for the endpoint being down

Finally, reliability includes what happens when your endpoint is unavailable. Because senders retry, a brief outage usually resolves itself: the missed deliveries are retried once you are back. But retries do not last forever, so a long outage can mean permanently missed events. Knowing the sender's retry behavior, and having a way to reconcile or re-fetch events you might have missed during an outage, covers this. Many services let you query past events or replay missed deliveries, which is the backstop when retries give up. Designing the receiver to be idempotent also makes such replays safe, since reprocessing a recovered event does no harm. Together, idempotency, attention to ordering, and a plan for outages make a webhook integration dependable rather than fragile.
