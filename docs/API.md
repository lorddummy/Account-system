# API (Planned)

REST API for the Account system. Works in parallel with the [Marker-TITO replacement](https://github.com/lorddummy/Marker-TITO-replacement) ticket API.

## Base

- Base URL: TBD (e.g. `https://api.<tenant>.accounts.example/v1`)
- Auth: TBD (API key for cage/EGM; no player auth in V1 — session identified by card/session token)
- Idempotency: Load and debit/credit support idempotency keys

## Accounts

### Create or get account (by card)

**POST** `/accounts/lookup`

Resolves a card to an account (creates account on first use if desired).

- **Request:** `{ "card_id": string, "property_id": string }`
- **Response:** `{ "account_id": string, "balance_cents": number, "created": boolean }`

---

### Get balance

**GET** `/accounts/:account_id/balance`

- **Response:** `{ "account_id", "balance_cents", "currency" }`
- Auth: system or cage only (not player-facing in V1)

---

### Load (card punch)

**POST** `/accounts/load`

Add credits to the account identified by card. Used at cage/kiosk.

- **Request:** `{ "card_id": string, "amount_cents": number, "property_id": string, "redemption_point_id": string (optional), "idempotency_key": string (optional) }`
- **Response:** `{ "account_id": string, "balance_cents": number, "transaction_id": string }`

---

### Debit / credit (for EGM or cage)

**POST** `/accounts/:account_id/credit`  
**POST** `/accounts/:account_id/debit`

Adjust balance (e.g. play at machine, payout at cage). Idempotency recommended.

- **Credit request:** `{ "amount_cents": number, "reason": string (e.g. "win", "ticket_redemption", "load"), "reference_id": string (optional), "idempotency_key": string (optional) }`
- **Debit request:** `{ "amount_cents": number, "reason": string (e.g. "play", "payout"), "reference_id": string (optional), "idempotency_key": string (optional) }`
- **Response:** `{ "account_id", "balance_cents", "transaction_id" }`

---

## Sessions (card in)

### Start or update session

**POST** `/sessions`

Bind a machine to an account (player “card in” or QR scan).

- **Request:** `{ "machine_id": string, "account_id": string (or "card_id" to resolve), "property_id": string }`
- **Response:** `{ "session_id": string, "machine_id": string, "account_id": string, "started_at": string }`
- Side effect: any existing session for this `machine_id` is ended; account may be unbound from previous machine.

---

### Get active account for machine

**GET** `/sessions/machine/:machine_id`

Used by EGM to know which account to debit/credit.

- **Response:** `{ "session_id": string, "account_id": string, "started_at": string }` or 404 if no session.

---

### End session

**POST** `/sessions/:session_id/end`

Player “cards out” or session timeout.

- **Response:** `{ "session_id", "ended_at" }`

---

## Transaction history (audit)

**GET** `/accounts/:account_id/transactions`

Optional: paginated list of balance-changing events (load, debit, credit) for audit and dispute.

- **Query:** `?from=iso8601&to=iso8601&limit=50`
- **Response:** `{ "transactions": [ { "transaction_id", "amount_cents", "reason", "balance_after", "created_at", "reference_id" }, ... ] }`

---

## Data Model (Planned)

- **Account:** `account_id` (UUID), `property_id`, `balance_cents`, `currency`, `created_at`, `updated_at`. Optional: `card_id` (primary linked card).
- **Card → account:** Mapping table `card_id` + `property_id` → `account_id` (one account per card per property).
- **Session:** `session_id`, `machine_id`, `account_id`, `property_id`, `started_at`, `ended_at`.
- **Transaction:** `transaction_id`, `account_id`, `amount_cents` (signed), `reason`, `balance_after`, `reference_id`, `idempotency_key`, `created_at`.
