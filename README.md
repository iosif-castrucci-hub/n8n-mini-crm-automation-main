# Mini CRM Automation — Lead Capture & Follow-up Reminder

![n8n Workflow Overview](screenshot/screenshot_Demo.png)

A no-code automation project built with **n8n**, **Google Sheets**, **Telegram**, and **Gmail**.

This project demonstrates a simple but practical CRM automation system for small businesses, freelancers, and service providers.
It captures incoming leads, stores them in a Google Sheets-based CRM, sends real-time notifications, delivers automatic confirmation emails, and reminds the business owner when a lead has not been followed up.

---

## Project Overview

The goal of this project is to automate a basic lead management process:

1. A new lead submits a contact form or sends data to a webhook.
2. n8n receives the lead data.
3. The lead is saved into a Google Sheets CRM.
4. The business owner receives an instant Telegram notification.
5. The lead receives an automatic confirmation email.
6. A scheduled workflow checks for leads still marked as `Nuovo`.
7. If a lead has not been followed up, n8n sends a Telegram reminder.
8. After the reminder is sent, the CRM is updated to prevent duplicate reminders.

This project is designed as a demo/prototype of how small businesses can automate repetitive lead follow-up tasks without building a full custom CRM from scratch.

---

## Tech Stack

* **n8n Cloud** — workflow automation platform
* **Google Sheets** — lightweight CRM database
* **Telegram Bot API** — real-time notifications
* **Gmail** — automatic confirmation emails
* **Webhook Trigger** — receives new lead data
* **Schedule Trigger** — runs automatic checks periodically

---

## Main Features

### Lead Capture

The first workflow block receives a new lead through an n8n Webhook and saves the information inside Google Sheets.

Captured fields include:

* Lead ID
* Date
* Name
* Email
* Phone
* Message
* Source
* Status
* Reminder
* Notes

---

### CRM Storage with Google Sheets

Google Sheets is used as a simple CRM database.

Each new lead is automatically added as a new row with the default status:

```txt
Stato = Nuovo
Reminder = No
```

This allows the business owner to manually update the lead status after contacting the customer.

---

### Real-Time Telegram Notification

When a new lead is received, n8n sends an automatic Telegram notification to the business owner.

Example notification:

```txt
🔥 New lead received

Name: Mario Rossi
Email: mario@example.com
Phone: 3331234567
Message: I would like more information
Source: Website
```

This helps the business owner react quickly to new opportunities.

---

### Automatic Confirmation Email

After the lead is saved, n8n sends an automatic confirmation email to the customer.

Example email:

```txt
Hi Mario,

we have received your request and we will contact you as soon as possible.

Thank you,
CRM Solutions
```

This improves the customer experience and confirms that the request was received successfully.

---

### Automatic Follow-up Reminder

The second workflow block runs on a schedule and checks the Google Sheets CRM.

It looks for leads where:

```txt
Stato = Nuovo
Reminder = No
```

If a lead still needs attention, n8n sends a Telegram follow-up reminder.

Example reminder:

```txt
⏰ Follow-up reminder

Lead still marked as New.

Name: Lorenzo Quaglia
Email: example@email.com
Phone: 3331234567
Message: I would like information about payment methods
Source: Manual test
Created At: 28/05/2026 16:53

Please contact this lead as soon as possible.
```

After sending the reminder, n8n updates the Google Sheets row:

```txt
Reminder = Yes
```

This prevents duplicate reminders for the same lead.

---

## Workflow Structure

The demo is organized into two main automation blocks inside n8n.

---

## Block 1 — Lead Capture

This block starts when a new lead is received.

### Nodes

```txt
01 Webhook - Receive New Lead
02 Google Sheets - Save Lead to CRM
03 Telegram - Notify Business Owner
04 Gmail - Send Confirmation Email
```

### Flow

```txt
Webhook
↓
Google Sheets
↓
Telegram
↓
Gmail
```

### Purpose

This block handles the initial lead acquisition process:

* receives the lead data
* saves it in the CRM
* notifies the business owner
* confirms the request by email

---

## Block 2 — Automatic Reminder

This block runs on a schedule and checks if there are leads that still need follow-up.

### Nodes

```txt
05 Schedule - Daily Lead Check
06 Google Sheets - Read CRM Leads
07 Filter - Find New Leads
08 Telegram - Send Follow-up Reminder
09 Google Sheets - Mark Reminder as Sent
```

### Flow

```txt
Schedule Trigger
↓
Google Sheets
↓
Filter
↓
Telegram
↓
Google Sheets Update
```

### Purpose

This block prevents leads from being forgotten by:

* reading all CRM leads
* filtering leads still marked as new
* sending a reminder to the business owner
* updating the reminder status

---

## Google Sheets Structure

The Google Sheet contains the following columns:

```txt
Lead ID
Data
Nome
Email
Telefono
Messaggio
Fonte
Stato
Reminder
Note
```

