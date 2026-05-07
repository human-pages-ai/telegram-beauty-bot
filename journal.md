# Telegram Beauty Bot — Journal

## 2026-05-07 — Cycle 1: Planning

### Context

Initialized the project. This bot is a Telegram port of an existing, fully-working WhatsApp beauty bot located at:
`/home/ubuntu/projects/agent-factory/identities/1778157722191-create-a-whatsapp-bot-that-helps-women-f/workspace/src/`

The WhatsApp version already has:
- A complete conversation state machine (SERVICE_TYPE → LOCATION → BUDGET → SEARCHING → RESULTS → BOOKING)
- OSM Overpass API search with Google Places fallback
- Haversine-based distance scoring with a weighted ranking algorithm (35% rating + 35% distance + 20% budget + 10% review confidence)
- TTL-based in-memory session store
- Full message formatting with emojis

**Bot token confirmed present**: `TG_BOT_TOKEN` found in `~/.agent-factory/credentials.env`.
**Google Places key**: Not present — bot will rely on OSM Overpass API only.

### Plan Summary

4 milestones, 9 tasks:

**M1 — Scaffolding** (2 tasks)
- Initialize Node.js/TypeScript project with `package.json`, `tsconfig.json`, install: `node-telegram-bot-api`, `@types/node-telegram-bot-api`, `tsx`, `typescript`, `axios`, `node-cache`
- Create `src/` directory skeleton: `index.ts`, `bot.ts`, `sessionStore.ts`, `salonSearch.ts`, `messageFormatter.ts`, `types.ts`

**M2 — Core modules** (4 tasks)
- `types.ts`: TypeScript interfaces for Session, Salon, ServiceType, BudgetLevel, BotState
- `sessionStore.ts`: Port from JS, use plain Map + TTL timers instead of NodeCache (avoids extra dep)
- `salonSearch.ts`: Port from JS to TypeScript, switch from axios to native fetch, add GPS coordinate shortcut (Telegram sends exact lat/lon — no Nominatim geocoding needed in that path)
- `messageFormatter.ts`: Port from JS, change WhatsApp *bold* to HTML `<b>bold</b>`, add inline keyboard builder functions returning `InlineKeyboardMarkup` objects

**M3 — Telegram bot engine** (3 tasks)
- `bot.ts` message handler: same state machine as WhatsApp version, plus handle `msg.location` for GPS
- `bot.ts` callback query handler: parse `service:X`, `budget:X`, `salon:X`, `action:X` callback data
- `index.ts`: entry point, polling mode, load env, wire handlers, graceful shutdown

**M4 — Testing** (3 tasks)
- Live test with real Telegram chat
- Bug fixes
- Final git commit

### Key Telegram-specific design decisions

1. **Inline keyboards replace text numbers**: Service selection uses a 2×4 button grid. Budget uses a 4-button row. Results use one button per salon. Navigation (Back, New Search) are inline buttons on the detail view.

2. **Callback data format**: `"service:nails"`, `"budget:2"`, `"salon:0"`, `"action:back"`, `"action:new"` — simple colon-separated prefix:value.

3. **Location messages**: When a user shares their GPS location via Telegram's native location button, `msg.location.latitude` and `msg.location.longitude` are used directly — bypassing the Nominatim geocoding step entirely.

4. **HTML parse mode**: All bot messages use `parse_mode: 'HTML'`. User-supplied content (salon names, addresses) must be HTML-escaped before insertion to prevent injection.

5. **Session key**: `chatId` (number) instead of phone number (string). Both are unique per-conversation identifiers.

6. **No webhook**: Long-polling only for v1 simplicity. The `node-telegram-bot-api` library handles polling internally.

## 2026-05-07T16:30:24.169Z — m1.t1
Created `package.json` with all required dependencies (node-telegram-bot-api, @types/node-telegram-bot-api, axios, node-cache, tsx, typescript) and `tsconfig.json` targeting ES2022/CommonJS with strict mode. The `.gitignore` was already present. Ran `npm install` — 198 packages installed successfully.

## 2026-05-07T16:31:30.685Z — m1.t2
Created all 6 TypeScript source files in `workspace/src/`: `types.ts` (interfaces/enums), `sessionStore.ts` (Map-based TTL cache replacing NodeCache), `salonSearch.ts` (native fetch instead of axios, GPS coord shortcut for Telegram location messages), `messageFormatter.ts` (HTML parse mode, InlineKeyboardMarkup builders for service/budget/results/detail views), `bot.ts` (full state machine with text, GPS location, and callback_query handlers), `index.ts` (polling entry point with credentials.env loader and graceful shutdown). `tsc --noEmit` passes with zero errors.

### Risks & mitigations

- **Overpass API rate limits**: Add a 1-second delay between geocoding (Nominatim) and search (Overpass). Both require respectful use-agent headers (already in WhatsApp version).
- **No Google Places key**: OSM results will have no ratings. The ranking algorithm handles `rating: null` gracefully (defaults to 3.5 baseline), so results still sort sensibly by distance.
- **Callback query staleness**: If a user presses an old inline button after their session expires, the bot will reset them to SERVICE_TYPE with a friendly message.

## 2026-05-07T16:36:51.519Z — m2.t2
Verified `src/sessionStore.ts` was already written during m1.t2 and is complete. Uses plain `Map<number, {session, timer}>` with `setTimeout` for TTL — no NodeCache dependency. Exports `getSession`, `saveSession`, `resetSession`, and `STATES` (re-exported `BotState` enum). `timer.unref()` prevents the TTL timer from blocking process exit. Task marked done.

## 2026-05-07T16:35:49.068Z — m2.t1
Verified `src/types.ts` was already written during m1.t2 and is complete. It exports: `BotState` enum (GREETING/SERVICE_TYPE/LOCATION/BUDGET/SEARCHING/RESULTS/BOOKING), `ServiceType` union type (nails|hair|eyebrows|lashes|waxing|massage|makeup|general), `BudgetLevel` (1|2|3|4 numeric union), `Salon` interface with all OSM/Google Places fields plus computed `distance`/`score`, `ResolvedLocation` interface, and `Session` interface keyed by `chatId` (number). Task marked done.
