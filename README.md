# Account System

**Player account and session layer** that works **in parallel** with the [Marker-TITO replacement](https://github.com/lorddummy/Marker-TITO-replacement) (paperless ticket system).

Same real-casino flow — sit down, **card in**, put money in, play, cash out (paperless voucher) — with the account system handling **who is at the machine** and **balance**; the ticket system handling the **paperless voucher** on cash-out.

## What This Does

- **Card in / session** — When a player sits at a machine and inserts their card (or scans QR), this system binds **that machine** to **that account** for the session. The EGM knows who is playing.
- **Account balance** — Credits can be loaded at the cage (“card punch”) and used at the machine; cash-out can credit the account and/or issue a digital ticket.
- **Works with tickets** — Cash-out can issue a digital ticket (via the ticket system) and optionally tie it to the account for audit/loyalty. Ticket redemption can credit an account.

## Flow (with Ticket System)

| Step | Account system | Ticket system |
|------|----------------|---------------|
| Sit down | **Card in** → session: machine ↔ account | — |
| Put money in | Optional: load from account balance | — |
| Play | Debit/credit account (or cash in machine) | — |
| Cash out | Optional: credit account | **Issue digital ticket** (paperless voucher) |
| Redeem at cage | Optional: payout from account | **Validate/redeem ticket** |

## Repo Structure

```
Account-system/
├── README.md           # This file
├── docs/               # Architecture, API, integration with ticket system
├── src/                # Backend / API (to be implemented)
├── .gitignore
└── LICENSE
```

## Status

**Early stage.** This repo defines the product and will hold the backend/API. Integrates with [Marker-TITO-replacement](https://github.com/lorddummy/Marker-TITO-replacement) for the full paperless flow.

## License

See [LICENSE](LICENSE).
