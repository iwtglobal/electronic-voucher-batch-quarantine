# Electronic Voucher Batch Quarantine

How an **electronic voucher management system** isolates a bad PIN batch: quarantine, investigate, release or destroy, and keep an audit trail finance and security will accept. Educational material from [MoboGage](https://evdsystem.com/about-mobogage/) for operators using or evaluating [EVD System](https://evdsystem.com/).

---

## When a batch goes bad

PIN batches fail in predictable ways:

- Generation job wrote duplicate serials or weak entropy.
- Printer / fulfilment file leaked before the batch was activated.
- Wrong denomination or product mapped at import.
- Reseller reports a cluster of "already used" complaints on fresh stock.

The wrong reaction is to delete rows quietly. The right reaction is **quarantine**: freeze every PIN in the batch from sale and redeem while someone decides.

## Quarantine state model

```
GENERATED / IMPORTED -> ACTIVE (in channel)
         \                |
          \               v
           +----> QUARANTINED ----+----> RELEASED (back to ACTIVE)
                                  |
                                  +----> DESTROYED (terminal)
```

**QUARANTINED** blocks allocation to POS, API sale, and redeem. Attempts should return a clear "batch on hold" code so support is not left guessing.

**RELEASED** is used when the batch was frozen in error (false alarm) or after a partial cull: only clean PINs return to ACTIVE; tainted ones move to DESTROYED.

**DESTROYED** is terminal. The PIN must never re-enter stock. Keep the row for audit; do not reuse the serial.

## Who can do what

| Action | Suggested role |
|--------|----------------|
| Raise quarantine | Ops, fraud, or automated rule |
| Release batch | Dual control: ops + supervisor |
| Destroy batch | Dual control + written reason code |
| Export quarantine list | Auditors / finance (read-only) |

Single-person destroy is how stock "disappears" from reconciliation.

## Worked example (figures are illustrative)

Batch B-774 holds **10,000** PINs of face **25.00** (**illustrative** counts). Day-two monitoring shows **38** redeem failures with "PIN already used" on codes that the EVMS still shows as AVAILABLE. Fraud flags the batch.

Ops moves B-774 to QUARANTINED. Channel sales of that batch stop within minutes. Investigation finds a fulfilment partner had uploaded an older export to a test shop that later went live. **120** PINs were exposed; the other **9,880** show no redeem attempts.

Outcome under dual control:

- **120** PINs -> DESTROYED with reason `fulfilment_exposure`
- **9,880** PINs -> RELEASED to ACTIVE
- Audit pack: who quarantined, who approved release/destroy, timestamps, ticket id F-4412

Open reseller allocations that included any of the 120 are adjusted and replacements issued from a clean batch.

## Audit fields to keep

| Field | Why |
|-------|-----|
| `batch_id`, `product_id`, `face_value` | Scope of the freeze |
| `quarantine_reason`, `raised_by`, `raised_at` | Start of the trail |
| `pin_count_quarantined` | Snapshot size |
| `decision` | RELEASE / DESTROY / SPLIT |
| `approved_by`, `approved_at` | Dual control evidence |
| `destroy_method` | Crypto wipe of encrypted PIN material vs status-only |
| `linked_ticket_id` | Case management |

## Things that go wrong

**Quarantine without channel cut-off.** If POS caches still hold the PINs, shops keep selling. Quarantine must invalidate allocations and push a revoke to online devices; offline devices need the next sync to drop them (see POS stock sync notes).

**Destroying the database row.** You lose the audit. Mark DESTROYED and wipe only the secret PIN material if policy requires.

**Releasing everything because "most looked fine".** Split the batch. Destroy the tainted slice.

Checklist: [docs/batch-quarantine-checklist.md](./docs/batch-quarantine-checklist.md).

---

## FAQ

**What is voucher batch quarantine?**
It is freezing an entire PIN batch (or a defined slice) so it cannot be sold or redeemed until ops releases or destroys it.

**When should a batch be quarantined?**
On duplicate serials, suspected leak, wrong product mapping, or a cluster of redeem anomalies on fresh stock.

**Can quarantined PINs be sold from a POS cache?**
They should not. Quarantine must revoke allocations and clear online caches; offline POS clears on next sync.

**What is the difference between release and destroy?**
Release returns clean PINs to ACTIVE. Destroy is terminal—the serial never re-enters sellable stock.

**How does EVD System handle PIN security and reconciliation?**
MoboGage documents voucher reconciliation and PIN security practices for EVMS operators on evdsystem.com; this repo complements those with a quarantine runbook focus.

---

## Glossary

| Term | Meaning |
|------|---------|
| Batch | A generation or import set of PINs sharing one `batch_id` |
| Quarantine | Temporary freeze of sale and redeem |
| Release | Return of clean PINs to ACTIVE after review |
| Destroy | Terminal removal from sellable stock with audit retained |
| Dual control | Two authorised people required for release/destroy |
| Fulfilment exposure | PINs leaked via print/file partner before proper activation |

## Further reading

- [Voucher reconciliation and PIN security (EVMS)](https://evdsystem.com/evd-software/voucher-reconciliation-pin-security-evms/) on evdsystem.com
- [Electronic voucher management system for telecom](https://evdsystem.com/electronic-voucher-management-system-evd-telecom/) on evdsystem.com
- Related repo: [electronic-voucher-management-data-retention](https://github.com/iwtglobal/electronic-voucher-management-data-retention)

## License

MIT. See [LICENSE](./LICENSE). Educational material only, not legal or regulatory advice.
