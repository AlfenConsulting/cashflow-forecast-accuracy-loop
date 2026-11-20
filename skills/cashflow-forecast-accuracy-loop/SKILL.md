---
name: cashflow-forecast-accuracy-loop
description: "Build a 13-week rolling cashflow forecast from accounting actuals plus confirmed assumptions, then check every earlier forecast against what actually happened and report accuracy against a target. Use when asked for a cash position, runway or cashflow forecast, to flex a scenario (extra sold day, late invoice, lost client), or to run the weekly forecast accuracy check, including from a scheduled task."
---

> **Not advice.** This skill is general information, not tax, VAT, accounting or legal advice. It contains no rates or rules: it asks you for yours, and the forecast is only as reliable as your answers and your data.

# Cashflow forecast and accuracy loop

A 13-week rolling forecast at weekly granularity, built from the accounting system's actuals and the owner's confirmed assumptions, plus a weekly loop that scores each past forecast against reality. The loop is the point: a forecast nobody measures never gets better.

Two modes share one method.

- **Interactive.** Someone asks for cash position, runway or a scenario. Ask a short set of questions, build the table, flag risks.
- **Unattended.** A scheduled task runs the loop every Monday with nobody to answer questions. Use stated defaults, say plainly what was assumed, and leave open questions for the owner.

## Inputs this skill expects

Replace these with whatever your setup offers. The method does not depend on a particular accounting product.

- Read access to the accounting system: bank and card balances, open sales invoices, open supplier invoices, VAT position, payroll runs.
- A place to keep a forecast log (one fenced JSON snapshot per run, append only) and a confirmed assumptions note.
- A target accuracy (default 90 percent).

## Step 0: Confirm which company the numbers come from

Before pulling any figure, confirm that every call resolves to the company you mean to forecast. Read the company name from the tool result or from a quick period or chart-of-accounts read. If more than one accounting connection is live and it is not obvious which is the right one, stop and ask (interactive) or run on the unambiguous one and record which was used (unattended). Never mix two entities, and never use a demo or test tenant.

## Step 1: Read the confirmed assumptions and the log

- Read the assumptions note. It holds forward revenue, billing cadence, payment timing, tax and payroll conventions the owner has already confirmed. Apply them instead of asking again.
- Read the forecast log so this run can be compared with earlier ones.
- If the note lacks any answer from the setup questions below, ask them (interactive) or carry them as open (unattended).
- If the owner changes an assumption in conversation, save it to the assumptions note so the unattended loop picks it up too, and say that you did.

## Step 2: Pull the baseline from the accounting system

Do not ask the owner for numbers the system already has.

- **Starting cash.** Use the bank's own balance, not the ledger balance: the ledger lags the bank feed until lines are reconciled. Count bank accounts and company cards together, and treat card top-ups as internal transfers, not costs.
- **Receivables.** Open sales invoices with amount and due date, placed in the week they are expected to be paid under the confirmed payment-timing assumption.
- **Payables and recurring costs.** Open supplier invoices and standing outgoings with due dates. Use the real invoice amount once it exists, not the placeholder.
- **VAT.** Use the filing cadence the company actually has. Take the booked figure for the period and its real due date.
- **Payroll and payroll taxes.** Use booked or prepared payroll runs for amounts, and the cash dates from the setup answers. The date cash leaves the account can be earlier than the stated payout date, and withheld amounts are often paid to the authorities on a later date than the salary.
- **Month-end items.** Apply accruals that settle as cash on the last day of the month, not the first.

Watch for data that looks like test data (the company invoicing itself, demo project names). Exclude it by default, list it in the output and in the log, and escalate if it persists over several runs.

If a tool call fails or a figure looks stale, say so plainly instead of estimating silently.

## Step 3: Build the 13-week forecast

Use Monday-morning points: each point is the cash balance at the start of that Monday, which equals the close of the previous week. The run date itself is the starting point and is the actual balance that day.

One row per week: opening balance, inflows by source, outflows by type (payroll, payroll taxes, VAT, supplier costs, subscriptions, other), net movement, closing balance. Show the window's lowest point and the date it begins.

Unattended defaults:

- Project no new revenue beyond what is booked or confirmed in the assumptions note. A new client or new sold days with no booking and no confirmation needs the owner's input.
- Exclude any anomaly from the baseline and list it explicitly.
- If something needed (a VAT figure, a payroll run) is not booked yet, carry forward the previous snapshot's assumption and flag it as still open.
- Never invent days worked, rates, amounts or tax details. State the assumption, show how it was derived, and flag what is unverified.

