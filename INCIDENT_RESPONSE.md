# Incident Response Runbook — Vero Protocol

**Classification:** INTERNAL — SECURITY SENSITIVE  
**Owner:** Vero Security Team  
**Last Reviewed:** 2026-06-19

---

## Severity Levels

| Level | Label    | Response SLA | Examples                                      |
|-------|----------|-------------|-----------------------------------------------|
| P0    | CRITICAL | 15 min      | Active exploit, funds at risk, bridge drained |
| P1    | HIGH     | 1 hour      | Auth bypass, unauthorized admin ops           |
| P2    | MEDIUM   | 4 hours     | Data inconsistency, relayer anomaly           |
| P3    | LOW      | 24 hours    | Minor contract deviation, code debt           |

---

## Phase 1 — Detection

Sources monitored by `vero-audit-guard`:
- `anomaly-detector`: nonce spikes, failed tx bursts, unauthorized addresses
- `scanner-engine`: static analysis on every PR to `vero-core-contracts`
- `verifiable-audit-trail`: on-chain hash mismatch alerts

**Immediate action on alert:**
1. Page on-call engineer via PagerDuty.
2. Capture raw alert payload and timestamp.
3. Open incident channel `#inc-<YYYYMMDD-HHMM>` in Slack.

---

## Phase 2 — Containment

### P0/P1 Protocol Pause
```bash
# Invoke emergency pause on the core contract (requires multisig)
stellar contract invoke \
  --id <CONTRACT_ID> \
  --source EMERGENCY_KEYPAIR \
  -- emergency_pause
```

### Relayer Isolation
```bash
# Stop relayer service immediately
systemctl stop vero-relayer   # or kubectl rollout pause deployment/vero-relayer
```

### Revoke Compromised Keys
1. Rotate `AUDIT_KEYPAIR_SECRET` and all `RELAYER_*` env vars.
2. Submit key rotation transaction on Stellar.
3. Update GitHub Secrets via org admin.

---

## Phase 3 — Analysis

- Pull latest scan report: `reports/latest-scan.json`
- Verify on-chain audit hash matches local report (use `verifiable-audit-trail`)
- Review Horizon transaction history for anomalous addresses
- Examine `anomaly-detector` alert log

---

## Phase 4 — Eradication & Recovery

1. Deploy patched contract with incremented version.
2. Re-run full `scanner-engine` scan; confirm zero CRITICAL findings.
3. Anchor new clean report hash on-chain.
4. Gradually resume relayer traffic under enhanced monitoring.

---

## Phase 5 — Post-Mortem

- Required within **48 hours** of P0/P1 resolution.
- Template: `docs/post-mortem-template.md`
- Publish sanitized summary in `reports/post-mortems/`.

---

## Contact Matrix

| Role               | Contact           |
|--------------------|-------------------|
| Security Lead      | security@vero.xyz |
| Protocol Engineer  | eng@vero.xyz      |
| Bug Bounty Reports | See VULNERABILITY_DISCLOSURE.md |