### Example Row

| Lead ID | Data             | Nome            | Email                                         | Telefono   | Messaggio                | Fonte       | Stato | Reminder | Note |
| ------- | ---------------- | --------------- | --------------------------------------------- | ---------- | ------------------------ | ----------- | ----- | -------- | ---- |
| 7       | 28/05/2026 16:53 | Lorenzo Quaglia | [example@email.com](mailto:example@email.com) | 3331234567 | I would like information | Manual test | Nuovo | No       |      |

---

## Status Logic

The CRM uses a simple lead status system.

### Lead Status

```txt
Nuovo
Contattato
```

Optional future statuses:

```txt
In trattativa
Vinto
Perso
```

### Reminder Status

```txt
No
Yes
```

The reminder workflow only sends alerts when:

```txt
Stato = Nuovo
Reminder = No
```

After the reminder is sent, the value becomes:

```txt
Reminder = Yes
```

---

## Webhook Payload Example

The webhook expects a JSON payload like this:

```json
{
  "nome": "Mario Rossi",
  "email": "mario@example.com",
  "telefono": "3331234567",
  "messaggio": "I would like more information",
  "fonte": "Website"
}
```

---

## Example cURL Test

The webhook can be tested using a `POST` request.

```bash
curl -X POST "https://your-n8n-domain.app.n8n.cloud/webhook-test/new-lead" \
-H "Content-Type: application/json" \
-d '{
  "nome": "Mario Rossi",
  "email": "mario@example.com",
  "telefono": "3331234567",
  "messaggio": "I would like more information",
  "fonte": "Manual test"
}'
```

---

## n8n Expressions Used

### Save Lead to Google Sheets

```txt
Lead ID: {{ $execution.id }}
Data: {{ $now.format('dd/MM/yyyy HH:mm') }}
Nome: {{ $('Webhook').item.json.body.nome }}
Email: {{ $('Webhook').item.json.body.email }}
Telefono: {{ $('Webhook').item.json.body.telefono }}
Messaggio: {{ $('Webhook').item.json.body.messaggio }}
Fonte: {{ $('Webhook').item.json.body.fonte }}
Stato: Nuovo
Reminder: No
```

---

### Telegram New Lead Notification

```txt
🔥 New lead received

Name: {{ $('Webhook').item.json.body.nome }}
Email: {{ $('Webhook').item.json.body.email }}
Phone: {{ $('Webhook').item.json.body.telefono }}
Message: {{ $('Webhook').item.json.body.messaggio }}
Source: {{ $('Webhook').item.json.body.fonte }}
```

---

### Telegram Follow-up Reminder

```txt
⏰ Follow-up reminder

Lead still marked as New.

Name: {{ $json.Nome }}
Email: {{ $json.Email }}
Phone: {{ $json.Telefono }}
Message: {{ $json.Messaggio }}
Source: {{ $json.Fonte }}
Created At: {{ $json.Data }}

Please contact this lead as soon as possible.
```

---

### Update Reminder Status

The final Google Sheets node updates the same row using:

```txt
row_number: {{ $('07 Filter - Find New Leads').item.json.row_number }}
Reminder: Yes
```

---

## Demo Use Case

This automation can be used by:

* freelancers
* consultants
* agencies
* local businesses
* service providers
* small sales teams
* appointment-based businesses

Example use cases:

* website contact forms
* landing page lead capture
* quotation requests
* service inquiries
* customer follow-up reminders
* small CRM systems

---

## Business Value

This project solves a common problem for small businesses:

> Leads arrive, but nobody follows up quickly enough.

With this automation, every lead is:

* captured immediately
* stored in a CRM
* sent to the business owner
* acknowledged by email
* checked later for follow-up
* reminded automatically if not handled

This reduces missed opportunities and improves response time.

---

## Future Improvements

Possible future upgrades include:

* custom landing page form
* WhatsApp notifications
* lead scoring
* multiple reminder levels
* email follow-up sequences
* CRM pipeline stages
* dashboard reporting
* weekly performance summary
* AI-generated lead summaries
* Google Calendar integration
* automatic task creation
* customer segmentation by source

---

## Project Status

This project is currently a working demo built in n8n Cloud.

Current version includes:

* Webhook lead capture
* Google Sheets CRM storage
* Telegram notification
* Gmail confirmation email
* Scheduled reminder workflow
* Reminder status update

---

## Screenshots


## Author

Built by **CRM Solutions**

GitHub: `crm-solutions-lab`
Email: `crmsolution.contact@gmail.com`

---

## Disclaimer

This project is a demo automation system and should be customized before being used in production.

For real client usage, make sure to review:

* data privacy
* access permissions
* email sending limits
* API credentials
* Google Sheets permissions
* n8n execution limits
* error handling
* backup strategy

---

## License

This repository is intended for portfolio and educational purposes.
You may adapt the workflow structure for your own projects.
