# Phase 1: ISP-style network

**Status:** Pending

## Goal
Turn the L009 into the ISP core: VLANs with filtering, a management network separated from customers, and a PPPoE server for subscribers.

## Plan
- [ ] Bridge with VLAN filtering on the L009
- [ ] Management VLAN for the OLT, separate from customer traffic
- [ ] Routed link between the hAP (upstream) and the L009
- [ ] PPPoE server on the customer VLAN, with IP pool and speed profiles
- [ ] Firewall rules: management Wi-Fi allowed, customers isolated
- [ ] Clean up leftover interfaces and routes on the hAP

## Folders
- `configs/`: reviewed `.rsc` exports (no passwords or public IPs)
- `evidence/`: screenshots and command output

## Results
(To be filled in)

## Lessons learned
(To be filled in)