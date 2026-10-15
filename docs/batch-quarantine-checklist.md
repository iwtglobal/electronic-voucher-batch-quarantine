# Batch Quarantine Checklist

For EVMS ops when a PIN batch is suspected bad.

## Raise quarantine

1. Identify `batch_id` and confirm product / face value.
2. Move batch (or tainted serial range) to QUARANTINED.
3. Revoke open allocations to POS / resellers for those PINs.
4. Push cache invalidate to online channels.
5. Open ticket with reason, sample failing PINs, and counts.
6. Notify reseller support if stock was already in the field.

## Decide release vs destroy

- [ ] Evidence reviewed (logs, fulfilment files, redeem failures)
- [ ] Tainted serial list produced (may be empty on false alarm)
- [ ] Dual-control approval recorded for RELEASE and/or DESTROY
- [ ] DESTROYED PINs: secret material wiped per policy; row kept for audit
- [ ] RELEASED PINs: state ACTIVE; allocations re-issued if needed
- [ ] Finance informed of any liability / replacement stock

## After close

- [ ] Channel monitoring for 24–48h on related products
- [ ] Root cause tagged (generation, fulfilment, mapping, fraud)
- [ ] Runbook update if a new failure mode appeared
