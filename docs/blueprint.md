# Crypto Tracker Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking crypto prices with customizable alerts, watchlists, and morning summaries. Users manage personalized coin lists, set price-threshold and percentage-move alerts with cooldowns, and receive optional daily updates. Owner gains insights into usage patterns and top alert triggers.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto traders
- crypto hobbyists

## Success criteria

- users receive accurate price alerts within 30-minute cooldown windows
- morning summaries delivered at user-specified local times
- quiet hours suppress alerts without losing notifications

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize user profile and onboarding flow
- **/price** (command, actor: user, command: /price) — Check full watchlist prices or specific ticker with 1h/24h changes
- **Manage Watchlist** (button, actor: user, callback: watchlist:manage) — Edit watchlist entries with add/remove/alert buttons
- **Set Price Alert** (button, actor: user, callback: alert:price) — Configure price threshold alerts for selected coins
- **Set % Alert** (button, actor: user, callback: alert:percent) — Configure percentage move alerts with 1-hour lookback

## Flows

### Onboarding Setup
_Trigger:_ /start

1. Collect timezone
2. Ask about morning summary preference
3. Seed watchlist with Bitcoin/Ethereum/Toncoin

_Data touched:_ user profile

### Alert Creation
_Trigger:_ alert:price|alert:percent

1. Select coin
2. Choose alert type/direction
3. Set threshold/percent value
4. Confirm cooldown rules

_Data touched:_ alert, watchlist entry

### Morning Summary
_Trigger:_ scheduled (user timezone)

1. Compile watchlist prices
2. Highlight 24h moves >5%
3. Format summary with actionable insights

_Data touched:_ user profile, watchlist entry

### Quiet Hours Handling
_Trigger:_ alert firing during quiet hours

1. Queue alert notification
2. Deliver after quiet hours end
3. Check cooldown before sending

_Data touched:_ alert, user profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user profile** _(retention: persistent)_ — User preferences and settings
  - fields: chat_id, locale, timezone, quiet_hours, summary_time, last_alert_check
- **watchlist entry** _(retention: persistent)_ — Tracked cryptocurrency with metadata
  - fields: ticker, display_name, latest_price, alert_status
- **alert** _(retention: persistent)_ — Price alert configuration and history
  - fields: type, direction, threshold, lookback_window, last_fired, active
- **owner metrics** _(retention: persistent)_ — Aggregated usage statistics
  - fields: total_users, alert_type_counts, top_coins

## Integrations

- **Telegram** (required) — Bot API messaging
- **Crypto Price Feed** (required) — Real-time price data with retries
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- admin-only command to view metrics dashboard
- configure seed watchlist coins

## Notifications

- Price threshold alerts with directional triggers
- Percentage move alerts with 1-hour lookback
- Daily morning price summary
- Queued alert delivery after quiet hours

## Permissions & privacy

- All user data encrypted at rest
- No cross-user data sharing
- Explicit consent required for morning summaries

## Edge cases

- Price feed outages with silent retries
- Conflicting alert conditions during cooldown
- User timezone daylight saving transitions
- Invalid ticker symbols in free-text input

## Required tests

- End-to-end alert delivery during/after quiet hours
- Cooldown suppression of duplicate alerts
- Morning summary formatting with real price data

## Assumptions

- Price feed API key will be provided separately
- Default seed coins (BTC, ETH, TON) meet user needs
- 30-minute cooldown prevents alert spam
