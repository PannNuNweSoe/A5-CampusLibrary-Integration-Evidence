# A5 – Group-9 – Integration Evidence

**Service:** campus-library-app  
**Base URL:** https://campus-library-app.vercel.app  
**Environment:** Production  
**Deployment ID:** dpl_HSsWFnu9X28PcwhYYNHF8ovfShyS

> ⚠️ **Status note:** Sections 3, 4 (partial), 5, and 6 are not fully evidenced yet. Each incomplete section below states exactly what's missing and how to capture it. Per the audit sheet, a feature without documented evidence is considered incomplete — see the "Still needed" callouts before submitting.

> **Image location:** Put all screenshots in the `evidence/` folder of this repository.  
> The image references below use relative paths so they render directly on GitHub.

---

## 1. Consumer Proof
*(campus-library-app calling an external partner API)*

- **Partner URL:** configured via the `PARTNER_API_URL` environment variable, called from `GET /api/integration/partner-status`
- **Request timestamp:** 2026-09-22T15:20:14.959Z
- **Response body:**

```json
{
  "success": true,
  "partner_data": {
    "slideshow": {
      "author": "Yours Truly",
      "date": "date of publication",
      "slides": [
        { "title": "Wake up to WonderWidgets!", "type": "all" },
        { "items": ["Why <em>WonderWidgets</em> are great", "..."] }
      ]
    }
  }
}
```

**Evidence:** Postman `GET /api/integration/partner-status` → `200 OK`, 407 ms, 619 B.

![Consumer Proof - Partner Status](evidence/01-consumer-partner-status.png)

**Figure 1.** `campus-library-app` calls the configured partner API through `/api/integration/partner-status` and receives a successful `200 OK` response.

---

## 2. Provider Proof
*(campus-library-app acting as the provider, serving its own endpoint)*

- **Endpoint URL:** `GET https://campus-library-app.vercel.app/api/integration/status`
- **Internal request log (Vercel):**

```json
{"requestId":"tq6cw-1790090404457-e3dc8e367226","timestamp":1790090404457,"requestMethod":"GET","requestPath":"/api/integration/status","responseStatusCode":200,"environment":"production","branch":"main"}
```

→ 2026-09-22T15:20:04.457Z

- **Response body (partner-visible confirmation of service identity):**

```json
{
  "success": true,
  "data": {
    "team": "Group-9",
    "service": "campus-library",
    "status": "ok",
    "version": "1.0",
    "timestamp": "2026-09-22T15:03:19.584Z"
  }
}
```

**Evidence:** Postman `GET /api/integration/status` → `200 OK`, 891 ms, 525 B, matched to Vercel log line above.

![Provider Proof - Integration Status](evidence/02-provider-status-postman.png)

**Figure 2.** Provider endpoint confirms the Group-9 Campus Library service identity and returns `200 OK`.

![Provider Proof - Vercel Log](evidence/03-provider-vercel-log.png)

**Figure 3.** Vercel production log confirms the internal request to `/api/integration/status` and the `200` response.

---

## 3. Webhook Receiver
*(campus-library-app receiving an incoming webhook)*

- **Incoming payload:** *not yet captured*
- **Secret verification result:** *not yet captured*
- **Stored log:** *not yet captured*

**Evidence on hand:** only the `GET /api/integration/webhook` sanity check (`200 OK`, `"Webhook endpoint is available. Use POST to send webhook events."`) — this confirms the route exists but is **not** proof of receiving and verifying a real webhook.

![Webhook Receiver - Route Sanity Check](evidence/04-webhook-receiver-get-sanity.png)

**Figure 4.** Sanity check confirms that the webhook route is deployed and reachable. This does not yet prove processing of an incoming webhook.

**Still needed:** In Postman, send `POST /api/integration/webhook` with header `X-Webhook-Secret: <your WEBHOOK_SECRET value>` and a JSON body containing `event_id` and `event_type`. Screenshot:

1. The request (headers + body)
2. The `200` response
3. (If required by your rubric) a Supabase table row or Vercel log line showing the event was stored

