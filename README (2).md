# Telegram Customer Support Bot

An automated customer support system built with n8n that integrates with the Telegram Bot API. The bot classifies incoming user messages using rule-based intent detection (keyword and regex matching) and routes each query to the appropriate response — order status, FAQs, billing, or escalation to a human agent. All interactions are logged to Google Sheets for tracking and reporting.

## Overview

This project was built to demonstrate how a lightweight, no-code automation platform can replace a traditional backend for handling repetitive customer support workflows. The bot receives messages via Telegram webhooks, determines user intent through a series of conditional checks, and responds automatically without requiring any custom server code.

## Features

- Real-time message handling via Telegram webhooks
- Rule-based intent classification using string matching and regular expressions
- Order ID extraction and validation via regex
- Multi-branch conditional routing (FAQ, order inquiry, billing, human escalation, general help)
- Automatic logging of every interaction to Google Sheets
- Fully built and tested using a self-hosted n8n instance

## Tech Stack

| Component | Technology |
|---|---|
| Workflow automation | n8n (self-hosted) |
| Messaging platform | Telegram Bot API |
| Local tunneling | ngrok |
| Data logging | Google Sheets API |
| Routing logic | n8n IF nodes (string and regex conditions) |

## How It Works

1. **Telegram Trigger** receives incoming messages through a webhook.
2. **Extract User Data** (Set node) parses the raw Telegram payload into clean fields: user ID, name, username, message text, chat ID, and timestamp.
3. The cleaned message is evaluated against several parallel conditions to determine intent:
   - FAQ requests
   - Order status inquiries
   - Requests for human support
   - Billing-related questions
   - General help/greeting messages
4. A secondary regex check validates whether the message contains a properly formatted order ID.
5. Each intent branch triggers a dedicated response through a Telegram "Send Message" node.
6. Every completed interaction is appended as a new row in a connected Google Sheet.

## Setup Instructions

### Prerequisites
- n8n installed locally or via Docker
- A Telegram bot token, created via @BotFather
- ngrok, for exposing the local n8n instance during development
- A Google account with Sheets API access

### Installation

1. Import `customer_support.json` into your n8n instance via **Workflows → Import from File**.
2. Create a Telegram bot using @BotFather and copy the generated API token.
3. In n8n, create a Telegram API credential using this token and assign it to the Telegram Trigger node and all Send Message nodes.
4. Set up a Google Sheets OAuth2 credential and connect it to the logging nodes.
5. Start ngrok to expose the local instance:
