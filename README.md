# MAX Number Checker API — Bulk Verification Examples

The **MAX Number Checker API** is a bulk, asynchronous, file-based API used to verify MAX messenger registration and optionally retrieve documented profile fields. Upload an input file, poll the returned task ID, then download the exported results. This repository contains runnable examples in Python, Node.js, Go, Java, C#, PHP, browser JavaScript, and Shell.

- **Base URL:** `https://api.checknumber.ai`
- **Auth:** `X-API-Key` request header
- **Full API docs:** https://docs.checknumber.ai/max-checker
- **Get an API key:** https://platform.checknumber.ai

> [!NOTE]
> This is a bulk, file-based, asynchronous API. It does not perform a single real-time lookup in one request/response cycle.

## Table of Contents

- [How do I check if a phone number is registered on MAX?](#how-do-i-check-if-a-phone-number-is-registered-on-max)
- [How do I process MAX checks in bulk?](#how-do-i-process-max-checks-in-bulk)
- [What request parameters does the API take?](#what-request-parameters-does-the-api-take)
- [What does the task response look like?](#what-does-the-task-response-look-like)
- [How do I retrieve a MAX profile by phone number?](#how-do-i-retrieve-a-max-profile-by-phone-number)
- [What does the MAX number checker API cost?](#what-does-the-max-number-checker-api-cost)
- [MAX number checker API vs alternatives](#max-number-checker-api-vs-alternatives)
- [Code examples](#code-examples)
- [Status codes](#status-codes)
- [FAQ](#faq)
- [Support & legal](#support--legal)

## How do I check if a phone number is registered on MAX?

Send a `POST` request to `https://api.checknumber.ai/v1/tasks` with the API key, `numbers.txt`, and `task_type=max`. The response contains a `task_id` used to retrieve task status and the final `result_url`.

```bash
curl --location 'https://api.checknumber.ai/v1/tasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'file=@"./numbers.txt"' \
  --form 'task_type="max"'
```

Each line in `numbers.txt` is one phone number in E.164 format, such as `+14155552671`.

## How do I process MAX checks in bulk?

1. **Submit** the input file to `POST /v1/tasks`.
2. **Poll** `POST /v1/gettasks` with the returned `task_id` while status moves from `pending` to `processing`.
3. **Download** the file at `result_url` when status becomes `exported`.

```bash
curl --location 'https://api.checknumber.ai/v1/gettasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'task_id="YOUR_TASK_ID"'
```

See [`examples/`](./examples) for complete submit → poll → download implementations.

## What request parameters does the API take?

**Submit task — `POST /v1/tasks`**

| Parameter | Type | Description |
| --- | --- | --- |
| `file` | file | Text file containing one E.164 phone number per line for the main task type. Email variants use `emails.txt`. |
| `task_type` | string | One of: `max`, `max_full`. |

**Check status — `POST /v1/gettasks`**

| Parameter | Type | Description |
| --- | --- | --- |
| `task_id` | string | Task ID returned by task creation. |

**Main task result fields — `task_type=max`**

| Field    | Description                                       | Example      |
| -------- | ------------------------------------------------- | ------------ |
| `Number` | Phone number in E.164 format                      | +79991234567 |
| `max`    | Whether the number is registered on MAX messenger | yes, no      |

## What does the task response look like?

```json
{
  "created_at": "2026-07-13T08:24:56Z",
  "updated_at": "2026-07-13T08:25:43Z",
  "task_id": "example-task-id",
  "user_id": "example-user-id",
  "status": "exported",
  "total": 1000,
  "success": 998,
  "failure": 2,
  "result_url": "https://example.invalid/results.xlsx"
}
```

All variants use this task envelope. Only `task_type`, input content, and exported result columns vary.

## How do I retrieve a MAX profile by phone number?

Returns MAX registration plus the documented profile and activity fields. Use `task_type=max_full`; the submit, poll, and download workflow is otherwise unchanged.

```bash
curl --location 'https://api.checknumber.ai/v1/tasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'file=@"./numbers.txt"' \
  --form 'task_type="max_full"'
```

Input: one phone number per line in E.164 format (for example, `+14155552671`).

Result fields (copied from the [canonical documentation](https://docs.checknumber.ai/max-full-checker)):

| Field       | Description                                                      | Example                                            |
| ----------- | ---------------------------------------------------------------- | -------------------------------------------------- |
| `Number`    | Phone number in E.164 format                                     | +79991234567                                       |
| `max`       | Whether the number is registered on MAX messenger                | yes, no                                            |
| `id`        | MAX account internal user id (only when `max=yes`)               | 75240310                                           |
| `name`      | Account display name (first / last name, may be partially empty) | Екатерина Сагайдак                                 |
| `gender`    | Account gender (when populated by the user)                      | male, female                                       |
| `last_seen` | Activity status / last seen description                          | last seen recently, last seen 2 days ago           |
| `avatar`    | Avatar image URL (empty when user has no avatar)                 | https://i.oneme.ru/i?r=...                         |

## What does the MAX number checker API cost?

Rates vary by task type and are billed against the account balance. See the current terms at **https://checknumber.ai/pricing** and query the remaining balance with `GET https://api.checknumber.ai/v1/balance`.

## MAX number checker API vs alternatives

| Capability | CheckNumber API | Generic carrier lookup | Manual checking |
| --- | --- | --- | --- |
| Checks the documented MAX signal | Yes | No | May be possible |
| Bulk file upload | Yes | Varies | No |
| Asynchronous processing | Yes | Varies | No |
| Downloadable structured results | Yes | Varies | No |

*This compares general approaches, not named vendors. Capabilities vary by provider and account access.*

## Code examples

| Language | Example |
| --- | --- |
| Python | [`examples/python`](./examples/python) |
| Node.js | [`examples/nodejs`](./examples/nodejs) |
| Go | [`examples/go`](./examples/go) |
| Java | [`examples/java`](./examples/java) |
| C# | [`examples/csharp`](./examples/csharp) |
| PHP | [`examples/php`](./examples/php) |
| JavaScript (browser) | [`examples/javascript`](./examples/javascript) |
| Shell (curl) | [`examples/shell`](./examples/shell) |

Set the API key before running a server-side example:

```bash
export CHECKNUMBER_API_KEY="YOUR_API_KEY"
```

## Status codes

| Status | Billing | Description |
| --- | --- | --- |
| `200` | `charge` | Request successful; task created or status retrieved. |
| `400` | `free` | Invalid parameters or file format. |
| `500` | `free` | Internal server error; retry later. |

## FAQ

**Is this a single-number real-time lookup?**  
No. Submit a file, poll for completion, and download the result file.

**Which task types are available in this repository?**

- `task_type=max` — [MAX Number Checker](https://docs.checknumber.ai/max-checker); Checks whether each phone number is registered on MAX messenger. Input: numbers.txt.
- `task_type=max_full` — [MAX Full Profile Checker](https://docs.checknumber.ai/max-full-checker); Returns MAX registration plus the documented profile and activity fields. Input: numbers.txt.

**Which input format should I use?**  
Phone-based checks require one E.164 number per line. Email variants require one email address per line.

**How do I check my balance?**  
Call `GET https://api.checknumber.ai/v1/balance` with the `X-API-Key` header.

## Support & legal

- **Documentation:** https://docs.checknumber.ai/max-checker
- **Dashboard and API keys:** https://platform.checknumber.ai
- **License:** [MIT](./LICENSE) for the sample code

Process only data you are authorized to use and comply with applicable privacy laws and platform terms. CheckNumber is not affiliated with or endorsed by MAX.

---

*Last updated: 2026-07-13 · Maintained by CheckNumber. Canonical docs: https://docs.checknumber.ai/max-checker*
