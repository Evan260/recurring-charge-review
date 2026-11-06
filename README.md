# Recurring Charge Review

Ask your NAS what you are actually paying for.

If you have a NAS, you probably have years of invoices, receipts and statements on it: some downloaded from email, some scanned, some dropped into a folder called `inbox` and never looked at again. Any one of those documents tells you almost nothing. All of them together tell you what your life costs.

This skill reads them and answers three questions:

- **What is billing me on a repeating basis?** Every vendor that charges on a cycle, how often, and how much you have paid them in total.
- **What has quietly gone up?** The first price, the current price, and any single increase big enough to have been worth noticing at the time.
- **What stopped?** Charges that billed like clockwork and then went quiet, which is either a service you cancelled or one whose paperwork stopped arriving.

It then offers to save a CSV with one row per document, so you can filter it by tax year and hand it to whoever does your books.

## How to use it

Ask in plain language:

> What subscriptions am I paying for?

> Find recurring charges in /volume1/documents/finance

> Has anything I pay for gone up in price in the last three years?

> Pull together my receipts for the 2026 tax year

It will ask which folder to read and how far back to go, then tell you how many documents it found before it starts.

## What it will and will not do

It reads documents and, if you say yes, writes one CSV where you choose.

It does not move, rename, delete or overwrite anything. It does not send your documents anywhere or share them. It asks before saving a file and asks before looking in a folder you did not name.

Nothing leaves the NAS.

## What it needs

Documents in PDF, JPEG, PNG, HEIC, WebP or plain text, in one folder or in subfolders under it. They do not need tidy names, consistent formats, or to be sorted. Scans, email downloads and photos of paper receipts all work.

Three years is the default window, because a yearly subscription needs three occurrences before a trend exists.

## What it is careful about

A review like this is worthless if the numbers are wrong, so the skill refuses in all the places where guessing would be easy:

- It uses the invoice date, not the due date and not the date the file was saved, so a 2021 invoice copied onto the NAS last week is not counted as this year's.
- It never converts currencies. A vendor billing in two currencies is reported twice, because today's exchange rate is not what you paid.
- It sets aside duplicates, the same bill saved once from email and once scanned, and it shows you both file paths so you can overrule it.
- When a document cannot be read, it lists the file and the reason rather than estimating an amount.
- When nothing is found, it says so and stops rather than offering something similar.

## Where the rules come from

The grouping, duplicate and document-reading rules in `references/` come from building accounting software that does this work for small businesses, where getting a vendor name or an invoice date wrong is not a cosmetic problem.
