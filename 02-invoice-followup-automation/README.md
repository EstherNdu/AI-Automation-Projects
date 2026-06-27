# Invoice Follow-up Automation

An n8n workflow that automatically chases unpaid invoices on behalf of a business, sending escalating reminders so no 
payment slips through the cracks without anyone needing to manually follow up.

## The Problem

Late payments are one of the biggest cash flow problems for freelancers and small businesses, not because clients don't 
intend to pay, but because following up is awkward, time-consuming, and easy to forget. Invoices sit unpaid simply because 
nobody got around to chasing them.

## How It Works

```
Schedule Trigger → Airtable (Search) → If (overdue?) → Switch (which stage?) 
                                                              ↓
                                    Polite Reminder / Firm Follow-up / Final Warning
                                                              ↓
                                              Airtable (Update stage + last contacted)
```

| Node | Purpose |
|---|---|
| **Schedule Trigger** | Runs every morning at 9 AM |
| **Airtable (Search)** | Reads all invoices where Status = "Unpaid" |
| **If** | Checks if the due date has passed **and** enough days have passed since last contact |
| **Switch** | Routes each invoice to the correct follow-up stage (0 → polite, 1 → firm, 2 → final) |
| **Gmail (×3)** | Sends the appropriately-toned email for that stage |
| **Airtable (Update)** | Logs the new stage and last contacted date — one dedicated update node per stage to avoid double-counting |

## Escalation Logic

| Stage | Trigger | Tone |
|---|---|---|
| 0 → 1 | Invoice overdue, no contact yet | Friendly reminder |
| 1 → 2 | 3+ days since last contact | Firmer follow-up |
| 2 → 3 | 3+ days since last contact | Final warning (48-hour deadline) |
| 3+ | No further automated action | Manual follow-up recommended |

The automation **stops automatically** the moment a client's invoice Status is changed to "Paid" in Airtable — no further emails are sent.

## Tools Used

- Airtable (free tier) — invoice database
- Gmail — automated email delivery
- n8n — workflow orchestration

## My Setup Process

I started by creating an Airtable base called **Invoice Tracker** with a table for invoices, since I hadn't used Airtable before and wanted a cleaner way to manage structured business data than a spreadsheet. I set up columns for Client Name, Email Address, Phone Number, Amount, Due Date, Status, Follow up stage, and Last Contacted — then set **Status** to default to "Unpaid" and **Follow up stage** to default to "0," and marked Due Date and Email Address as required fields so incomplete records couldn't slip through.

To connect Airtable to n8n, I generated a Personal Access Token from Airtable's developer hub with read/write record scopes and schema read access, scoped to just the Invoice Tracker base.

With the credentials ready, I built the workflow from scratch in n8n, node by node:

1. **Schedule Trigger** — set to run automatically every morning at 9 AM
2. **Airtable (Search Records)** — reads every invoice where Status equals "Unpaid"
3. **If node** — checks two conditions together: whether the due date has passed, and whether enough time has passed since the client was last contacted (using a fallback date for invoices that have never been contacted, so brand new ones are still picked up on the first run)
4. **Switch node** — routes each invoice based on its current Follow up stage (0 → polite reminder, 1 → firm follow-up, 2 → final warning)
5. **Three separate Gmail nodes** — one per stage, each with its own email tone, escalating from friendly to firm to urgent

I originally connected all three Gmail nodes into a single shared Airtable Update node that incremented the Follow up stage by 1 each time. This caused a real bug — because all three branches fed into one node, the same record was sometimes updated multiple times in a single run, and one of my test invoices ended up with a Follow up stage in the hundreds of thousands. I fixed this by giving each Gmail stage its own dedicated Update node with a hardcoded stage value (1, 2, or 3) instead of an incrementing expression, so no matter how many times a branch fired, the result stayed correct.

I also initially used `.first()` when referencing the Airtable record ID in the Update node, which meant every execution updated the same record regardless of which client was actually being processed. Switching to `.item` fixed this so each client's record updates independently.

Once the stage logic was solid, I tested all three escalation stages manually by adjusting the Follow up stage value directly in Airtable, confirmed the workflow correctly stops contacting a client once their Status is changed to "Paid," and then published the workflow to run automatically on schedule.

## Lessons Learned

- Airtable field data is nested under a `fields` object — reference with `$json.fields['Field Name']`
- Using `.first()` in an Update node always grabs the same record regardless of which item is being processed — use `.item` instead so each record updates independently
- Letting multiple branches feed into one shared Update node causes the same record to be updated multiple times in a single run — solved by using a dedicated Update node per escalation stage with a hardcoded stage value instead of an incrementing expression
- Leaving unused fields visible (but empty) in an Update node causes Airtable to silently overwrite existing data with blank values — only map the fields you actually intend to change
- A date fallback pattern — `{{ $json.fields['Last Contacted'] || '2000-01-01' }}` — ensures brand-new invoices with no contact history are still picked up on the first run
- Airtable's Typecast option resolves date-format mismatches without needing to manually reformat every expression

## Status

✅ Core logic working — escalation, interval spacing, and auto-stop-on-payment all functioning correctly. Currently solo-tested with sample invoices before offering as a service.

## Potential Upgrades

- WhatsApp Business API integration for SMS-style reminders (currently deferred — requires Meta business verification and a dedicated, non-personal WhatsApp number)
- Auto-creating invoice rows from a payment platform (e.g. Paystack/Flutterwave) instead of manual entry
- CRM integration for a full client communication history
