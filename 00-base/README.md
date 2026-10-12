# Phase 0: Foundation

**Status:** Complete (2026-10-10)

## Goal
Document the starting point of the lab and prepare the tools needed for the rest of the project.

## Tasks
- [x] Back up the hAP ax lite (RouterOS 7.8)
- [x] Back up the L009
- [x] Record the current topology and addressing (see `docs/`)
- [x] Update RouterOS on both MikroTik devices
- [x] Install Linux and Docker on the lab server
- [x] Create the public and private GitHub repositories

## Starting point
- Triple NAT: ISP ONU, Reyee router and hAP edge router
- The hAP does NAT, DHCP for customers and OLT management
- The L009 only bridges ports

## Lessons learned
- Triple NAT (ISP ONU, Reyee router and hAP) made the lab harder to reason about; documented in `docs/troubleshooting/asymmetric-routing.md`.
- A server with the wrong default gateway caused asymmetric routing: replies came back through a different router than the one that sent them, and the firewall dropped them.
- Backups before every change made it safe to experiment.