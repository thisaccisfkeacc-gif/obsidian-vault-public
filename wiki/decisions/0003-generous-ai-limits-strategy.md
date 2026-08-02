# ADR-0003: Generous AI Quotas & Dynamic Backend Control Strategy

- **Status**: Active
- **Date**: 2026-08-01
- **Category**: Strategy & Architecture

## Context
Deciding initial AI request limits for Free Trial vs Paid users in PowerX Keys, balancing initial user growth/delight against server API costs.

## Decision
1. **Phase 1 (Launch & Hype Phase)**:
   - **Free Trial**: 20–25 AI requests / day.
   - **Paid / Lifetime**: 75–100 AI requests / day.
2. **Fair Request Weighting**:
   - 1 Chat Message = 1 count.
   - 1 AI Builder Macro Generation = 2 counts (higher token footprint).
3. **Backend Master Knob**:
   - Quotas enforced per account ID in Supabase Edge Function (prevents VPN/IP resets).
   - Keeps client app decoupled from quota values.
4. **Phase 2 (Milestone-Triggered Adjustment)**:
   - Adjust quotas down dynamically (e.g., 10 Free / 30 Paid) when hitting any of 3 checkpoints:
     - 5–10 paying customers (revenue pays for official API keys).
     - 1,000 total app downloads.
     - Monthly server AI API bill reaches $20–$30.

## Rationale
Zero friction during launch maximizes adoption and positive reviews. Server-side control allows instant quota tuning without requiring app client updates.
