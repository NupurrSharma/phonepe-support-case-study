# PhonePe: Reducing Support Load from Failed & Pending UPI Transactions
**A Product Management Case Study**

## Context

PhonePe processes over 1,000 crore UPI transactions a month and holds close to half of India's total UPI volume — more than Google Pay and Paytm combined with every smaller player (NPCI, 2026). At that scale, even a small failure rate turns into millions of confused users a month, many of whom have just watched money leave their bank account with no confirmation of where it went.

PhonePe doesn't publish its internal support volumes, contact rates, or staffing numbers, so this case study leans on public NPCI and industry data wherever it exists, and is explicit about what's an assumption on top of that. That's the same discipline I'd want to bring into a real product review — a number is only useful if you know exactly how confident to be in it.

## 1. The Problem

The visible symptom is a steady stream of "where is my money" support contacts for transactions that are failed, pending, or stuck in a state where PhonePe shows one status and the bank or merchant shows another. It's tempting to treat this as a staffing question — hire more agents, cut the wait time. But that only addresses the symptom.

The real question is why a user on a real-time payment system ever needs to talk to a human being to find out what happened to a payment they just made.

| Customer problem | Why it happens | What the customer actually needs |
|---|---|---|
| Debited but not credited | Technical decline or beneficiary bank timeout; auto-reversal hasn't run yet | A clear status and a guaranteed reversal timeline |
| PhonePe shows success, merchant shows failed | Deemed-approval mismatch between PhonePe, NPCI, and the receiving bank | One reconciled source of truth |
| Sent to the wrong UPI ID | Fat-fingered entry, weak confirmation step before authentication | Stronger pre-payment verification and a self-serve dispute path |
| Refund pending for days | Manual reconciliation queue after automated reversal fails | Proactive status updates instead of silence |

## 2. Root Cause

NPCI splits every UPI decline into two categories, and the split matters because it tells you who owns the fix. A business decline is on the user's side — wrong PIN, insufficient balance, an exceeded limit. A technical decline sits with the bank or NPCI infrastructure — downtime, timeouts, middleware failures. Technical declines have actually improved a lot industry-wide, falling from an estimated 8–10% of transactions in 2016 to under 1% by 2025 as banks hardened their systems. But at PhonePe's volume, even a fraction of a percent is millions of failed transactions a month.

The failures that actually drive support contact aren't the ones a user can self-diagnose — wrong PIN, just retry. They're the debited-but-not-credited and deemed-approved cases, where the customer did everything right and the system simply can't tell them what happened yet. RBI and NPCI guidelines call for automatic reversal in most of these cases, but when that doesn't fire instantly, it drops into a manual bank-side reconciliation queue. That's exactly where public complaint records show refunds stalling for days, and in some disputed cases, months.

The pattern underneath most of these complaints isn't the failure itself. It's the silence that follows it.

**Current path:** debit happens → app shows "failed" or "pending" → no clear timeline → user waits, nothing changes → user contacts support → agent manually checks bank-side status → user waits again.

**Product-led path:** debit happens → system determines within seconds whether it's auto-reversible or needs manual review → user gets a specific, honest status ("reversal in progress, credited by [date]" vs "this needs manual review, update within X hours") → most cases never need a support contact at all.

## 3. Business Goals

The primary goal is reducing unnecessary, avoidable support contact for failed and pending transactions — without doing it in a way that quietly damages trust. Underneath that:

- Lower cost-to-serve per transaction dispute
- Faster resolution on debited-not-credited cases specifically
- Protect trust metrics, which matter more for a payments app ahead of an IPO than almost any other product category — one viral "PhonePe took my money" story does real brand damage
- Reduce load on the Grievance Officer and RBI Ombudsman escalation channels, which are visible to the regulator, not just to CX
- Prevent the same failure categories from recurring

## 4. Product Outcomes, Not Features

| Business goal | Product outcome | Example solution |
|---|---|---|
| Reduce support contacts | Lower contact rate per failed transaction | Real-time status engine |
| Reduce cost-to-serve | Higher self-service resolution | In-app dispute and refund tracker |
| Protect trust | Faster time-to-reversal | Auto-reconciliation with beneficiary banks |
| Reduce escalations | Fewer Grievance Officer / Ombudsman cases | Proactive, SLA-bound communication |
| Prevent recurrence | Lower repeat-failure rate by bank corridor | Bank health monitoring and smart routing |

