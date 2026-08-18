# WhatsApp Checker API

Check whether phone numbers are registered on WhatsApp with the NumberChecker asynchronous API.

- **Landing page:** https://numberchecker.ai/products/whatsapp-checker
- **API base:** `https://api.numberchecker.ai`
- **Authentication:** `X-API-Key`
- **Documentation:** https://docs.numberchecker.ai/whatsapp-bulk-checker
- **Get an API key:** https://platform.numberchecker.ai

This is an asynchronous, file-based API. Submit a text file, poll the task, and download the exported result.

## Products and limits

| task_type | Product | Minimum rows | Maximum rows |
|---|---|---:|---:|
| `ws` | WhatsApp Checker | 500 | 10,000,000 |

## Quickstart

```bash
export NUMBERCHECKER_API_KEY="YOUR_API_KEY"
curl -X POST 'https://api.numberchecker.ai/v1/tasks' \
  -H "X-API-Key: $NUMBERCHECKER_API_KEY" \
  -F 'file=@examples/numbers.txt' \
  -F 'task_type=ws'
```

A successful submission returns HTTP `202` (or `200` on a legacy deployment) with a `task_id`. Poll until `status` is `exported`:

```json
{
  "task_id": "d4g8o46p2jvh04o9uolg",
  "status": "pending",
  "total": 5000,
  "estimated_amount": { "amount": "0.500000", "currency": "USD" },
  "message": "Task created successfully"
}
```

```bash
curl -X POST 'https://api.numberchecker.ai/v1/gettasks' \
  -H "X-API-Key: $NUMBERCHECKER_API_KEY" \
  -F 'task_id=YOUR_TASK_ID'
```

Task states are `pending`, `processing`, `exported`, and `failed`. Download `result_url` only after export. Treat HTTP `400`, `401`, `402`, `404`, `413`, and `500` as errors; retry only transient server failures.

An exported task has the counters `total`, `success`, and `failure`, plus `result_url` and, when available, `user_id` and `actual_amount`. Preserve the downloaded column names exactly.

## WhatsApp Checker

Use `task_type=ws`. Upload 500–10,000,000 E.164 phone numbers, one per line.

| Field | Description |
|---|---|
| `number` | Submitted phone number. |
| `activated` | Registration or activation result. |

NumberChecker treats this repository as one standalone product and supports only `task_type=ws`.

## Authentication, input, and error handling

- Send the API key in `X-API-Key`. Server-side examples read it from `NUMBERCHECKER_API_KEY`.
- Upload plain text with one international phone number per line. E.164 format is recommended; the included sample uses fictional numbers.
- `400` means invalid input, task type, or row count; `401` invalid key; `402` insufficient balance; `404` unknown task; `413` oversized upload; `500` temporary server error.
- Retry `500` with capped exponential backoff. Do not retry `400`, `401`, `402`, `404`, or `413` without correcting the cause.
- Never expose an API key in production browser code. The browser example documents the raw request shape only; proxy through your backend.

## Runnable examples

Complete submit → poll → download clients are available for [Python](examples/python), [Node.js](examples/nodejs), [Go](examples/go), [Java](examples/java), [C#](examples/csharp), [PHP](examples/php), [JavaScript](examples/javascript), and [Shell](examples/shell).

Run examples from the `examples` directory so `numbers.txt` resolves correctly. Each language folder contains its prerequisites and command. The C# sample includes a .NET project file; Java uses source-file mode on Java 11+.

## Pricing and responsible use

See https://numberchecker.ai/pricing for current pricing. Use this API only for data you are authorized to process and comply with applicable privacy laws and platform terms. This project is not affiliated with or endorsed by WhatsApp or Meta.

_Last reviewed: 2026-08-17_
