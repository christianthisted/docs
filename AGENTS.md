# AGENTS.md — Keen API Documentation

This file provides instructions for AI agents working on the Keen developer documentation (Mintlify). It defines project terminology, writing style, and content boundaries.

## About this project

- This is a documentation site built on [Mintlify](https://mintlify.com)
- Pages are MDX files with YAML frontmatter
- Configuration lives in `docs.json`
- Run `mint dev` to preview locally
- Run `mint broken-links` to check links

## About Keen

Keen is a B2B partner management platform that enables companies to scale through structured referral and reseller partnerships. The platform provides shared lead visibility, attribution by channel, and partnership performance tracking.

There are two sides to every partnership on Keen:

- **Receiving company** — the program owner (hub) that receives referred leads.
- **Referring company** — the partner that refers leads into the channel.

The Keen API lets companies programmatically manage leads and channels within their partnerships.

## Audience

The reader is a **developer at a Keen customer company** integrating the Keen API into their own systems (CRM sync, custom lead forms, internal tools, reporting pipelines). Assume they understand REST APIs and authentication patterns, but do **not** assume familiarity with Keen's domain model. Define domain terms on first use.

## Domain Glossary

Use these terms consistently. Do **not** invent synonyms.

| Term | Definition | Notes |
|------|-----------|-------|
| **Company** | A business entity on Keen. Every API key belongs to exactly one company. | Never say "organization", "tenant", or "account" when referring to a company. |
| **Channel** | A partnership surface between two companies where leads are shared and tracked. Each channel has its own statuses, properties, and reward triggers. | Not a messaging channel. Do not call it a "pipeline" or "funnel". |
| **Lead** | A shared contact record within a channel — the core unit of partnership activity. | Not a "deal" or "opportunity". |
| **Lead status** | A named stage in a lead's lifecycle, defined per channel (e.g. "New lead", "Qualified", "Won"). | Referred to by both `status` (name string) and `lead_status_id` (UUID). |
| **Property definition** | A custom field schema defined on a channel (name, type, options). | Called `property_definitions` in responses. |
| **Property value** | The value of a custom property for a specific lead. | Called `property_values` in responses. |
| **Purchase** | A revenue event tied to a lead, optionally linked to a product. | Used to calculate reward payouts. |
| **Product** | A purchasable item defined by a company, referenced in purchases. | |
| **Reward trigger** | A rule on a channel that automatically generates a reward event when conditions are met (new lead, status change, purchase). | |
| **Reward event** | A payout record generated from a reward trigger, with an amount and approval status. | |
| **API key** | A UUID credential used in the `X-Api-Key` header. Scoped to a single company. | Generated in **Settings > API keys** in the Keen dashboard. |
| **Receiving / Referring** | The two sides of a channel partnership. The receiving company owns the program; the referring company sends leads. | Always lowercase when used inline. |

### Terms to avoid

| Don't use | Use instead |
|-----------|-------------|
| organization, tenant, account | company |
| pipeline, funnel, deal stage | channel, lead status |
| deal, opportunity, prospect | lead |
| commission | reward |
| endpoint | action (when referring to Keen API actions) |
| token, bearer token | API key |

## API Structure

The Keen API is a single HTTP endpoint that accepts JSON POST requests. Every request includes an `action` field that determines the operation. This is not a RESTful resource-per-URL design — document it as an **action-based API**.

**Base URL:** `{KEEN_API_URL}/functions/v1/openapi-operations`

**Authentication:** All actions (except `lead_external`) require the `X-Api-Key` header.

### Available actions

| Action | Purpose |
|--------|---------|
| `get_leads` | List leads for a channel |
| `create_lead` | Create a new lead in a channel |
| `update_status` | Update a lead's status by name |
| `update_status_id` | Update a lead's status by ID |
| `get_channels` | List channels accessible to your company |
| `create_purchase` | Record a purchase against a lead (with product ID) |
| `create_purchase_custom` | Record a purchase with a custom product name and optional reward trigger IDs |
| `lead_external` | Create a lead without authentication (requires the channel's public API to be enabled) |

## Writing Style

### Voice and tone

- **Clear and direct.** Short sentences. Active voice. Say what the API does, not what it "allows you to" do.
- **Professional but approachable.** Match the tone of the Keen app — clean, modern SaaS. No slang, no hype, no filler.
- **Confident, not cautious.** Write "Returns the created lead" not "Should return the created lead".
- **Second person.** Address the reader as "you" / "your company".

### Formatting rules

- Use **sentence case** for all headings (e.g. "Create a lead", not "Create a Lead").
- Use backtick formatting for field names, action names, header names, and values: `channel_id`, `X-Api-Key`, `"New lead"`.
- Use tables for parameter lists and response field descriptions.
- Use fenced code blocks with language tags (`json`, `bash`, `python`) for all examples.
- Keep code examples minimal and realistic — use plausible field values, not "foo" / "bar" / "test123".
- Every action page should include: description, authentication note, request body table, example request, example response, and error cases.

### Language rules

- Write "API key" (two words, lowercase "key"), not "API Key" or "api-key" in prose.
- Write "lead status" (lowercase) in prose, `lead_status_id` in code context.
- Use "returns" not "will return". Present tense throughout.
- Use "request body" not "payload" or "request payload".
- Use "response" not "response body" or "response payload".
- Do not use "please" in technical instructions.
- Do not start sentences with "Note:" or "Important:" — use Mintlify callout components (`<Note>`, `<Warning>`, `<Info>`) instead.

### Example style

Good:

> Pass the channel ID in the request body. The API returns all leads in that channel, ordered by creation date (newest first).

Bad:

> You can use this endpoint to retrieve all of the leads that belong to a specific channel. Please note that leads will be returned in reverse chronological order.

## Content Boundaries

### In scope

- All actions available through the `openapi-operations` function.
- Authentication with API keys (`X-Api-Key` header).
- Channel access rules (your company must be a creator, partner, referring, or receiving company on the channel).
- Lead lifecycle: creating leads, updating statuses, recording purchases.
- Custom properties: how `property_definitions` and `property_values` work.
- The `lead_external` action and how public/no-auth API access works per channel.
- Error responses and status codes (400, 401, 403, 404, 500).
- Reward trigger behavior as a side effect of lead creation, status changes, and purchases (explain that rewards fire automatically — do not document reward configuration).
- Webhook side effects: mention that channel webhooks may fire on lead creation and status updates.

### Out of scope — do not document

- The Keen dashboard UI or how to configure settings in the app (link to the help center if needed).
- Internal Supabase schema, RLS policies, or database structure.
- Other Edge Functions (`lead-operations`, `channel-operations`, `company-operations`, `reward-operations`, `integration-operations`, etc.) — these are internal.
- Workflow automation (triggers, XYFlow, webhook-inbound).
- Integrations (Zapier, HubSpot, Pipedrive, Make.com).
- Keen AI features.
- User authentication (login, signup, OAuth, magic links) — the API uses API keys only.
- Admin operations, team management, or company settings.
- Reseller/partner program management.
- CSV exports, notifications, or analytics.

### When referencing the dashboard

When you need to tell the reader where to find something in the Keen app (e.g. where to generate an API key, where to find a channel ID), use this pattern:

> Navigate to **Settings > API keys** in your Keen dashboard.

Do not describe UI layouts, button positions, or include screenshots of the dashboard.

## Page Structure Convention

Each action page should follow this structure:

1. **Action name** (sentence case, e.g. "Create a lead") — brief one-sentence description of what this action does.
2. **Authentication** — state that `X-Api-Key` is required (or note if it's the `lead_external` exception).
3. **Request** — table of request body fields: field name, type, required/optional, description.
4. **Response** — table of response fields with types and descriptions.
5. **Example** — request and response:

```bash
# Example request
curl ...
```

```json
// Example response
{ ... }
```

6. **Errors** — table or list of possible error responses with status codes and messages.

## File and Folder Conventions

- Use kebab-case for file names: `create-lead.mdx`, `get-channels.mdx`.
- Group API action pages under an `api-reference/` directory.
- Use `_snippets/` for reusable content blocks (auth header, base URL, error format).
- Keep the `docs.json` navigation structure flat within the API reference group.

## Mintlify Components

Prefer these Mintlify components:

- `<ParamField>` for documenting request/response fields.
- `<ResponseExample>` for showing example responses.
- `<CodeGroup>` when showing the same request in multiple languages.
- `<Note>`, `<Warning>`, `<Info>`, `<Tip>` for callouts — use sparingly.
- `<Card>` and `<CardGroup>` for overview/index pages.
- `<Steps>` for sequential instructions (e.g. "Getting started" guides).

## Brand Details

- **Product name:** Keen (capitalized, never "KEEN" or "keen" in prose).
- **Legal entity:** Keen ApS.
- **Email domain:** keenpartner.com.
- **Support email:** Reference "our team" generically; do not hardcode support emails in docs.
- **Logo:** Do not embed logos in documentation pages — Mintlify handles branding via `docs.json`.
