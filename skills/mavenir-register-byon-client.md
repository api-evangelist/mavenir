---
name: Register a BYON client and keep it connected (Mavenir RACM)
description: >-
  Register a device into a mobile operator's IMS network through a Mavenir BYON
  RACM deployment, open the notification channel, register a push token, refresh
  the access token, and deregister cleanly.
api: openapi/mavenir-byon-racm-openapi.yml
operations:
  - POST /session
  - PUT /session/{sessionId}
  - DELETE /session/{sessionId}
  - POST /push
generated: '2026-07-25'
method: generated
source: openapi/mavenir-byon-racm-openapi.yml
---

# Register a BYON client and keep it connected (Mavenir RACM)

> **Note on operation names.** The RACM definition declares **no `operationId` on
> any operation**. This skill therefore addresses operations by method + path, and
> so must any client generated from it. Do not invent operationIds; if you need
> them, the suggested names are recorded in
> `overlays/mavenir-byon-racm-overlay.yaml` as API Evangelist annotations, not as
> Mavenir's.

## Before you start

**There is no Mavenir-hosted endpoint.** The server is templated as
`{apiRoot}/racm/{apiVersion}/{userId}` with `apiRoot` defaulting to
`http://localhost:9091`. You need the base URL of an operator's WebRTC Gateway
deployment, the `apiVersion` (`v1`) and the `userId` the operator allocated. Access
comes through a carrier or a Mavenir channel partner; Mavenir sells no self-serve
access.

You also need an HTTP Bearer access token from the operator's auth server. Every
operation declares `BearerAuth` with the scope strings `read` and `write`.

`transactionId` (header) is required on every operation in this skill.

## Steps

### 1. Create the RACM session — `POST /session`

Headers: `transactionId`. Body: `RacmRequest` with `deviceId` (your device id in
UUID format). Note this is the **only** operation here that does not require the
`clientId` header — it is the operation that mints it.

`200 OK` returns `RacmResponse`:

- `connectionInformation.clientId` — **store this.** It is required as a header on
  every subsequent RACM operation and on every call handling operation.
- `notificationChannelInformation.channelURL` — the WebSocket URL to open.
- `resourceURL` — the session URL, with the `sessionId` embedded. Use it verbatim;
  the spec states "There is no need of constructing the URL".

`400` returns the `ErrorInfo` envelope with `INVALID_ARGUMENT`.

### 2. Open the notification channel

Open a WebSocket to `channelURL` and hold it. Call invitations
(`SessionInvitationNotification`) and session state changes
(`SessionStatusNotification`) arrive here — they are not delivered over HTTP and
there are no webhooks. Message shapes are catalogued in
`asyncapi/mavenir-byon-events.yml`.

### 3. Register a push token — `POST /push` (optional)

Headers: `transactionId`, `clientId`. Body: `PushInformation` with `deviceId`
(required), plus `OSName`, `OSVersion`, `appName`, `BundleID` and `WebToken` (the
Web Push FCM registration token). The spec marks this operation optional — "the
client could use any external mechanism to update the PNS server" — and states that
Chrome browser Web Push is currently supported.

`200` means the device token was accepted. `400`/`403` return the `ErrorInfo`
envelope.

> Caution: this operation carries an operation-level `servers` override pointing at
> `http://swaggerhub.intinfra.com/device/v1/{deviceSessionId}` — an authoring
> artifact left in the contributed definition, not a deployable host. Ignore it and
> use the deployment's `apiRoot`.

### 4. Refresh the access token — `PUT /session/{sessionId}`

Headers: `transactionId`, `clientId`, path `sessionId`. This operation exists
solely to share a **new** access token with the WebRTC Gateway; the spec says the
token "is expected to be received from the Auth server". Call it before your
current token expires so an in-progress call is not dropped. Use the `resourceURL`
returned in step 1. `200` on success.

### 5. Deregister — `DELETE /session/{sessionId}`

Headers: `transactionId`, `clientId`, path `sessionId`. `204 No Content` on
success. This ends registration; the notification channel and any active call
handling ability go with it.

> The RACM spec wires its `NotFound404` component to the **400** response of this
> operation — an authoring slip carried verbatim from the source. Handle a `400`
> here as "session not found" as well as bad input.

## Rules

- No idempotency correlator exists on this API. Unlike call handling (which has
  `clientCorrelator`), retrying `POST /session` is not safe by contract — reconcile
  first by checking whether you already hold a `clientId` and `resourceURL`.
- Errors use the custom `{status, code, message}` envelope, not RFC 9457. See
  `errors/mavenir-problem-types.yml`.
- No rate limits, quotas or 429 responses are documented anywhere in this spec.
- Conventions (headers, addressing, versioning) are captured in
  `conventions/mavenir-conventions.yml`.
