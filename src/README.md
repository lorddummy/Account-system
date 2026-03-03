# Source

Backend and API for the Account system will live here.

Planned:

- Account service (create/lookup by card, balance, load, credit/debit)
- Session service (bind machine ↔ account, get active account for machine, end session)
- Card → account mapping
- Transaction audit log
- REST API for cage, EGM, and optional orchestration with ticket system

Tech stack TBD (e.g. Node, Go, or Python). Must integrate with [Marker-TITO-replacement](https://github.com/lorddummy/Marker-TITO-replacement) for cash-out (issue ticket) and optional ticket redemption → account credit.
