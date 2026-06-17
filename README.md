# Didibot 🤖 — AI-Powered Philadelphia City Guide Chatbot

Didibot is a conversational AI chatbot that acts as a personal city guide for Philadelphia. Ask it anything about restaurants, nightlife, rooftop bars, stadiums, shopping, or FIFA World Cup 2026 events — and it returns instant, accurate recommendations powered by a custom RAG (Retrieval-Augmented Generation) pipeline.

**Try it live on Telegram:** [@didilelass_bot](https://t.me/didilelass_bot)

---

## What It Does

Users interact with Didibot directly inside Telegram. They send a message like:

> *"Where's a good rooftop bar for a date night?"*
> *"Best spot to watch the World Cup in Philly?"*
> *"I want upscale Italian near Rittenhouse."*

Didibot searches its knowledge base, finds the most relevant venues, and returns a conversational, human-like recommendation — instantly.

---

## How It Works

Didibot is built on a **RAG (Retrieval-Augmented Generation)** architecture. Here's the full pipeline:

```
User Message (Telegram)
        ↓
   n8n Workflow
        ↓
  Query Embedding          ← OpenAI Embeddings API
  (text → vector)
        ↓
  Vector Search            ← Qdrant Vector Database
  (find relevant chunks)
        ↓
  Context + Prompt         ← Assembled by n8n
        ↓
  LLM Response             ← OpenAI GPT / Anthropic Claude
        ↓
  Reply sent to user       ← Telegram Bot API
```

---

## Tech Stack

| Component | Tool |
|---|---|
| Workflow Automation | [n8n](https://n8n.io) (self-hosted) |
| Vector Database | [Qdrant](https://qdrant.tech) (cloud) |
| Embeddings | OpenAI `text-embedding-3-small` |
| LLM | OpenAI GPT-4o / Anthropic Claude |
| Chat Interface | Telegram Bot API |
| Knowledge Base | Custom Markdown files |
| Hosting | Hostinger VPS |

---

## Architecture Overview

Didibot has two separate workflows running in n8n:

### 1. Ingestion Workflow (Knowledge Base → Qdrant)
This workflow is triggered manually whenever the knowledge base is updated. It:

1. Accepts file uploads (Markdown, PDF, images, video) via an n8n form
2. Routes each file by MIME type through a **Switch node**
3. For **Markdown / PDF files**: reads the binary content, cleans the text, and passes it directly to Qdrant
4. For **images**: uploads to Cloudinary, generates a description via an AI Agent, and stores the URL + description in Qdrant
5. For **videos**: same as images — Cloudinary upload → AI description → Qdrant
6. All content is chunked, embedded via OpenAI, and stored as vectors in the `didibot` Qdrant collection

```
Form Upload
     ↓
  Explode (split files)
     ↓
  Switch (by MIME type)
     ├── image/png  → Cloudinary → AI Agent → Qdrant
     ├── image/jpeg → Cloudinary → AI Agent → Qdrant
     ├── video/mp4  → Cloudinary → AI Agent → Qdrant
     ├── text/markdown → Clean text → Qdrant
     └── application/pdf → Extract text → Qdrant
```

### 2. Chat Workflow (Telegram → Response)
This workflow is triggered every time a user sends a message to the Telegram bot. It:

1. Receives the message via a **Telegram Trigger** node
2. Embeds the user's query using OpenAI
3. Runs a **vector similarity search** in Qdrant to find the most relevant knowledge chunks
4. Passes the retrieved context + the user's original message to the LLM
5. Sends the LLM's response back to the user via Telegram

```
Telegram Message
       ↓
  Embed query (OpenAI)
       ↓
  Search Qdrant (top-k chunks)
       ↓
  Build prompt (context + query)
       ↓
  LLM generates response
       ↓
  Send reply via Telegram
```

---

## Knowledge Base

The knowledge base is a structured Markdown file covering:

- 🍽️ Top Restaurants (Zahav, Vernick, Kalaya, Suraya, Laser Wolf, Steak 48, Barclay Prime, Almyra)
- 🎉 Nightclubs & Late-Night (NOTO, Roar, Mr. Ivy, The Barbary, Voyeur, Frame, Bleu Martini, Silk City, and more)
- 🌆 Rooftop Bars (SkyHigh, El Techo, Assembly, Attico, Stratus, Loch Bar)
- 🏟️ Sports Venues (Lincoln Financial Field, Citizens Bank Park, Wells Fargo Center)
- 🛍️ Shopping (King of Prussia Mall, Philadelphia Premium Outlets)
- ⛸️ Outdoor & Seasonal (Independence Blue Cross RiverRink)
- ⚽ FIFA World Cup 2026 (Philadelphia Stadium, Fan Festival at Lemon Hill, Stateside Live!, watch party venues)

Each venue entry includes name, neighborhood, address, description, atmosphere, hours, and key tags — optimized for semantic search and retrieval.

---

## How Telegram Is Connected

The Telegram bot is connected to n8n via the **Telegram Bot API**:

1. A bot is created through [@BotFather](https://t.me/BotFather) on Telegram, which generates a **Bot Token**
2. In n8n, a **Telegram Trigger** node listens for incoming messages using that token (via webhook)
3. When a user messages the bot, Telegram sends the message payload to the n8n webhook URL
4. n8n processes the message through the chat workflow and calls the **Telegram node** to send the reply back
5. The user sees the response directly in their Telegram chat — no app download or sign-up required

```
User sends message on Telegram
          ↓
Telegram API → POST to n8n Webhook URL
          ↓
n8n processes (embed → search → LLM)
          ↓
n8n Telegram node → sends reply
          ↓
User receives response in Telegram chat
```

---

## Project Status

Didibot is actively maintained and in continuous development. Planned improvements include:

- [ ] Expanded knowledge base (more neighborhoods, events, seasonal content)
- [ ] Multi-city support
- [ ] Web interface beyond Telegram
- [ ] User preference memory (personalized recommendations)
- [ ] Integration with real-time data (hours, reservations, events)

---

## Author

Built by **Idriss** — data & analytics professional, Philadelphia resident, and AI builder.

Connect on https://www.linkedin.com/in/idriss-dem/  | Try the bot: [@didilelass_bot](https://t.me/didilelass_bot)

---

*Built with n8n · Qdrant · OpenAI · Telegram*

