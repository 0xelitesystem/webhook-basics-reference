# What a webhook is

A webhook is a request a service sends to a URL you control when an event happens, so that you learn about the event immediately without having to ask. It is the reverse of the usual direction: instead of your code calling a service's API to check for news, the service calls your endpoint to deliver the news the moment it occurs. People often describe it as a reversed API call, or as push instead of pull.

## Push instead of poll

Without webhooks, the way to find out whether something changed is to poll: repeatedly call the service and ask, has anything happened yet. Polling is wasteful, because most calls return nothing new, and it is slow, because you only learn about an event at the next poll. A webhook flips this. You register a URL with the service once, and from then on the service sends a request to that URL whenever the relevant event happens. You learn about events as they occur, with no wasted polling, which is why webhooks are the standard way services notify each other in near real time.

## What a webhook delivery looks like

A webhook delivery is an ordinary HTTP request, almost always a POST, sent to your URL, carrying a payload that describes the event, usually as JSON. The payload says what happened, a payment succeeded, a message was delivered, a subscription was cancelled, along with the relevant details. Your endpoint receives this request, reads the payload, and does whatever the event calls for: update a record, send a confirmation, kick off a process. From your side, building a webhook receiver is building a small endpoint that accepts a POST and acts on its contents.

## Common uses

Webhooks show up wherever one service needs to tell another that something happened. Payment processors send them for successful and failed charges, refunds, and disputes. Email and messaging services send them for deliveries, bounces, and replies. Form and survey tools send them on submission. Version control and deployment services send them on pushes and builds. The pattern is the same throughout: an event occurs in one system, and a webhook carries the news to yours so it can react. If you are integrating services, you will almost certainly receive webhooks, and you may also send them.

## The catch: the URL is exposed

The convenience of webhooks comes with a structural risk that shapes everything else in this reference. Your webhook URL is, in practice, reachable by anyone who learns it, and an endpoint that simply acts on whatever it receives will act on fabricated requests just as readily as genuine ones. An attacker who finds or guesses the URL can send fake events, a fake payment success, a fake cancellation, and trick your system into acting on them. This is why a webhook receiver cannot just trust its input. The rest of this reference is largely about closing that gap: verifying the sender, and handling delivery reliably so genuine events are processed exactly once.

## When not to use a webhook

Webhooks fit events you need to react to promptly and that the other service emits. They are a poor fit when you need data on your own schedule rather than when the other service decides, when you need to query or search rather than be notified, or when you control both ends and a direct call is simpler. They also add a moving part: an endpoint that must stay up and reachable. For occasional, on-demand needs, a plain API call when you need the data is simpler than standing up and securing a webhook receiver. Reach for a webhook when the value is in learning about events as they happen.
