# 🤖 Telegram Customer Support Bot (n8n Automation)

An automated customer-support chatbot built on **n8n**, integrated with the **Telegram Bot API**. The bot understands user intent through pattern matching (keywords + regex) and routes each message to the correct automated response — order status, FAQs, billing queries, or escalation to human support. All interactions are logged to **Google Sheets** for tracking and analytics.

---

## 📌 Overview

Businesses often need a lightweight, no-code way to handle repetitive customer queries without building a full backend. This project simulates that use case: a Telegram bot that can independently classify and respond to common support requests, while logging every interaction for visibility.

---

## 🏗️ Architecture

```
                          ┌────────────────┐
                          │ Telegram Trigger│
                          └────────┬────────┘
                                   │
                          ┌────────▼────────┐
                          │ Extract User Data│  (Set node — cleans raw
                          └────────┬────────┘   Telegram payload)
                                   │
        ┌───────────┬─────────────┼─────────────┬──────────────┐
        ▼           ▼             ▼             ▼              ▼
  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌─────────────┐ ┌──────────┐
  │ is FAQ?  │ │Is Order  │ │Need Human │ │Is Billing   │ │Is Help   │
  │          │ │Inquiry?  │ │Help?      │ │Query?       │ │Required? │
  └────┬─────┘ └────┬─────┘ └─────┬─────┘ └──────┬──────┘ └────┬─────┘
       │            │             │              │             │
       ▼            ▼             ▼              ▼             ▼
   Send FAQ    Has Order ID?  Escalate to     Billing        Welcome /
   response      (regex)      human agent     response       Help menu
                    │
             ┌──────┴──────┐
             ▼             ▼
        Order found   Ask for valid
        response      Order ID
                                   │
                                   ▼
                          ┌────────────────┐
                          │  Google Sheets   │
                          │ (interaction log)│
                          └────────────────┘
```

---

## ✨ Features

- **Real-time message handling** via Telegram webhooks (no polling)
- **Intent classification** using keyword matching (`contains`) and regex patterns
- **Order ID detection** via regex to validate and extract order numbers from free text
- **Multi-branch conditional routing** using n8n `IF` nodes (FAQ, order status, billing, human escalation, general help)
- **Interaction logging** to Google Sheets for every conversation (user ID, name, message, timestamp)
- Built and tested with a **local n8n instance** exposed publicly via **ngrok**

---

## 🛠️ Tech Stack

| Layer                  | Tool / Service               |
|-------------------------|-------------------------------|
| Workflow automation     | [n8n](https://n8n.io) (self-hosted, local) |
| Messaging platform      | Telegram Bot API              |
| Tunneling (local dev)   | ngrok                         |
| Data logging            | Google Sheets API             |
| Routing logic           | n8n `IF` nodes (regex + string conditions) |

---

## ⚙️ How It Works

1. **Telegram Trigger** — listens for incoming messages via webhook.
2. **Extract User Data** — a `Set` node that pulls clean fields (`user_id`, `user_name`, `username`, `message_text`, `chat_id`, `timestamp`) out of Telegram's raw payload.
3. **Intent routing (`IF` nodes)** — the cleaned message text is checked against multiple conditions in parallel:
   - `is FAQ` — matches `/faq`, `faq`, `question`
   - `is order inquiry` — matches order/tracking-related keywords or `/status`
   - `Need Human help?` — matches `/contact`, `human`, `agent`
   - `is billing query` — matches `/billing`, `invoice`, etc.
   - `is help required` — fallback for `/start` / `/help` / general greetings
4. **Has Order ID** — a secondary regex check (`[A-Za-z]{2,5}\d{4,8}`) that detects whether the message contains a valid order ID format, and responds differently if one is or isn't found.
5. **Send a text message (×N)** — each branch has its own dedicated Telegram response node.
6. **Append row in sheet** — every response is logged to Google Sheets for record-keeping.

---

## 🚀 Setup Instructions

### Prerequisites
- [n8n](https://docs.n8n.io/hosting/installation/npm/) installed locally (or via Docker)
- A Telegram bot token (create one via [@BotFather](https://t.me/BotFather))
- [ngrok](https://ngrok.com/) account (for exposing localhost during local development)
- A Google account with Sheets API access (for logging)

### Steps
1. Clone this repository and import `customer_support.json` into your n8n instance:
   `n8n editor → Workflows → Import from File`
2. Create a Telegram bot via **@BotFather** and copy the API token.
3. In n8n, create a **Telegram API credential** using that token, and assign it to both the `Telegram Trigger` node and all `Send a text message` nodes.
4. Set up a **Google Sheets OAuth2 credential** and connect it to the `Append row in sheet` nodes.
5. Start ngrok to expose your local n8n instance:
   ```bash
   ngrok http 5678
   ```
6. Set the `WEBHOOK_URL` environment variable to your ngrok HTTPS URL, then start n8n:
   ```bash
   set WEBHOOK_URL=https://your-ngrok-url.ngrok-free.dev/
   n8n start
   ```
7. Activate/Publish the workflow in n8n.
8. Message your bot on Telegram to test it — try `/help`, `/faq`, `/contact`, or an order ID like `ORD12345`.

---

## 🖼️ Screenshots

*(Add your workflow screenshot and a Telegram conversation screenshot here)*

```
![Workflow](./screenshots/workflow.png)
![Bot conversation](./screenshots/chat-demo.png)
```

---

## 📚 What I Learned

- Setting up and debugging **webhooks** for a locally hosted automation tool using ngrok
- Writing and troubleshooting **regex patterns** for intent/entity detection
- Understanding the difference between `AND` / `OR` condition logic and how regex alternation (`|`) interacts with anchors (`^`, `$`)
- Handling real-world API errors (invalid tokens, webhook conflicts, message length limits, parse-mode formatting issues)
- Structuring a multi-branch conversational flow without writing traditional backend code

---

## 🔮 Future Improvements

- Replace fixed keyword/regex routing with an AI-based intent classifier
- Add persistent conversation memory for multi-turn context
- Deploy on a cloud server with a permanent domain (replacing ngrok for production use)
- Add inline keyboard buttons for a guided menu experience

---

## 📄 License

This project is for educational/portfolio purposes.
