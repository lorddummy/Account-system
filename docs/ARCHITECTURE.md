# Architecture

## Role

The **Account system** answers:

- **Who is at this machine?** — Card in (or QR scan) binds the machine to a player account for the session.
- **What is their balance?** — Credits loaded at cage, used at machine; cash-out can go to account and/or to a digital ticket.

It runs **in parallel** with the **Marker-TITO replacement** (paperless ticket system): same flow, two systems. Ticket system = voucher out; Account system = card in + identity + balance.

## Core Concepts

### Account

- One **account** per player per property (or global, depending on deployment).
- Holds **balance** (same currency as tickets, e.g. cents).
- Linked to **card** (e.g. loyalty card barcode/RFID) so “card punch” at cage = add credits to that account.

### Session (card in)

- When the player **inserts card** (or scans machine QR / machine scans player QR), create a **session**: `machine_id` ↔ `account_id`.
- Only one active account per machine; one account typically one machine at a time.
- EGM asks: “Who is at machine X?” → Account system returns `account_id` (and optionally balance).

### Balance

- **Credit** — Load at cage (“card punch”), or ticket redemption (if ticket system credits account), or cash-out to account.
- **Debit** — Play at machine (when using account balance), or payout at cage.
- All changes in an **audit log** (transaction history).

## Flows

1. **Card punch (load)** — Cage/kiosk: scan card → resolve to account → add credits. Audit: load, amount, location, time.
2. **Session start (card in)** — Player at machine: card in or QR scan → create/update session (machine ↔ account). EGM can now debit/credit this account.
3. **Play** — EGM debits account (or uses cash in machine); winnings credit account (or stay in machine). Optional: all play goes through account.
4. **Cash out** — EGM can (a) request digital ticket from **ticket system** (paperless voucher), and/or (b) credit **account**. Optionally tie issued ticket to `account_id` in ticket system for audit.
5. **Redeem at cage** — Ticket system: validate/redeem ticket (cash payout). Account system: “payout balance” (cash out from account). Ticket redemption can also credit account instead of cash.

## Integration with Ticket System

- **Same property, same currency** — Account and tickets use same `property_id` and currency (e.g. cents).
- **Cash-out** — Backend can call ticket system to **issue ticket** and store `account_id` on the ticket (optional). Or credit account only; or both.
- **Ticket redeem** — Ticket system can call account system to **credit account** instead of (or in addition to) cash payout.
- **Audit** — Both systems keep audit trails; `account_id` on tickets links voucher activity to the account.

## Components (Planned)

- **Account service** — Create/lookup account (e.g. by card_id). Balance: credit/debit with audit.
- **Session service** — Create session (machine ↔ account), resolve “active account for machine,” end session.
- **API** — REST for load, session bind, balance, debit/credit (for EGM/cage).
- **Card mapping** — card_id → account_id (per property).

## Out of Scope (V1)

- Full player identity (name, KYC) — account can be anonymous/keyed by card only.
- Loyalty/comps logic — can layer on later using account_id and transaction history.
