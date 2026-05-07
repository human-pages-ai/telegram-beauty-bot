# Telegram Beauty Bot

You are an autonomous agent building a Telegram bot that helps women find hair salons, nail salons, eyebrow studios, and other beauty services near them.

## Goal
Build and deploy a Telegram bot (using the existing bot token in credentials.env) that:
1. Lets users choose a beauty service type (nails, hair, eyebrows, lashes, waxing, massage, makeup)
2. Takes their location (text address, city, or coordinates)
3. Asks their budget preference
4. Searches for nearby salons using OSM Overpass API (free) + Google Places if API key available
5. Ranks results by rating, distance, and budget match
6. Shows top 5 results with inline keyboard buttons for selection
7. Shows salon details with phone, directions, hours, and a "Book" action (deep link to Google Maps or phone call)
8. Runs as a long-polling bot (no webhook needed for v1)

## CRITICAL: Reuse Existing Code

There is a COMPLETE working WhatsApp version of this exact bot. Port it to Telegram, don't rebuild from scratch.

### Previous WhatsApp bot (port this):
- **Conversation state machine**: `/home/ubuntu/projects/agent-factory/identities/1778157722191-create-a-whatsapp-bot-that-helps-women-f/workspace/src/bot.js`
  - States: SERVICE_TYPE → LOCATION → BUDGET → SEARCHING → RESULTS → BOOKING
  - Session management with reset keywords
  - Full error handling

- **Salon search + ranking**: `/home/ubuntu/projects/agent-factory/identities/1778157722191-create-a-whatsapp-bot-that-helps-women-f/workspace/src/salonSearch.js`
  - OSM Overpass search with service-specific tags
  - Google Places fallback
  - Haversine distance calculation
  - Ranking algorithm: 35% adjusted rating + 35% distance + 20% budget + 10% review confidence
  - Review confidence uses log-scale normalization

- **Message formatting**: `/home/ubuntu/projects/agent-factory/identities/1778157722191-create-a-whatsapp-bot-that-helps-women-f/workspace/src/messageFormatter.js`
  - Service emojis, greeting, prompts, result cards, salon detail view
  - Service type parsing (text + number input)
  - Budget level parsing

- **Session store**: `/home/ubuntu/projects/agent-factory/identities/1778157722191-create-a-whatsapp-bot-that-helps-women-f/workspace/src/sessionStore.js`
  - TTL-based in-memory cache

### Human-pages ecosystem (reference patterns):
- **Geocoding with caching**: `/home/ubuntu/projects/human-pages/backend/src/lib/geocode.ts` — Nominatim geocoding, but uses Prisma cache (you'll use in-memory)
- **Geo utilities**: `/home/ubuntu/projects/human-pages/backend/src/lib/geo.ts` — Haversine, bounding box
- **Telegram inline keyboards**: `/home/ubuntu/projects/examples/payroll-bot/src/telegram-approve.ts` — Inline keyboard buttons, callback query polling, message editing after response
- **Telegram notifications**: `/home/ubuntu/projects/examples/errand-bot/src/notify.ts` — HTML parse mode formatting
- **API client with retries**: `/home/ubuntu/projects/examples/errand-bot/src/api.ts` — Exponential backoff
- **Bot conversation lifecycle**: `/home/ubuntu/projects/examples/marketing-bot/src/bot.ts` — Multi-phase state machine

## Tech Stack
- **Runtime**: Node.js with TypeScript (use tsx for dev)
- **Telegram library**: `node-telegram-bot-api` or raw Telegram Bot API (fetch-based, like in payroll-bot)
- **Search**: OSM Overpass (free) + Google Places (if GOOGLE_PLACES_API_KEY set)
- **Session**: In-memory Map with TTL (port from sessionStore.js)
- **No database needed** for v1

## Telegram-Specific Improvements Over WhatsApp Version
- Use **inline keyboard buttons** instead of text numbers for service selection and results
- Use **callback queries** for navigation (back, new search)
- Use **location sharing** — Telegram supports sending GPS location natively
- Use **HTML parse mode** for rich formatting (bold, links)
- Add **/start** and **/help** commands
- Show a "Share Location" button alongside text input option

## Rules
- Execute the assigned task fully. Do not work on other tasks.
- Always update board.json and journal.md when done.
- Git commit your work after each task.
- If blocked, set task status to "blocked" with notes and write "NOTIFY: reason" to notifications.md.
- Source ~/.agent-factory/credentials.env for API keys (TG_BOT_TOKEN is there).
- Log spending in spending.json: {"items": [{"date": "ISO", "what": "description", "amount": 0.00, "currency": "USD"}]}
- Budget ceiling: $50. Single purchase limit: $20.
- If you need a tool that doesn't exist, build it in the tools/ directory.
- Use free tiers and open APIs first. Don't spend money without a good reason.
- Write production code, not prototypes.
- The bot should run with: `npx tsx src/index.ts` from the workspace directory.
