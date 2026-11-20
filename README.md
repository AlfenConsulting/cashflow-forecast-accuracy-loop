# Cashflow forecast and accuracy loop

A Claude skill that builds a 13-week rolling cashflow forecast from your accounting actuals and confirmed assumptions, then scores every earlier forecast against what actually happened.

Most small companies forecast once and never check. This skill makes the check a weekly habit: it compares each past forecast with the real balance, reports accuracy against a target, and ranks what to fix first.

## What it does

- Builds a 13-week forecast with Monday-morning cash points and shows the lowest point and its date.
- Compares each past forecast with the actual combined bank and card balance using `min(forecast, actual) / max(forecast, actual)`.
- Reports week 1 and week 4 accuracy against a target (default 90 percent), with the number of measurements behind every average.
- Separates real evidence from pro forma recalculations and from weeks where nothing was due.
- Ranks the drivers of each miss: payment timing, tax amount or date, standing costs, new items, anomalies.
- Runs interactively or as an unattended weekly scheduled task.

## What you need

- Read access to your accounting system through a connector or tools of your choice. The method is system-independent.
- Two files or documents you choose, one for the forecast log and one for your confirmed assumptions. The skill uses only the places you name. It does not use Claude's memory or chat history.

## Install

This repository is a Claude plugin with one skill, in `skills/cashflow-forecast-accuracy-loop/SKILL.md`.

- Claude Code: `/plugin install cashflow-forecast-accuracy-loop --marketplace AlfenConsulting/cashflow-forecast-accuracy-loop`
- Claude apps: zip the `skills/cashflow-forecast-accuracy-loop` folder and upload it where your Claude app accepts skills.

The skill sends nothing anywhere and fetches nothing from the web. It only reads the accounting data and notes you give it access to.

## Adapt it

The skill holds no country-specific rates or rules. On the first run it asks a short set of setup questions (VAT cadence, payroll dates, statutory contributions, holiday pay, corporation tax) and saves your answers as confirmed assumptions. Change an answer any time and later runs pick it up.

## Examples

All figures below are invented.

1. First run, no data yet:
   "I run a small consulting company. Give me a 13-week cashflow forecast."
   The skill asks where to keep the log and the assumptions note, then asks
   the setup questions before it builds anything.

2. Forecast from your own figures:
   "Bank balance 180.000 DKK. Open invoices: Client A 60.000 DKK due
   4 December, Client B 45.000 DKK due 18 December. Rent 12.000 DKK on the
   1st, salary 40.000 DKK on the last banking day of the month. Holiday pay:
   I don't know."
   You get a 13-week table with Monday points, the lowest point and its date,
   and holiday pay listed as an open assumption.

3. Weekly accuracy check:
   "It's Monday. Actual balance is 171.500 DKK. Score last week's forecast."
   You get the accuracy for each horizon with the number of measurements
   behind it, and the likely driver of the gap.

## Limits

The skill only reads data. It does not book, approve, send or pay anything. Every figure is AI-produced and needs a human review. Tax, VAT and payroll rules come from you, not from the skill.

## Privacy

This plugin contains instructions only. It runs no code, has no server and
collects nothing. Alfen Consulting ApS receives no data from it.

When you use it, Claude reads the accounting data and notes you give it
access to, such as balances, invoices and payroll runs. These can include
personal data about employees, customers or suppliers. You decide what Claude
gets access to, and you remain responsible for that data under your own data
protection obligations.

If you connect an accounting system, data passes between Claude and that
system under that provider's terms. The forecast log and the assumptions note
are stored where you choose to keep them. The plugin itself sends nothing to
any other service and fetches nothing from the web.

How Claude handles your data is governed by your agreement with Anthropic.
Questions about this plugin: finance@alfenconsulting.dk

## Not advice

This skill is general information and does not constitute tax, VAT, accounting
or legal advice. It contains no rates or rules of its own: it asks you for
yours, and the forecast is only as reliable as your answers and your data. If
you are unsure of
an answer, check it with your accountant or the tax authority.

## Security

See `SECURITY.md` to report a problem with this plugin.

## Licence

The code and text are published under the MIT licence (see `LICENSE`).
The licence does not grant any right to use the name "Alfen Consulting",
the Alfen logo or other Alfen trademarks. Modified versions must not be
presented as made, reviewed or endorsed by Alfen Consulting ApS.

Published by Alfen Consulting ApS, a finance and executive consulting practice in Denmark. Alfen runs this loop weekly on its own cash.
