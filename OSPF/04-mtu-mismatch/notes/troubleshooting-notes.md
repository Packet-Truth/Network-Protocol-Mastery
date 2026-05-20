# Troubleshooting Notes

Observed issue:
- OSPF stuck in EXSTART/EXCHANGE

Checks performed:
- Verified Hello packets
- Verified Area ID
- Verified timers
- Verified DBD exchange
- Checked MTU values

Root cause:
- MTU mismatch between R3 and R4

Fix:
- Match interface MTU values
