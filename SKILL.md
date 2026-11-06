# Recurring Charge Review

## What this skill does

It reads the invoices, receipts and billing statements already stored on the NAS and answers a question no single document can answer: what is this household or business actually paying on a repeating basis, and how has each of those charges changed over time.

It produces three things:

1. A list of recurring charges, each with its vendor, how many times it has been billed, the first and most recent amount, the change between them, and the total paid across all occurrences.
2. A list of charges that look dormant: a vendor that billed on a steady cycle and then stopped, which usually means either a cancelled service or a renewal that failed quietly.
3. A single CSV of every document it read, so the same work can be filtered by tax year and handed to an accountant.

The skill reads documents and writes one report. It never moves, renames, deletes or sends anything.

## When to use it

Use it when someone asks any of these, or anything close to them:

- "What subscriptions am I paying for?"
- "Find my recurring charges."
- "Has anything I pay for gone up in price?"
- "What am I still being billed for that I don't use?"
- "Pull together my receipts for the 2026 tax year."
- "How much have I spent with this vendor?"

Do not use it to answer a question about one specific document. Reading a single invoice is a plain document question and does not need this skill.

## What it needs before it starts

Ask for these two things if they were not given:

- **Where to look.** A folder on the NAS. Subfolders are included.
- **How far back.** A number of years, or "everything". If the user has no preference, use the last three years, because a yearly billing cycle needs at least three occurrences before a trend means anything.

Do not ask anything else up front. Everything else is inferred from the documents.

## How it works

### Step 1: Find candidate documents

Search the given folder for PDF, JPEG, PNG, HEIC, WebP and plain text files.

A file is a candidate if its text contains at least one of: an amount with a currency symbol or ISO 4217 code, or one of the words invoice, receipt, statement, bill, subscription, renewal, order confirmation, payment, facture, rechnung, factura.

Report how many files were found and how many are candidates before going further. If there are no candidates, say so plainly, name the folder that was searched, and stop. Do not widen the search to unrelated files to produce a result.

### Step 2: Extract one record per document

From each candidate, extract exactly these fields:

- `vendor`: the name of the party being paid, as printed.
- `date`: see the date rules below.
- `total`: the final amount payable, after tax and after any discount.
- `currency`: a three-letter ISO 4217 code.
- `kind`: one of invoice, receipt, statement, credit note, order confirmation.
- `period`: a billing period if the document states one, for example "1 Mar 2026 to 31 Mar 2026".
- `source_file`: the full path of the file this came from.

**Which date to use.** Use the invoice or issue date, not the due date, not the printed date, not the date the file was saved. A statement covering a period is dated by the period end. If a document shows no usable date, record the date as unknown and keep the record. Do not substitute the file's modified time, because a file copied onto the NAS carries the copy date and would place a 2021 invoice in the current year.

**Which amount to use.** Use the total payable, not the subtotal and not any single line. If the document shows tax separately, the total is the figure including tax. If a document shows an amount due of zero because it was already paid, use the amount charged, not the zero.

**Credit notes and refunds.** Record these with a negative total and `kind` of credit note. They are subtracted from a vendor's total later, never counted as another occurrence of the charge.

**Currency.** Read it from the symbol, the ISO code, the address, the language, or the tax name. GST or HST means CAD, VAT means EUR or GBP, and so on. If the currency genuinely cannot be determined, record it as unknown rather than guessing. Never convert between currencies. A vendor billing in two currencies is reported as two separate recurring charges, because converting with today's rate would misstate what was actually paid.

Use the actual extracted records from this step for every step that follows. Do not re-read the files or work from memory of what they probably said.

### Step 3: Normalise the vendor name

The same vendor is printed differently on every document, and card statements mangle it further. Normalise before grouping:

1. Lowercase the name.
2. Replace every character that is not a letter or a digit with a space.
3. Remove these noise words: pos, ach, debit, credit, card, purchase, payment, pmt, ref, reference, transaction, txn, visa, mastercard, amex, eft, online, pre, auth, authorized, withdrawal, deposit, transfer, the, and, inc, llc, ltd, gmbh, bv, sa, pty, limited, corp, corporation, co, company, subscription, renewal, monthly, annual, invoice, receipt.
4. Remove any token that is only digits, which removes card trailers, store numbers and reference numbers.
5. Remove single-letter tokens.

Two documents belong to the same vendor when their normalised names share enough words. Measure that with token-set similarity: divide the number of words the two names have in common by the number of distinct words across both. Group them when that value is 0.6 or higher, or when one normalised name is wholly contained in the other.

Group with care across these cases, which are the ones that go wrong:

