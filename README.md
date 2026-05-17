# remix-emails

HTML email templates for Remix IDE, edited here by marketing and synced into
the Remix platform's `email_templates` table by a sidecar service.

## How it flows

```
You edit .html (and maybe manifest.json) in this repo
      │
      ▼ git push to main
      │
      ▼ GitHub webhook (or manual "Sync from Git" button in Admin)
      │
      ▼ email-templates-sync sidecar
      │   • git pulls this repo
      │   • reads manifest.json
      │   • UPSERTs each template into email_templates (source='git')
      │
      ▼
Admin UI Email Templates page → preview, test-send, etc. (unchanged)
Events dispatcher → renders & sends as configured
```

## Editing a template

1. Edit the relevant `.html` file (or add a new one).
2. If you added a file, add an entry to [`manifest.json`](manifest.json).
3. Commit, push, merge to `main`. Sync runs automatically.

You can also trigger a sync manually from the admin UI's Email Templates page
("Sync from Git" button).

## manifest.json

Each entry maps a `slug` → `file`, plus the metadata stored in DB:

| field         | required | notes                                                        |
| ------------- | -------- | ------------------------------------------------------------ |
| `slug`        | yes      | Stable key. Used by the events dispatcher / send API.        |
| `file`        | yes      | Path relative to repo root.                                  |
| `name`        | yes      | Human-readable name shown in admin.                          |
| `subject`     | yes      | Email subject line. Supports `{{handlebars}}` variables.     |
| `description` | no       | Internal description.                                        |
| `variables`   | no       | Map of `{{var}}` name → human description (for preview UI).  |

## Variable syntax

Templates use Handlebars-style `{{variable}}` placeholders. The events
dispatcher and the test-send tool populate them from the event payload, user
record, and a few standard helpers.

Standard always-available variables:

- `{{user.name}}`, `{{user.email}}`, `{{user.id}}`
- `{{appName}}`
- `{{unsubscribe_url}}` (per-user)

Plus anything you pass via the event payload (e.g. `{{discount.code}}`,
`{{plan_name}}`).

## Previewing locally

Open the `.html` file directly in a browser. For full fidelity (including
variable substitution), use the **Preview** button on the admin Email
Templates page after the next sync — that uses the actual rendering pipeline.

## Conventions

- Keep files self-contained (inline CSS, no external JS).
- Hosted images only (no embedded base64).
- Use `<title>` for the human-readable subject too — sync falls back to this
  if `subject` is omitted in the manifest.
