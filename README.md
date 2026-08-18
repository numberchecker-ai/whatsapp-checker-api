# WhatsApp Checker API — Bulk Registration Verification | NumberChecker

The **NumberChecker WhatsApp Checker API** verifies whether phone numbers are registered on WhatsApp. It is an asynchronous, file-based API for bulk list cleaning: upload E.164 phone numbers, poll the task, and download the exported registration results. This is the official NumberChecker example repository, with integrations in seven languages.

- **Product page:** https://numberchecker.ai/products/whatsapp-checker
- **API documentation:** https://docs.numberchecker.ai/whatsapp-bulk-checker
- **API base URL:** `https://api.numberchecker.ai`
- **Authentication:** `X-API-Key`
- **Get an API key:** https://platform.numberchecker.ai

> [!NOTE]
> This repository covers the standalone NumberChecker registration product `ws` only. Activity, avatar, demographic, and profile-enrichment products are outside its scope.

## What does the NumberChecker WhatsApp Checker API verify?

The API accepts a text file containing 500–10,000,000 phone numbers and returns a bulk task result. Each exported row contains the submitted `number` and an `activated` registration result. It does not send messages or expose private WhatsApp content. Use the result as a registration signal and process only numbers you are authorized to verify.

| API fact | Value |
|---|---|
| Product code | `ws` |
| Input | Text file, one phone number per line |
| Recommended format | E.164, including country code |
| Processing mode | Asynchronous bulk task |
| Minimum batch | 500 numbers |
| Maximum batch | 10,000,000 numbers |
| Export fields | `number`, `activated` |

## How do I check WhatsApp numbers in bulk?

The integration has three steps:

1. Submit the input file to `POST /v1/tasks` with `task_type=ws`.
2. Poll `POST /v1/gettasks` with the returned `task_id`.
3. When `status=exported`, download the result from `result_url`.

```bash
export NUMBERCHECKER_API_KEY="YOUR_API_KEY"

curl -X POST 'https://api.numberchecker.ai/v1/tasks' \
  -H "X-API-Key: $NUMBERCHECKER_API_KEY" \
  -F 'file=@examples/numbers.txt' \
  -F 'task_type=ws'
```

A successful submission returns HTTP `202` (or `200` on a legacy deployment) and a `task_id`:

```json
{
  "task_id": "d4g8o46p2jvh04o9uolg",
  "status": "pending",
  "total": 5000,
  "estimated_amount": { "amount": "0.500000", "currency": "USD" },
  "message": "Task created successfully"
}
```

Poll the task until it reaches a terminal state:

```bash
curl -X POST 'https://api.numberchecker.ai/v1/gettasks' \
  -H "X-API-Key: $NUMBERCHECKER_API_KEY" \
  -F 'task_id=YOUR_TASK_ID'
```

Task states are `pending`, `processing`, `exported`, and `failed`. Download `result_url` only after export. An exported task includes `total`, `success`, and `failure`, plus `result_url` and, when available, `user_id` and `actual_amount`. Preserve the downloaded column names exactly.

## What are the request and result fields?

**Submit task — `POST /v1/tasks`**

| Parameter | Type | Description |
|---|---|---|
| `file` | file | Text file containing one phone number per line. |
| `task_type` | string | Must be `ws` for this product. |

**Check task — `POST /v1/gettasks`**

| Parameter | Type | Description |
|---|---|---|
| `task_id` | string | Identifier returned when the task was created. |

**Downloaded result**

| Field | Description |
|---|---|
| `number` | Submitted phone number. |
| `activated` | WhatsApp registration or activation result returned by the service. |

## How are authentication and errors handled?

- Send the API key in `X-API-Key`. Server-side examples read it from `NUMBERCHECKER_API_KEY`.
- `400` means invalid input, task type, or row count; `401` invalid key; `402` insufficient balance; `404` unknown task; `413` oversized upload; `500` temporary server error.
- Retry transient `500` responses with capped exponential backoff. Do not retry `400`, `401`, `402`, `404`, or `413` without correcting the cause.
- Never expose an API key in production browser code. The browser example demonstrates the request shape only; proxy production requests through your backend.

Check the current account balance with `GET /v1/balance` using the same authentication header.

## Runnable examples in seven languages

Complete submit → poll → download clients are available for:

| Language | Example |
|---|---|
| Python | [`examples/python`](examples/python) |
| Node.js | [`examples/nodejs`](examples/nodejs) |
| Go | [`examples/go`](examples/go) |
| Java | [`examples/java`](examples/java) |
| C# | [`examples/csharp`](examples/csharp) |
| PHP | [`examples/php`](examples/php) |
| Shell / curl | [`examples/shell`](examples/shell) |

Run examples from the `examples` directory so `numbers.txt` resolves correctly. Each language folder lists its prerequisites and command.

## Frequently asked questions

### Is this a realtime single-number API?

No. It is an asynchronous bulk API designed for uploaded number lists.

### Which NumberChecker product does this repository support?

Only `task_type=ws`, the standalone WhatsApp registration checker. Other WhatsApp activity and profile products are outside this repository's scope.

### What phone-number format should I use?

Use international E.164 format with a leading `+` and country code, one number per line.

### Where can I find current pricing?

See the [NumberChecker pricing page](https://numberchecker.ai/pricing). This repository intentionally does not hard-code a price.

## Official resources and responsible use

- **Product:** https://numberchecker.ai/products/whatsapp-checker
- **Documentation:** https://docs.numberchecker.ai/whatsapp-bulk-checker
- **Dashboard and API keys:** https://platform.numberchecker.ai
- **Pricing:** https://numberchecker.ai/pricing
- **OpenAPI contract:** [`openapi.yaml`](openapi.yaml)
- **License:** [MIT](LICENSE) for the sample code

Use this API only for data you are authorized to process and comply with applicable privacy laws and platform terms. WhatsApp and Meta trademarks belong to their respective owners; no affiliation or endorsement is implied.

---

*Last reviewed: 2026-08-18 · Maintained by NumberChecker. Canonical product page: https://numberchecker.ai/products/whatsapp-checker*
