# Glowie: AI Client Chatbot for Skincare Ecommerce

An AI agent chatbot built for a fictional skincare ecommerce store, that handles a full customer
conversation from first message to booked consultation, without a human needing to step in.

## The Problem

Small ecommerce stores lose sales when customers have questions and nobody replies fast enough, especially outside 
business hours. A skincare store also has a specific problem on top of that: customers need personalized product 
recommendations based on skin type, tone, and concern, which a generic FAQ bot cannot give. This automation solves both 
by using an AI agent that can actually reason and provide responses based the customer's needs, not just matching keywords.

## How It Works

```
Chat Trigger → AI Agent (system prompt + tools) → Response
                     ↓
      check consultation slot (Code Tool)
      Google Calendar (booking tool)
      Google Sheets (lead capture tool)
```

| Component | Purpose |
|---|---|
| **AI Agent** | Core of the workflow, holds the system prompt defining identity, tone, intake flow, and recommendation logic |
| **Product Catalog** | Real Nigeria-available skincare products (face wash, serum, toner, moisturizer, exfoliators, sunscreen, face cream, body cream, body wash) tagged by skin type, skin tone, and concern |
| **check consultation (Code Tool)** | Validates whether a requested day/time falls within bookable consultation hours |
| **Google Calendar** | Books confirmed consultation slots directly from the conversation |
| **Google Sheets** | Logs customer name, email, and phone number captured during the chat |

## Tools Used

- n8n - AI Agent node and workflow orchestration
- Google Calendar API - consultation booking
- Google Sheets API - lead capture / CRM-style record keeping
- Custom Code node - business rule validation

## My Setup Process

I started by choosing a B2C skincare ecommerce store as the use case over other niches I considered (real estate, fashion, 
fitness supplements), since skincare gives a natural reason for the AI to ask diagnostic questions and make
personalized recommendations rather than just answering FAQs.

I built out a product catalog using real, Nigeria-available skincare products across nine categories, and tagged each one by 
skin type, skin tone, and concern so the AI agent would have structured data to reason over instead of guessing.

I then wrote a detailed system prompt covering the agent's identity and tone, the intake flow for gathering skin information,
the recommendation logic for matching products to a customer's profile, and escalation rules for when the agent should hand 
off to a human instead of continuing.

Since the business only takes consultation bookings on specific days and hours (Tuesday, Thursday, and Saturday, 12 to 3 PM
WAT), I built a custom Code Tool called `check consultation ` so date and day-of-week validation happens in code rather 
than being left to the model to calculate, which is more reliable.

I connected a Google Calendar node as a tool so the AI agent can create a booking directly, using `$fromAI` to pass the 
confirmed start time from the conversation and calculating the 30 minute end time in an expression.

Finally, I connected a Google Sheets node (Append or update row) as a tool so customer name, email, and phone number are 
logged automatically the moment they're captured in conversation, giving the store a simple CRM-style record without any
manual data entry.

## Lessons Learned

- **LLMs shouldn't do date and time math.** Letting the agent calculate day-of-week and validate consultation slots on its own 
was unreliable. A small JavaScript Code Tool now checks the date and confirms it falls in the allowed window, so the model
just calls the tool instead of doing arithmetic itself.

- **Escalation should lead somewhere.** Instead of a dead-end "let me get a human," every unanswerable question routes into
booking a free consultation. That turns a limitation into a conversion point instead of a frustration.

- **Enforce constraints at the tool level, not just the prompt.** The prompt states the booking window, but a persistent
user can argue with a prompt. The Calendar tool itself is scoped so it cannot create events outside that window, 
no matter what the model tries to pass it.

- **Small syntax slips cause confusing errors.** A single capitalization typo (`$FromAI` instead of `$fromAI`) produced 
an error about unbalanced parentheses that had nothing to do with the real issue.

## Status

Full workflow (system prompt, product catalog, date validation tool, calendar booking, and Sheets lead capture) built 
and working end to end.

## Potential Upgrades

- Real inventory/stock sync so recommendations reflect live availability
- WhatsApp integration for customers who prefer messaging over web chat
- Post-consultation follow-up sequence for customers who booked but didn't purchase

<img width="826" height="293" alt="image" src="https://github.com/user-attachments/assets/5af97841-38b7-48dd-9932-876c4e36ef3b" />
<img width="737" height="379" alt="image" src="https://github.com/user-attachments/assets/6e15e8f2-ca50-4e8b-86e8-bff83c9c5092" />
<img width="913" height="313" alt="image" src="https://github.com/user-attachments/assets/22e6421a-c46e-4572-84ae-31f9b0010c80" />
<img width="643" height="313" alt="image" src="https://github.com/user-attachments/assets/1d94d831-eee3-480c-a928-6f5d6435cada" />