"Add a chatbot for transaction queries" is a feature. "Increase self-service resolution rate for failed-transaction contacts" is the outcome, and it matters which one drives the roadmap. A chatbot rarely moves that number on its own, because most of these cases need real account-level status data, not conversational deflection.

## 5. Solution Strategy

**Phase 1 — kill the silence.** Real-time, plain-language transaction status instead of a generic "Pending." A visible reversal countdown the moment a debit-without-credit is detected. Proactive push or SMS the moment status changes, rather than expecting the user to keep checking. Honest language that distinguishes "your bank is still confirming" from "this needs manual review." This is the phase that should absorb the largest chunk of contacts, since uncertainty — not disagreement with the outcome — is what drives most of them in the first place.

**Phase 2 — make self-service actually resolve things.** A dispute tracker with live reconciliation status, not a static FAQ page. Auto-triggered reversal checks surfaced directly to the user instead of gated behind an agent. A guided flow for wrong-recipient transfers that sets honest expectations instead of implying an instant fix, since that category genuinely needs human or bank intervention. Automated resolution for the highest-confidence bucket: technical declines with confirmed auto-reversal eligibility. Done well, this frees agents to focus on the smaller set of cases — wrong recipient, suspected fraud, cross-bank disputes — that actually need judgment.

**Phase 3 — fix it before it fails.** NPCI already tracks per-bank technical and business decline rates; PhonePe can use that same signal internally to route around a bank corridor that's currently degraded. A feedback loop from support-ticket taxonomy back into engineering and ops priorities closes the gap between "we keep getting the same complaint" and "we fixed the thing causing it." The goal here isn't faster handling — it's fewer failures entering the system at all.

## 6. Estimating Support Capacity

PhonePe doesn't disclose contact-rate or staffing data, so this is a Fermi estimate anchored to real, sourced transaction volume — not a claim about actual headcount.

| Variable | Value | Basis |
|---|---|---|
| PhonePe daily UPI transactions | ~340M | NPCI monthly data, ~1,033 crore/month (2026) |
| Combined decline rate | 2% | Anchored to NPCI's falling technical-decline trend plus typical business-decline patterns |
| Daily failed/declined transactions | 6.8M | 340M × 2% |
| Self-resolved without contact | 90% | Assumption — most business declines are transient, user retries |
| Enter the in-app "failed transaction" flow | 680K/day | 10% of 6.8M |
| Resolved via self-service | 85% | Assumption — the target Phase 2 is designed to hit |
| Escalate to human-assisted support | 102,000/day | 15% of 680K |
| Average handling time | 8 minutes | Higher than a typical e-commerce query given verification and compliance steps |
| Productive shift time | 420 minutes | 8-hour shift, 7 productive hours |
| Capacity per agent per shift | ~52 cases | 420 ÷ 8 |
| Agent-shifts needed per day | ~1,960 | 102,000 ÷ 52 |

Adding a 30% buffer for peak load, leave, training, and absence puts the estimate at roughly **2,500–2,600 agents**, just for the failed/pending-transaction category — not PhonePe's full support org, which would also cover KYC, merchant support, lending, and insurance.

The number matters less than the method. Anchor to real, disclosed data wherever it exists, and flag every layer that's an assumption. That's the difference between guessing and giving an interviewer a reasoning chain they can correct with their own internal numbers.

## 7. KPI Tree

```
Business goal
  Reduce unnecessary support demand
  Reduce cost-to-serve
  Protect trust ahead of IPO
  Reduce regulatory escalation volume
        |
Product outcomes
  Lower contact rate per failed transaction
  Higher self-service resolution rate
  Faster time-to-reversal
  Fewer Grievance Officer / Ombudsman escalations
  Lower repeat-failure rate by bank corridor
        |
Product drivers
  Real-time plain-language status
  Proactive reversal countdown
  Self-serve dispute tracker
  Auto-reconciliation with beneficiary banks
  Bank-health monitoring and smart routing
  Support-ticket to engineering feedback loop
        |
Customer outcome
  Knowing exactly where the money is, without asking a human
```

## 8. KPI Framework

**Business:** cost per resolved dispute, support contacts per 10,000 transactions, regulatory escalation volume, user retention following a failed-transaction incident.

**Product:** self-service resolution rate, time-to-reversal (P50/P90), first contact resolution, repeat-contact rate on the same transaction, technical decline rate by bank corridor.

**Customer:** CSAT specifically on failed-transaction flows, customer effort score, share of users who understood their transaction status without contacting support.

## 9. Guardrail Metrics