<!--
After capturing the missing evidence, add:
![Webhook Receiver - POST Request and Response](evidence/08-webhook-receiver-post.png)

![Webhook Receiver - Stored Event](evidence/09-webhook-receiver-storage.png)
-->

---

## 4. Webhook Sender
*(campus-library-app triggering an outgoing webhook to the partner)*

- **Internal trigger action:** `POST /api/loans/idempotent` (loan creation triggers `sendWebhookToPartner`)
- **Outgoing payload (constructed server-side):**

```json
{
  "event_type": "loan.created",
  "source": "Group-9",
  "data": {
    "id": "7e248bc3-2fa0-49bd-bd51-6562bc4ff9c6",
    "copy_id": "copy-042",
    "user_id": "user-777",
    "status": "active",
    "created_at": "2026-09-22T15:11:25.368Z"
  }
}
```

![Webhook Sender - Loan Trigger](evidence/05-webhook-sender-loan-trigger.png)

**Figure 5.** Loan creation through `/api/loans/idempotent` acts as the internal trigger for the outgoing `loan.created` webhook.

- **Partner response log:** *not fully captured* — the Postman response body was cut off before showing the `webhook_delivery` object's status/body.

**Still needed:** Re-run the request and scroll/screenshot the full `webhook_delivery` field in the response (its `status` and `body`, or its `error` if the partner webhook URL isn't reachable), so the outcome of the outgoing call is documented, not just the trigger.

<!--
After capturing the missing evidence, add:
![Webhook Sender - Partner Delivery Result](evidence/10-webhook-sender-delivery.png)
-->

---

## 5. Idempotency Proof
*(same Idempotency-Key should return the same result on replay)*

### Request 1

`POST /api/loans/idempotent`

Header:

```text
Idempotency-Key: test-key-001
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "7e248bc3-2fa0-49bd-bd51-6562bc4ff9c6",
    "copy_id": "copy-042",
    "user_id": "user-777",
    "status": "active",
    "created_at": "2026-09-22T15:11:25.368Z"
  },
  "idempotent_replay": false
}
```

![Idempotency - Request 1](evidence/06-idempotency-request1.png)

**Figure 6.** First request with `Idempotency-Key: test-key-001` creates the loan and returns `idempotent_replay: false`.

### Matched Vercel Log

```json
{"requestId":"2x2ks-1790090424017-3a7e115b1635","timestamp":1790090424017,"requestMethod":"POST","requestPath":"/api/loans/idempotent","responseStatusCode":200}
```

→ 2026-09-22T15:20:24.017Z

![Idempotency - Vercel Request Log](evidence/07-idempotency-vercel-log.png)

**Figure 7.** Vercel production log confirms the successful first request to `/api/loans/idempotent`.

### Request 2 — Replay

**Request 2 (replay, same key):** *not yet run*

**Still needed:** Send the identical request again with the same `Idempotency-Key: test-key-001` header. Screenshot the response and confirm:

- same `id` as Request 1
- `"idempotent_replay": true`

Optionally also show the `idempotency_keys` row in Supabase as DB-level proof only one loan row was created despite two requests.

<!--
After capturing the missing evidence, add:
![Idempotency - Request 2 Replay](evidence/11-idempotency-request2.png)

![Idempotency - Database Proof](evidence/12-idempotency-db.png)
-->

---

## 6. Degradation Proof
*(graceful fallback when a dependency is unavailable)*

- **Breakage timestamp:** *not yet captured*
- **Fallback JSON output:** *not yet captured*
- **Automatic recovery log:** *not yet captured*

**Relevant code (already implemented, not yet demonstrated):** `GET /api/integration/partner-status` has a 3-second timeout; on failure it returns:

```json
{
  "success": false,
  "error": {
    "code": "DEPENDENCY_TIMEOUT_OR_NETWORK_ERROR",
    "message": "Partner API is unavailable or timed out after 3000ms.",
    "dependency": "partner-api",
    "timeout_ms": 3000,
    "retryable": true
  }
}
```

**Still needed:** Force a failure — e.g. temporarily set `PARTNER_API_URL` in Vercel to an invalid/unreachable URL, redeploy, and hit the endpoint in Postman to capture the `503` fallback response and its timestamp. Then restore the correct URL, redeploy, and capture a follow-up `200` request showing recovery.

<!--
After capturing the missing evidence, add:
![Degradation - Dependency Failure](evidence/13-degradation-failure.png)

![Degradation - Recovery](evidence/14-degradation-recovery.png)
-->

---

## Evidence Image File Map

| Figure | File | Evidence |
|---|---|---|
| 1 | `evidence/01-consumer-partner-status.png` | Consumer API call |
| 2 | `evidence/02-provider-status-postman.png` | Provider API response |
| 3 | `evidence/03-provider-vercel-log.png` | Provider internal log |
| 4 | `evidence/04-webhook-receiver-get-sanity.png` | Webhook route sanity check |
| 5 | `evidence/05-webhook-sender-loan-trigger.png` | Loan-triggered outgoing webhook |
| 6 | `evidence/06-idempotency-request1.png` | First idempotent loan request |
| 7 | `evidence/07-idempotency-vercel-log.png` | First request server log |
| 8 | `evidence/08-webhook-receiver-post.png` | Incoming webhook request/response — pending |
| 9 | `evidence/09-webhook-receiver-storage.png` | Incoming webhook persistence — pending |
| 10 | `evidence/10-webhook-sender-delivery.png` | Partner delivery result — pending |
| 11 | `evidence/11-idempotency-request2.png` | Replay with same key — pending |
| 12 | `evidence/12-idempotency-db.png` | DB single-creation proof — optional/pending |
| 13 | `evidence/13-degradation-failure.png` | `503` fallback — pending |
| 14 | `evidence/14-degradation-recovery.png` | Recovery to `200` — pending |

---

## Appendix — Raw Vercel Log Export

```json
{"requestId":"2x2ks-1790090424017-3a7e115b1635","timestamp":1790090424017,"deploymentId":"dpl_HSsWFnu9X28PcwhYYNHF8ovfShyS","projectId":"prj_M91ikZzIvv3JKmWwQ8BfugsJ3Kww","level":"info","source":"serverless","domain":"campus-library-app.vercel.app","requestMethod":"POST","requestPath":"/api/loans/idempotent","responseStatusCode":200,"environment":"production","branch":"main","cache":"MISS"}
{"requestId":"zmbb6-1790090419506-4d93b5cac3c0","timestamp":1790090419506,"deploymentId":"dpl_HSsWFnu9X28PcwhYYNHF8ovfShyS","projectId":"prj_M91ikZzIvv3JKmWwQ8BfugsJ3Kww","level":"info","source":"serverless","domain":"campus-library-app.vercel.app","requestMethod":"GET","requestPath":"/api/integration/webhook","responseStatusCode":200,"environment":"production","branch":"main","cache":"MISS"}
{"requestId":"k4r85-1790090414959-4174324cd5e3","timestamp":1790090414959,"deploymentId":"dpl_HSsWFnu9X28PcwhYYNHF8ovfShyS","projectId":"prj_M91ikZzIvv3JKmWwQ8BfugsJ3Kww","level":"info","source":"serverless","domain":"campus-library-app.vercel.app","requestMethod":"GET","requestPath":"/api/integration/partner-status","responseStatusCode":200,"environment":"production","branch":"main","cache":"MISS"}
{"requestId":"tq6cw-1790090404457-e3dc8e367226","timestamp":1790090404457,"deploymentId":"dpl_HSsWFnu9X28PcwhYYNHF8ovfShyS","projectId":"prj_M91ikZzIvv3JKmWwQ8BfugsJ3Kww","level":"info","source":"serverless","domain":"campus-library-app.vercel.app","requestMethod":"GET","requestPath":"/api/integration/status","responseStatusCode":200,"environment":"production","branch":"main","cache":"MISS"}
```
