# Case Study: Why Good Leads Go Cold, and How I Stopped It

**An AI-powered lead qualification system built with n8n**

---

## The problem

Think about a small agency or service business. They have a "Contact Us" or "Book a Consultation" form on their website, 
and 30 to 50 people fill it in every week and someone on the team opens each submission by hand and decides who to call first. 
However, this practice goes wrong in a few predictable ways:

- A person with a real budget and a two-week deadline waits a day or two for a reply, because their message was buried under
vague ones.
- Time gets spent on people who were just browsing.
- Nobody keeps a consistent record of who came in, how promising they were, or what happened next.

The slow response matters most. A lead who is ready to buy and hears nothing for 48 hours will often talk to someone else.

## The goal

Reach out to serious leads automatically, consistently and within seconds of the form being submitted, so the right 
people get a reply fast and the sales team only spends time where it counts.

## The solution

I built a system that reads every form submission as it arrives, scores it, and decides what happens next:

- **Hot and warm leads** get a short, friendly email with a link to book a meeting. At the same moment, the sales rep 
gets a notification with the lead's details and the reason they were scored that way.
- **Cold leads** are not ignored. They are logged so no lead is lost
- **Every lead**, whatever the score, is added to a Google Sheet with the scores and the reasoning, so the business has 
one clean record.

## How it works

1. A lead fills in a Google Form.
2. A small Google Apps Script fires the moment the form is submitted and sends the answers to n8n through a webhook. 
3. In n8n, an AI agent reads the submission and scores it against a rubric.
4. A Switch node sorts the lead into hot, warm or cold.
5. Gmail sends the lead a submission confirmation mail with a link to schedule a call when it is convenient for them
and sends the rep the ddetails of the lead for immediate follow up.
6. Google Sheets logs the lead.

### The scoring rubric

The AI does not just give a vague "good lead" or "bad lead". It scores four things from 1 to 5, and each one carries a 
different weight:

| What it looks at | Weight | Why |
|---|---|---|
| Urgency and timeline | 30% | Someone who needs this soon is far more likely to act |
| Budget fit | 25% | Money is a strong sign of real intent |
| Clarity of need | 25% | People who can describe their problem clearly usually know what they want |
| Company fit | 20% | Matters, but a slightly imperfect fit can still be a great lead |

The scores combine into a total out of 100: 80 and above is hot, 50 to 79 is warm, and anything below 50 is cold.

The AI also writes one or two sentences explaining its score so the  rep can see why a lead was flagged 
instead of trusting a number.

## How I built it

I built the whole thing from scratch in a self-hosted n8n instance, one piece at a time, and tested after each step.

I started with the trigger, because nothing else matters if the data never arrives. Once the webhook was receiving form 
submissions, I added the AI agent and its structured output so the score always comes back in the same shape. 
Then the routing,  the emails, and the spreadsheet. I finished by booking a meeting through the calendar link myself, 
to confirm that a lead's journey works from form to booked call.

**Tools used:** n8n, OpenAI, Google Forms, Google Apps Script, Google Sheets, Gmail, ngrok 
(to expose my local n8n during development).

## What went wrong along the way

Most of the real work was not the AI. It was getting the pieces to talk to each other. A few problems are worth sharing 
because they are the ones that would catch anyone building this for a client:

- **Test and production webhooks are different.** My form was sending data to the test address, which only works 
while the editor is open and listening. Switching to the production address fixed it.
- **"Completed" does not always mean it worked.** Google Apps Script reported a successful run even thogh n8n was not receiving 
the data. I firured that my ngrok had stopp running and when I started it, it worked
- **The AI's output has to match the format you ask for.** A mismatch between my prompt and the output schema made the 
agent fail until I aligned them .
- **Data does not always travel with you.** After the AI step, the original form answers were no longer on the item, 
so my emails went out with blank names and companies. I fixed it by pulling those details straight from the webhook node.
- **Connections expire.** At one point the Gmail connection stopped working and I had to reconnect it.

## What I tested

I ran the system end to end with sample submissions designed to land in different tiers. 
The routing, both emails and the spreadsheet row all worked. I also booked a real meeting through the calendar 
link and received the reminders.


## What I would improve next

- Move from a local setup and ngrok to a hosted n8n instance with a permanent address, so it runs without my laptop being on.
- Add a backup path and an error trigger for when the AI provider is down or out of quota, so no lead is ever dropped.
- Send hot-lead alerts to Slack as well as email.


## Who this helps

Any business that gets leads through a form: agencies, consultancies, SaaS companies, service businesses e.t.c. 
The details of the rubric change from one company to the next, but the problem is the same everywhere, and so is the fix.



---
