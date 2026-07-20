# BonusPoints Loyalty Bot — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot for small merchants to automate loyalty point accrual, tracking, and redemption. Supports staff-facing purchase recording, balance checks, manual adjustments, and CSV reporting with admin notifications for critical actions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- small retail shop owners
- cafe managers
- online store operators
- shop staff

## Success criteria

- Accurate automatic points calculation on purchases
- Admin alerts for large redemptions/manual adjustments
- CSV export of customer balances and transaction history

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with available actions
- **/sell** (command, actor: staff, command: /sell) — Record a new purchase transaction
  - inputs: purchase amount, receipt ID (optional)
  - outputs: points calculation confirmation
- **/balance** (command, actor: user, command: /balance) — Check customer points balance
  - inputs: customer identifier (phone/@username/ID)
  - outputs: current points balance
- **/redeem** (command, actor: staff, command: /redeem) — Process points redemption for discount
  - inputs: customer identifier, points amount or discount value
  - outputs: confirmation of points deduction
- **/adjust** (command, actor: admin, command: /adjust) — Manually adjust customer points balance
  - inputs: customer identifier, points change amount, reason
  - outputs: admin notification
- **/export** (command, actor: admin, command: /export) — Generate CSV report of transactions or balances
  - inputs: report type (customers/transactions), date range
  - outputs: CSV file attachment

## Flows

### Purchase recording
_Trigger:_ /sell

1. Enter purchase amount
2. Select/identify customer
3. Calculate points based on program settings
4. Confirm transaction

_Data touched:_ Customer, Transaction

### Points redemption
_Trigger:_ /redeem

1. Select customer
2. Enter redemption amount
3. Calculate discount value
4. Confirm redemption

_Data touched:_ Customer, Transaction

### Manual adjustment
_Trigger:_ /adjust

1. Select customer
2. Enter adjustment amount
3. Enter reason
4. Confirm change

_Data touched:_ Customer, Transaction

### Admin notifications
_Trigger:_ Large redemption/manual adjustment

1. Detect threshold breach
2. Format notification message
3. Send to admin group

_Data touched:_ ProgramSettings, AdminList

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Customer** _(retention: persistent)_ — Loyalty program participant with points balance
  - fields: identifier (Telegram ID/phone), name, points balance, opt-in status
- **Transaction** _(retention: persistent)_ — Recorded purchase or adjustment
  - fields: timestamp, type (purchase/redemption/adjustment), amount, points awarded/deducted, staff ID, receipt ID
- **ProgramSettings** _(retention: persistent)_ — Loyalty program configuration
  - fields: accrual mode (percent/fixed), rate (percent/fixed points), minimum spend, redemption rate, admin list

## Integrations

- **Telegram** (required) — Bot API messaging and file delivery
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/remove staff admins
- Configure accrual mode and rates
- Set redemption ratio
- Enable/disable customer notifications
- Configure admin notification group

## Notifications

- Admin alerts for large redemptions (>500 points)
- Notifications for manual balance adjustments
- Error alerts for invalid transactions

## Permissions & privacy

- Role-based access (staff/admin)
- Telegram ID/phone number as primary identifiers
- Data encryption at rest
- No third-party data sharing

## Edge cases

- Invalid customer identifiers
- Negative adjustment amounts
- Concurrent transaction updates
- Large redemption exceeding balance

## Required tests

- End-to-end purchase recording with points calculation
- Redemption workflow with discount calculation
- Manual adjustment with admin notification
- CSV export format validation

## Assumptions

- Default 5% accrual rate
- Russian as primary language
- Admin group configured during setup
- Telegram user IDs preferred for identification
