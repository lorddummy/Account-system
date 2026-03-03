# Integration with Marker-TITO Replacement (Ticket System)

The **Account system** and the **Ticket system** ([Marker-TITO-replacement](https://github.com/lorddummy/Marker-TITO-replacement)) work **in parallel**: one casino flow, two backends.

## Shared Concepts

- **Property** — Same `property_id` in both systems.
- **Currency** — Same units (e.g. cents) for ticket value and account balance.
- **Account on ticket** — Optional: when issuing a ticket, ticket system can store `account_id` for audit and loyalty.

## Integration Points

### 1. Cash-out at EGM

When the player cashes out:

- **Ticket system:** Issue digital ticket (paperless voucher) — `POST /tickets` with `value_cents`, `property_id`, `machine_id`. Optionally include `account_id` in metadata.
- **Account system:** Optionally credit the session’s account — `POST /accounts/:account_id/credit` with reason `cash_out` (so balance is updated and no ticket needed), or do both: issue ticket and credit account (operator choice).

Flow (orchestration in EGM or a small backend):

1. EGM gets `account_id` from Account system (`GET /sessions/machine/:machine_id`).
2. EGM (or middleware) calls Ticket system to **issue ticket**; optionally passes `account_id`.
3. Optionally call Account system to **credit** the same amount to `account_id` (if cash-out goes to account instead of or in addition to ticket).

### 2. Ticket redemption at cage

When the player redeems a ticket at cage:

- **Ticket system:** Validate and redeem ticket — `POST /tickets/redeem` with token, `property_id`, `redemption_point_id`.
- **Account system (optional):** If the ticket has an `account_id`, credit that account instead of (or in addition to) paying cash — `POST /accounts/:account_id/credit` with reason `ticket_redemption`, `reference_id` = ticket_id.

So: ticket is marked redeemed in ticket system; value can be paid as cash and/or added to account.

### 3. Load at cage (card punch)

Account system only: cage scans card, loads credits to account — `POST /accounts/load`. No ticket involved.

### 4. Session (card in)

Account system only: card in or QR at machine → `POST /sessions` binds machine to account. Ticket system is not involved until cash-out (issue ticket and/or credit account).

## Deployment

- Both systems can run in the same property/tenant.
- EGM or a thin **orchestration layer** can call both APIs: Account for session and balance, Ticket for issue/redeem.
- Alternatively, Ticket system can call Account system when redeeming (to credit account); Account system can call Ticket system to issue ticket on cash-out, or EGM calls both.

## Summary

| Event           | Account system                    | Ticket system              |
|----------------|------------------------------------|----------------------------|
| Card in        | Create session (machine ↔ account)| —                          |
| Load at cage   | Credit account (load)             | —                          |
| Play           | Debit/credit account (if used)    | —                          |
| Cash out       | Optionally credit account         | Issue digital ticket        |
| Redeem ticket  | Optionally credit account         | Validate/redeem ticket     |

Same flow as real casino; account = card in + balance; ticket = paperless voucher. Both systems work in parallel.
