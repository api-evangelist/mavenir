---
name: Place and manage a 1-1 VVoIP call (Mavenir BYON)
description: >-
  Originate a one-to-one voice or video call over WebRTC against a Mavenir
  BYON call handling deployment, track it to Connected, and terminate it cleanly.
api: openapi/mavenir-byon-call-handling-openapi.yml
operations:
  - postSessions
  - getSessionDetailsById
  - postSessionStatus
  - deleteSessionById
generated: '2026-07-25'
method: generated
source: openapi/mavenir-byon-call-handling-openapi.yml
---

# Place and manage a 1-1 VVoIP call (Mavenir BYON)

## Before you start

**There is no Mavenir-hosted endpoint.** This API is a Mavenir-authored reference
definition contributed to the CAMARA API Backlog. The server is templated as
`{apiRoot}/vvoip/{apiVersion}/{userId}` with `apiRoot` defaulting to
`http://localhost:9091`. You must be pointed at an operator's WebRTC Gateway
deployment by the carrier or channel partner that runs it. If you do not have that
base URL, stop — there is nothing to call.

You must already hold:

- a base URL for the deployment (`apiRoot`), `apiVersion` (`v1`) and your `userId`
- a `clientId`, minted by the RACM registration flow — see
  `mavenir-register-byon-client.md`. Every operation below requires it as a header.
- an HTTP Bearer access token from the operator's auth server (`Authorization:
  Bearer <token>`); the spec declares scope strings `read` and `write`
- an open WebSocket on the notification `channelURL` returned by RACM, because call
  progress arrives there and **not** in the HTTP responses

## Rules that are not optional

- `transactionId` (header, required) on **every** request. It is the only trace
  handle this API has. Generate a fresh one per request and log it.
- `clientId` (header, required) on every request in this skill.
- Set `clientCorrelator` (a UUID you generate) in the create body. The gateway will
  not alter it and returns it on the resource. It exists so you can recover from a
  failed create without double-dialing — **do not retry `postSessions` without
  it.**
- Never construct URLs. Use the `resourceUrl` returned on creation (origination) or
  carried in the invitation notification (termination). The spec says this on every
  operation.
- Errors come back as `{status, code, message}` — a custom envelope, not RFC 9457.
  Codes: `INVALID_ARGUMENT`, `UNAUTHENTICATED`, `PERMISSION_DENIED`, `NOT_FOUND`,
  `INTERNAL`, `NOT_IMPLEMENTED`, `UNAVAILABLE`. See
  `errors/mavenir-problem-types.yml`.
- A call that fails is **not** an HTTP error. `Failed`, `Busy`, `NoAnswer`,
  `NotReachable`, `Declined`, `SessionCancelled` and `Terminated` are values of the
  `status` field. Treat them as terminal outcomes.

## Steps

### 1. Originate the session — `postSessions`

`POST /sessions` with headers `transactionId`, `clientId` and a
`VvoipSessionInformation` body:

- `originatorAddress` (required) and `receiverAddress` (required) — URIs in `tel:`,
  `sip:` or `acr:` form, e.g. `tel:+911234567890`
- `originatorName` / `receiverName` — friendly names
- `offer.sdp` — your inline SDP offer (RFC 4566, CRLF preserved)
- `clientCorrelator` — your UUID
- do **not** send `resourceUrl`

`201 Created` returns the session with `status: Initial` and a `resourceUrl`.
Persist that URL and the `sessionId` inside it.

### 2. Follow progress on the notification channel

Session progress arrives as `SessionStatusNotification` messages on the WebSocket,
carrying `status`, a SIP `responseCode` (180 Ringing, 183 In-progress, 200
Connected), possibly an `answer.sdp`, and `sequenceNumber`. If `offerRequired` is
true the update arrived without SDP and you must send an offer. See
`asyncapi/mavenir-byon-events.yml`.

### 3. Poll only if you must — `getSessionDetailsById`

`GET /sessions/{sessionId}` with `transactionId`, `clientId` and `sessionId`
returns the current `VvoipSessionInformation`. Use the `resourceUrl` from step 1.
This is a fallback for reconciliation, not the primary progress mechanism.

### 4. Change session state — `postSessionStatus`

`PUT /sessions/{sessionId}/status` (the operationId is `postSessionStatus` even
though the method is PUT) with a `ReceiverSessionStatus` body. Use it to:

- answer an inbound call: `status: Connected`, `responseCode: 200`, plus your
  `answer.sdp`
- signal ringing: `status: Ringing`, `responseCode: 180`
- signal in-progress with early media: `status: InProgress`, `responseCode: 183`,
  plus `answer.sdp`
- hold: `status: Hold` with `responseCode: 200`; resume: `status: Resume`

`200` on success. The spec's own examples (`exMT180`, `exMT183`, `exMT200`) show
these exact shapes.

### 5. End the call — `deleteSessionById`

`DELETE /sessions/{sessionId}` cancels (as originator), declines (as receiver) or
terminates an established call. `204 No Content` on success.

**This is a safety-critical action** — it drops a live voice or video call.
`agentic-access/mavenir-agentic-access.yml` classifies it human-in-the-loop
required. An agent should confirm with a human before terminating a session it did
not originate.

## Failure handling

| Code | Meaning | Do this |
|---|---|---|
| `UNAUTHENTICATED` (401) | token missing/expired | refresh the token, then hand the new one to the gateway via the RACM refresh operation |
| `PERMISSION_DENIED` (403) | operation not allowed for this client | do not retry; the client or device is not authorized |
| `NOT_FOUND` (404) | sessionId does not exist | you probably built the URL yourself — use the returned `resourceUrl` |
| `INVALID_ARGUMENT` (400) | schema validation failed | fix the body; check SDP encoding and required address fields |
| `INTERNAL` (500) / `UNAVAILABLE` (503) | gateway side | retry `postSessions` **only** with the same `clientCorrelator` |
