# Didibot 🤖 — AI-Powered Philadelphia City Guide Chatbot

Didibot is a conversational AI chatbot that acts as a personal city guide for Philadelphia. Ask it anything about restaurants, nightlife, rooftop bars, stadiums, shopping, or FIFA World Cup 2026 events — and it returns instant, accurate recommendations powered by a custom RAG (Retrieval-Augmented Generation) pipeline.

And now it **learns what you like**: Didibot keeps a small taste profile for each user, so the more you chat, the better its picks get.

**Try it live:**
- **WhatsApp:** [message Didibot](https://wa.me/14127752442)
- **Telegram:** [@didilelass_bot](https://t.me/didilelass_bot)
- **Instagram:** [@askdidibot](https://www.instagram.com/askdidibot/)

---

## What It Does

Users chat with Didibot on WhatsApp, Telegram, or the web. They send a message like:

> *"Where's a good rooftop bar for a date night?"*
> *"Best spot to watch the World Cup in Philly?"*
> *"I want upscale Italian near Rittenhouse."*

Didibot searches its knowledge base, finds the most relevant venues, and returns a conversational, human-like recommendation — instantly, with photos when it has them.

---

## What's New ✨

### Personalized recommendations
Didibot now remembers each user's tastes. Tell it once that you love hip hop and Northern Liberties, and the next time you ask *"where should I go Friday?"* it already knows your vibe.

- After every chat, **Claude Haiku 5.5** reads the exchange and updates a weighted taste profile: categories, neighborhoods, music, cuisines, teams, vibe and budget.
- Interests you keep coming back to grow stronger; old ones **fade by 10% each conversation**, so the profile follows your tastes as they change.
- Before answering, the main model reads the profile and uses it to choose and order suggestions — without overriding what you're asking for right now.
- The update runs in the background, so replies are just as fast.

### Privacy first
- Only **interests** are saved — never the conversation, names, contact details or sensitive topics.
- New users are told once that Didibot remembers their preferences.
- Send **`FORGET`** anytime to delete your profile.

### WhatsApp, done right
- Replies on WhatsApp are kept short (top 3–5 picks, one line each) so they fit in a single message, with a safe split if an answer ever runs long.
- Each user has their own conversation memory on every channel.

---

## How It Works

Didibot is built on a **RAG (Retrieval-Augmented Generation)** architecture, with a personalization layer on top:

```
User Message (WhatsApp / Telegram / Web)
        ↓
   n8n Workflow → Normalize (one format for every channel)
        ↓
  Load the user's taste profile        ← n8n Data Table
        ↓
  AI Agent                             ← Anthropic Claude Sonnet 4.6
   ├─ searches the knowledge base      ← OpenAI embeddings + Qdrant
   └─ uses the taste profile to rank picks
        ↓
  Channel formatter (text, photos, length rules)
        ↓
  Reply sent to the user
        ↓ (in the background)
  Update the taste profile             ← Anthropic Claude Haiku 5.5
```

---

## Tech Stack

| Component | Tool |
|---|---|
| Workflow Automation | [n8n](https://n8n.io) (self-hosted) |
| Chat Model | Anthropic **Claude Sonnet 4.6** |
| Taste-Profile Model | Anthropic **Claude Haiku 5.5** |
| Embeddings | OpenAI `text-embedding-3-small` |
| Vector Database | [Qdrant](https://qdrant.tech) (cloud) |
| User Profiles | n8n Data Tables |
| Chat Interfaces | WhatsApp (via Twilio), Telegram Bot API, web chat |
| Media Hosting | Cloudinary |
| Knowledge Base | Custom Markdown files |
| Hosting | Hostinger VPS |

### Where the machine learning is
- **Large language models** (Claude Sonnet 4.6 and Claude Haiku 5.5) understand questions, write answers and build the taste profiles.
- **Embeddings** turn every venue and question into a "meaning fingerprint", so *"chill date spot"* can find a rooftop bar even with no matching words.
- **Vector search** (nearest-neighbor search in Qdrant) finds the closest matches by meaning.
- **Next step:** training a recommendation model on what people actually pick (see the roadmap).

---

## Architecture Overview

Didibot runs on three workflows in n8n:

### 1. Ingestion Workflow (Knowledge Base → Qdrant)
This workflow is triggered manually whenever the knowledge base is updated. It:

1. Accepts file uploads (Markdown, PDF, images, video) via an n8n form
2. Routes each file by MIME type through a **Switch node**
3. For **Markdown / PDF files**: reads the content, cleans the text, and passes it directly to Qdrant
4. For **images and videos**: uploads to Cloudinary, generates a description via an AI Agent, and stores the URL + description in Qdrant
5. All content is chunked, embedded via OpenAI, and stored as vectors in the `didibot` Qdrant collection

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

### 2. Chat Workflow (WhatsApp / Telegram / Web → Response)
Triggered every time a user sends a message. It:

1. Receives the message from a **Twilio (WhatsApp)**, **Telegram**, or **web chat** trigger
2. Normalizes it into one format and loads the user's taste profile
3. Lets the **AI Agent** search Qdrant for the most relevant knowledge chunks
4. Generates a structured JSON reply (text, photos, videos, follow-up question)
5. Formats it for the channel — photo albums on Telegram, short messages on WhatsApp — and only sends photo links that really came from the knowledge base

### 3. Taste-Profile Workflow (background)
Called after every reply. **Claude Haiku 5.5** updates the user's interests and the profile is saved — or deleted when the user sends `FORGET`.

---

## Knowledge Base

The knowledge base is a set of structured Markdown entries covering:

- 🍽️ Top Restaurants (Zahav, Vernick, Kalaya, Suraya, Laser Wolf, Steak 48, Barclay Prime, Almyra)
- 🎉 Nightclubs & Late-Night (NOTO, Roar, Mr. Ivy, The Barbary, Voyeur, Frame, Bleu Martini, Silk City, and more)
- 🌆 Rooftop Bars (SkyHigh, El Techo, Assembly, Attico, Stratus, Loch Bar)
- 🍸 Happy Hours (Winston on the Water, Graffiti Bar, TIME, Loch Bar, Margolis)
- 🏟️ Sports Venues (Lincoln Financial Field, Citizens Bank Park, and the arena for the Sixers and Flyers)
- ⚽ Sport Activities (Phield House indoor soccer, Philly Run Tribe)
- 🛍️ Shopping (King of Prussia Mall, Philadelphia Premium Outlets)
- ⛸️ Outdoor & Seasonal (Independence Blue Cross RiverRink)
- ⚽ FIFA World Cup 2026 (Philadelphia Stadium, Fan Festival at Lemon Hill, Stateside Live!, watch party venues)

Each venue entry includes name, neighborhood, address, description, atmosphere, hours, and key tags — optimized for semantic search and retrieval.

---

## Roadmap

- [x] WhatsApp support (via Twilio)
- [x] Personalized recommendations with per-user taste profiles
- [x] `FORGET` command to reset your profile
- [ ] **Live events feed** — concerts, comedy, theater and pro team home games, refreshed every Monday and Thursday, with questions like *"tonight"* or *"this weekend"* filtered by date
- [ ] **Google-verified places** — ratings, hours and open/closed status, plus discovery of new top-rated spots
- [ ] **Running in Philly** 🏃 — trails, loops and run clubs, plus races and group runs
- [ ] **Weekly "Philly Weekend" message** — opt in with `WEEKEND` for 6 picks every Thursday: a game, a show, a new spot, a deal, a hidden gem and a run
- [ ] **Quick-reply menu** — *Tonight*, *This weekend*, *Near me*
- [ ] **Host concierge mode** — scan a QR code at a rental and get that property's info plus the city guide
- [ ] **Trained recommendation model** — learn from what people actually pick

---

## Author

Built by **Idriss** — data & analytics professional, Philadelphia resident, and AI builder.

Connect on [LinkedIn](https://www.linkedin.com/in/idriss-dem/) | Try the bot: [WhatsApp](https://wa.me/14127752442) · [Telegram](https://t.me/didilelass_bot) | Follow: [@askdidibot](https://www.instagram.com/askdidibot/)

---

*Built with n8n · Claude · Qdrant · OpenAI · Twilio · Telegram*