## Step 4: Check earlier forecasts against actuals

For every point in earlier snapshots that has now passed, compare the forecast with the actual combined cash on that date.

```
accuracy = min(forecast, actual) / max(forecast, actual)
```

100 percent is perfect. The formula is symmetric and handles a zero or tiny actual gracefully.

Report by horizon:

- **Week 1** is the first forecast Monday after the run date. The starting balance is the actual and does not count as a measurement.
- **Week 4** (four weeks ahead) is the headline number, tracked against the target.
- Longer horizons once enough history exists. Show the number of measurements behind every average.

Be honest about what the numbers can and cannot say:

- A forecast recomputed after the fact with newer knowledge is pro forma. Mark it as such and do not count it as evidence of accuracy.
- A week in which nothing material was due tests nothing. A high score there is flattering, not proof.
- Check whether the log's dates mean the same thing across runs (end-of-week closing versus Monday-morning points). A one-week shift can make a miss look like a hit, or the reverse.

## Step 5: Rank what to refine

For every comparison below about 85 percent, find the driver and bucket it: timing slippage on a named invoice or counterparty, tax amount or date wrong, standing cost off, unanticipated new revenue or cost, or an anomaly that resolved differently than assumed. Roll the buckets into a ranked list, most frequent or most material first, and lead the report with it.

When a payment-timing assumption is tested, record the real gap between predicted and actual payment date. If the data keeps contradicting an assumption, say so and recommend updating it. Never edit the confirmed assumptions note without the owner's agreement; flag the mismatch instead.

## Step 6: Save

Append this run's snapshot to the log (never overwrite an earlier entry): snapshot date, assumptions used, anomalies flagged, which company and connection were confirmed, and all forecast points.

```json
{
  "snapshot_date": "YYYY-MM-DD",
  "points_are": "Monday morning, combined cash",
  "assumptions": { "notes": "..." },
  "anomalies_flagged": [],
  "weeks": [ ["YYYY-MM-DD", 0] ]
}
```

## Interactive extras

- Ask forward questions in one short batch: expected new invoicing (who, how many days, what rate), one-off costs or distributions, known slippage. Skip anything already in the assumptions note.
- After the table, call out in prose: any week below a safety buffer (default one month of fixed costs, and say that is the threshold used), the largest outflow week and why, and the assumption carrying most of the risk.
- Offer scenario flexing: add or remove a sold day, shift an invoice's payment date, drop a client, add a cost. Show the change against the baseline closing balance instead of repeating the whole table.

## Output

In an unattended run with any open assumption, start the output with a section headed "Open assumptions: this forecast is not final" that lists every open item and what it affects. Put nothing above it.

Then lead with the headline: the week-4 accuracy against the target and the single biggest driver of any miss, then the lowest cash point with its date. Follow with the ranked refinement list and the table or chart. State every unattended assumption made this run in plain language, because nobody was there to confirm it.

Formatting: use the number format of the company's market, lead with the number that matters, and keep it short.

Tax, VAT and payroll rules come only from the user. Use the rules recorded in the assumptions note. Never state a rate, threshold or deadline from your own knowledge as fact; ask the setup questions instead. Say that the forecast rests on the user's own inputs, and never present a calculated figure as final.

## Setup questions

Ask these on the first run, or whenever the assumptions note lacks an answer. Save each answer to the assumptions note so later runs, including unattended ones, use it. In unattended mode, do not guess: carry the item as open and list it in the output.

1. Currency and number format.
2. VAT: how often does the company file, and on which date is each period paid?
3. Payroll: which day is salary paid, and does cash leave the bank on that day or earlier?
4. Payroll taxes and contributions: which amounts are withheld or paid on top of salary, and on which dates are they paid to the authorities?
5. Other recurring statutory payments (for example pension or social contributions billed monthly or quarterly): amount and due date.
6. Holiday pay: does anyone on the payroll accrue holiday pay that will later leave as cash? If yes, how and when is it paid?
7. Corporation tax and any on-account payments: dates and amounts, or should they be placeholders for now?
8. Anything else that moves cash on a fixed schedule.

If the user does not know an answer, record it as an open assumption, show it in the output, and suggest they check it with their accountant or the tax authority. The skill keeps working with the open item marked.

## Limits

- Every number is AI-produced and not final. The owner reviews before anything is used externally.
- Tax, VAT, payroll and insurance rules are supplied by the user. The skill contains no rates or rules of its own and does not give advice.
- The skill only reads. It does not book, approve, send or pay anything.
