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
- A forecast log and a short note of confirmed assumptions, kept anywhere Claude can read and write.

## Install

Copy the `cashflow-forecast-accuracy-loop` folder into your Claude skills directory, or upload `SKILL.md` where your Claude app accepts skills.

## Adapt it

The skill holds no country-specific rates or rules. On the first run it asks a short set of setup questions (VAT cadence, payroll dates, statutory contributions, holiday pay, corporation tax) and saves your answers as confirmed assumptions. Change an answer any time and later runs pick it up.

## Limits

The skill only reads data. It does not book, approve, send or pay anything. Every figure is AI-produced and needs a human review. Tax, VAT and payroll rules come from you, not from the skill.

## Not advice

This skill is general information and does not constitute tax, VAT, accounting
or legal advice. It contains no rates or rules of its own: it asks you for
yours, and the forecast is only as reliable as your answers and your data. If
you are unsure of
an answer, check it with your accountant or the tax authority.

## Licence

The code and text are published under the MIT licence (see `LICENSE`).
The licence does not grant any right to use the name "Alfen Consulting",
the Alfen logo or other Alfen trademarks. Modified versions must not be
presented as made, reviewed or endorsed by Alfen Consulting ApS.

Published by Alfen Consulting ApS, a finance and executive consulting practice in Denmark. Alfen runs this loop weekly on its own cash.
