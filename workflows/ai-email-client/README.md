# ai-email-client

An AI email client as a kdeps workflow: triage, summarize, draft, reorganize
labels, and unsubscribe — with a single-page web UI. IMAP/SMTP for mail, an LLM
(DeepSeek by default) for the intelligence.

One API route (`POST /api/v1/mail`) dispatches on an `action` field; the browser
UI is served on the same port.

## Actions

| action        | what it does                                                        |
|---------------|---------------------------------------------------------------------|
| `inbox`       | Fetch the inbox over IMAP and triage each email (category, priority, summary, folder). Per-email cache — only new mail hits the LLM. |
| `reorganize`  | Propose a destination folder per email (reusing your real IMAP folders). |
| `apply`       | Move the selected emails into their folders (auto-creates missing folders, case-insensitive match). |
| `labels`      | AI plan to consolidate duplicate/near-duplicate folders (merge + delete). |
| `labelsApply` | Execute the label plan: move mail from redundant folders into canonical ones, then delete the emptied ones. |
| `protect`     | Set a protected-labels list that label cleanup never merges away or deletes. |
| `summarize`   | Summarize one email.                                                |
| `draft`       | Draft a reply in a chosen tone.                                     |
| `send`        | Send a drafted reply over SMTP.                                     |
| `unsubscribe` | List senders with a `List-Unsubscribe` header (RFC 2369/8058).      |
| `unsubApply`  | Unsubscribe from the selected senders (one-click POST or GET).      |

## Configure

Credentials live in `~/.kdeps/config.yaml` (never in the workflow):

```yaml
llm:
  backend: deepseek
  deepseek_api_key: sk-...
smtp_connections:
  default: { host: smtp.gmail.com, port: 587, username: you@gmail.com, password: app-password, tls: true }
imap_connections:
  inbox:   { host: imap.gmail.com, port: 993, username: you@gmail.com, password: app-password, tls: true }
```

Gmail: use an App Password, not your login password.

## Run

```bash
export KDEPS_API_AUTH_TOKEN=dev-token   # required whenever apiServer is set
kdeps run .                             # API + web UI on http://127.0.0.1:16395
```

Open `http://127.0.0.1:16395/`, paste the bearer token, and use the unified view:
per-email **Original / Summarize / Draft / Unsub** plus folder **Move**, with
**Reorganize**, **Label cleanup**, and **Find subscriptions** in the toolbar.

## Notes

- Actions that change your mailbox (move, label cleanup, unsubscribe, send) are
  always preview-then-confirm — the LLM never mutates mail on its own.
- The Original view renders HTML in a sandboxed iframe with a strict CSP;
  remote images are blocked until you toggle "Load remote images".

License: Apache-2.0
