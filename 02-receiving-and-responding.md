# Receiving and responding

A webhook receiver is a small endpoint that accepts the incoming request, reads the event, and responds. Two things about it matter more than they first seem: responding quickly with the right status so the sender knows you got it, and doing the slow work separately so a delivery is never lost because your processing took too long. Getting the receive-and-respond loop right is the foundation everything else sits on.

## The endpoint

A webhook endpoint is a URL that accepts an HTTP POST and reads the payload, usually JSON, from the request body. It needs to be publicly reachable, since the sending service has to be able to call it, and it should accept the method and content type the sender uses. Beyond that it is an ordinary endpoint. The one early decision that matters is keeping the URL hard to guess and treating it as sensitive, since, as the previous page noted, anyone who knows it can send to it. A hard-to-guess URL is a small barrier, not real security, but it reduces casual abuse while the verification on the next page provides the real protection.

## Respond quickly with the right status

The sending service is waiting for your response, and it judges success by the HTTP status you return. You should respond promptly with a success status, typically in the 200 range, to acknowledge that you received the delivery. If you are slow to respond, the sender may consider the delivery failed and retry it, even though you did receive it, which leads to duplicate deliveries. If you return an error status, the sender will usually retry later. So the response is a signal: a quick success status means stop, you got it, while a slow response or an error means the sender will try again. Responding fast and correctly keeps the sender from piling up retries for events you actually received.

## Do the slow work separately

The tension is that acting on an event, updating records, sending email, calling other services, can take time, but the sender wants a fast response. The standard resolution is to separate acknowledgment from processing: when the webhook arrives, do the minimum needed to accept it safely, respond with success immediately, and then do the actual work afterward, outside the request that is keeping the sender waiting. This way a slow piece of processing never causes the sender to think the delivery failed and retry it. The receiver's job in the moment is to accept the event reliably and acknowledge it fast; the work it triggers happens on its own time.

## Accept first, then validate the meaning

There is a useful distinction between accepting a delivery and agreeing the event is one you act on. You should verify the sender, covered next, before trusting the payload at all. But once verified, even an event you do not care about should be acknowledged with a success status rather than an error, because returning an error makes the sender retry an event that will keep being unwanted. Acknowledge that you received it, then decide whether it is relevant. Returning errors for events you simply do not handle turns the sender's retry machinery against you for no reason.

## Keep the receiver simple

The receiver should be as simple as possible: accept the request, verify the sender, acknowledge quickly, hand off the work. Complexity in the receiving path, slow operations, fragile dependencies, anything that can hang, is what causes missed acknowledgments and the retry storms that follow. The reliable pattern is a thin, fast receiver that does little itself and delegates the real work. This keeps the part the sender is waiting on quick and dependable, which is exactly what prevents the duplicate-delivery and missed-event problems that the reliability page addresses.