Reducing contact volume isn't automatically a win. If self-service hides the "talk to a human" option too aggressively to hit a target, trust breaks faster than it was built. Contact rate going down while CSAT drops, or Ombudsman escalations rise, means the product got worse, not better.

Worth watching alongside the primary metrics: CSAT and NPS specifically for users who hit a failed transaction, complaint escalation rate to the RBI Ombudsman, self-service abandonment rate (users bouncing back to a call after trying the tracker), and any spike in wrong-recipient disputes, which need human review rather than automation. A visible, one-tap path to a human agent has to stay available — self-service should absorb the simple, high-confidence cases, never the ambiguous or high-stakes ones.

## 10. Cross-Functional Ownership

| Team | Responsibility |
|---|---|
| Product | Define outcomes, prioritize roadmap, own the status and tracker experience |
| Engineering | Real-time status pipeline, auto-reconciliation with bank APIs, routing logic |
| Data | Failure taxonomy analysis, bank-corridor health dashboards |
| Support / CX | Ticket taxonomy, edge-case identification, agent tooling requirements |
| Risk & Compliance | RBI/NPCI guideline adherence for auto-reversal and fraud-flagged disputes |
| Bank / NPCI partnerships | Escalation path for recurring corridor-level technical declines |

The PM role here is more cross-functional than a typical support-reduction project, because payments disputes touch compliance directly — a wrong self-service resolution isn't just a bad experience, it can become a regulatory problem.

## 11. Validation Plan

Start by breaking down failed-transaction contacts by decline type, bank corridor, amount bracket, time-to-resolution, and whether an Ombudsman escalation followed. The likely biggest drivers, based on public complaint patterns, are debited-not-credited cases with delayed manual reversal, and merchant-side "deemed success" mismatches — but that needs confirming against real data, not assuming.

From there, talk to affected users — not just "why did you contact support" but "at what point did you stop trusting the in-app status and decide you needed a human." Ship the smallest version of Phase 1 (real-time status plus reversal countdown) to a segment and measure the contact-rate and CSAT delta before expanding. Scale only if both business and trust metrics move together — a self-service win that quietly increases Ombudsman escalations isn't actually a win.

## 12. Risks and Constraints

Every number in this case study comes from public NPCI and industry statistics, not PhonePe's internal data, so it's directionally reasoned rather than confirmed. Any automated reversal logic touching real money needs RBI/NPCI compliance review before rollout — this isn't a typical ship-and-iterate feature. Pushing self-service too aggressively can look like avoiding accountability for a genuine error, so wrong-recipient and suspected-fraud cases should route to humans faster, not slower. Bank-side technical declines are partially outside PhonePe's control; the roadmap can monitor and route around them but can't fix another bank's infrastructure directly. Any change to refund or reversal timelines needs legal and compliance sign-off given RBI's turnaround-time guidelines.

## 13. Final Answers

**How many support agents would this workload require?** Using PhonePe's public transaction-volume data as an anchor and clearly labeled assumptions for contact rate, self-service rate, and handling time, the estimate lands around 2,500–2,600 agents dedicated to failed and pending-transaction support. This is a planning-level estimate, not PhonePe's real headcount — every assumption is stated so it can be corrected with actual internal data.

**What product outcomes should the roadmap target?** Lower contact rate per failed transaction, higher self-service resolution, faster time-to-reversal, fewer regulatory escalations, and a lower repeat-failure rate by bank corridor — while protecting CSAT and keeping human support easy to reach for genuinely complex cases.

## Key Learning

The easy answer to a growing support queue is to hire more agents. The better product question is why a real-time payment system leaves people waiting, in silence, when something goes wrong.

For a payments product specifically, the stakes are higher than most support case studies, because the product isn't just software — it's money that isn't in someone's account yet. Self-service here isn't about deflecting a query. It's about answering "where is my money" honestly and faster than a human ever could.

---

### Sources

PhonePe transaction volume and UPI market share — NPCI monthly data and industry reporting (Inc42, Business Standard, sci-tech-today.com, coinlaw.io), 2026. Business decline / technical decline classification and per-bank decline reporting — NPCI UPI Ecosystem Statistics; Outlook Money, 2026. Technical decline trend from 2016 to 2025 — D91 Labs analysis, 2026. Refund and reversal timelines — patterns drawn from publicly filed user complaints and PhonePe's own refund-process guidance, 2025–2026; typical auto-reversal cited at 1–3 business days, with disputed cases extending to 5–7 days or longer.

This is an independent product-thinking exercise built on public data, not affiliated with or based on internal PhonePe information.

---

**Nupur** — Business Analyst / APM candidate