- A reseller is not the vendor. "Apple" billing for an app subscription and "Apple" billing for hardware are the same payee but not the same recurring charge, so split them when the amounts form two clearly separate patterns.
- A marketplace prefix ("amzn mktp", "sq", "sp", "paypal") is a payment route, not a vendor. Strip a leading route token and group on what follows.
- Two genuinely different vendors can normalise to similar names. If grouping two names would merge charges whose amounts never agree and whose cycles do not line up, keep them apart and say why in the report.

Report the groups and their member counts before analysing them. The user can correct a grouping, and a corrected grouping is used as given from then on.

### Step 4: Decide which groups are recurring

A vendor group is recurring when it has three or more records whose dates fall into a repeating cycle. Work out the cycle from the gaps between consecutive dates:

- 26 to 35 days apart: monthly
- 6 to 8 days: weekly
- 12 to 16 days: fortnightly
- 85 to 95 days: quarterly
- 175 to 190 days: half-yearly
- 350 to 380 days: yearly

Allow one gap in the series to be a whole number of multiples of the cycle, which covers a missing document rather than a missed payment. Say in the report that the document appears to be missing rather than claiming the charge stopped.

Two records are not enough to call something recurring. List those separately as "possible, only two occurrences" so the user can judge.

### Step 5: Find duplicates before counting anything

The same bill is frequently saved twice: once from email and once scanned, often with different file names.

Within a vendor group, treat two records as the same underlying charge when their totals are equal, or differ by one cent or by one percent, whichever is larger, **and** their dates are within 5 days of each other.

Mark the second one as a probable duplicate. Exclude it from the totals and the cycle calculation, and list it in a separate duplicates section naming both files, so the user can see what was set aside and overrule it. Never delete a file and never describe a file as deleted.

A genuine repeat charge inside the duplicate window is possible, for example two identical same-day purchases. When the two documents have different invoice or order numbers, treat them as separate charges rather than duplicates, and say so.

### Step 6: Measure the change in price

For each recurring group, after duplicates are excluded:

- Report the earliest amount and its date, and the latest amount and its date.
- Report the change between them in both the currency amount and as a percentage.
- Report the total of every occurrence, with credit notes subtracted.
- Flag any single step up of 15 percent or more between consecutive occurrences, with the two dates and the two amounts, because that is the kind of increase people do not notice.

### Step 7: Find dormant charges

A recurring group is dormant when the most recent occurrence is more than two full cycles old. Report the vendor, the last amount, the last date, and how many cycles have passed.

Say what this means in both directions, because the skill cannot tell which happened: either the service was cancelled, or it is still being billed and the paperwork has stopped arriving on the NAS. Do not state that money is still being spent.

### Step 8: Produce the report

Show the results on screen in this order: recurring charges sorted by total paid descending, then price increases, then dormant charges, then possible duplicates, then documents that could not be read.

Then ask whether to save the full record as a CSV. Save it only after the user agrees, and ask where to put it. Name the file `recurring-charge-review-<date>.csv` using the date the review was run.

The CSV has one row per document record, including the ones excluded as duplicates, with these columns:

`vendor_as_printed, vendor_group, date, total, currency, kind, period, cycle, is_duplicate, duplicate_of, source_file`

One row per document, never one row per group, so that the figures can be checked against the original files and filtered by tax year in a spreadsheet.

## When something is missing or unclear

Say so and stop rather than filling the gap:

- No candidate documents: name the folder searched and the file types looked for, and stop. Do not search elsewhere without being asked.
- A document that cannot be read, or whose total cannot be found: list it under "could not read" with its path and the reason. Never guess an amount.
- A date that cannot be determined: keep the record, mark the date unknown, and exclude it from cycle detection while still counting it in the vendor total.
- Fewer than three records for every vendor: report that no recurring pattern could be established yet and show what was found, rather than presenting a single purchase as a subscription.

Never present an estimate as a reading. Every figure in the report comes from a document, and every figure can be traced to the file it came from.

## What needs confirmation

Ask first, and wait for an answer, before:

- Saving the CSV or any other file, including where it goes.
- Searching any folder other than the one the user named.
- Treating a grouping the user has corrected as anything other than final.

The skill never deletes, moves, renames or overwrites a file, and never sends or shares anything outside the NAS. There is no situation in which it should offer to.

## Reference material

- `references/vendor-normalization.md`: worked examples of names that group and names that must not.
- `references/document-rules.md`: which date and which total to take from each kind of document, with the awkward cases.
- `references/report-format.md`: the exact on-screen layout and CSV column meanings.
