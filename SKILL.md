# Recurring Charge Review

## What this skill does

It reads the invoices, receipts and billing statements already stored on the NAS and answers a question no single document can answer: what is this household or business actually paying on a repeating basis, how has each of those charges changed over time, and what is worth doing about it.

It produces these things:

1. A list of recurring charges, each with its vendor, how many times it has been billed, the first and most recent amount, the change between them, and the total paid across all occurrences.
2. A list of charges that look dormant: a vendor that billed on a steady cycle and then stopped, which usually means either a cancelled service or a renewal whose paperwork stopped arriving.
3. The largest one-off purchases, so the report is not silent about the biggest number in the folder.
4. A short list of suggestions, drawn only from `references/alternatives.md`.
5. A single CSV of every document it read, so the same work can be filtered by tax year and handed to an accountant.

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

**If nobody answers, report and carry on.** This skill runs in conversations and in unattended ones. Where a step says to report something and wait, the unattended reading is: say it, then continue. The one thing that never happens without an answer is writing a file.

**The window is anchored at the day the review runs**, so three years means the three years ending today, not three calendar years and not three years back from the newest document.

**The window filters the analysis, not the reading.** Every document in the folder is read and gets a CSV row. The window decides which records count toward cycles, totals and dormancy.

**A near miss is included, not argued about.** A document that falls outside the window by 30 days or less is pulled back in when leaving it out would change a group's classification, for example by dropping it from recurring to two occurrences. Say that you did it and why. A four-day miss deciding whether a subscription exists is an artefact of the default, not a finding.

**A window that excludes documents has to say so.** Name how many were excluded and the date of the newest one. A default that changes the answer declares itself.

## How it works

### Step 1: Find candidate documents

Search the given folder for PDF, JPEG, PNG, HEIC, WebP and plain text files.

**Read each one however you can.** Most of these are images, so their text exists only once something has read them: your own vision, an OCR tool, or the text layer of a PDF. The tests below apply to the text you have read, not to bytes on disk.

A document is a candidate if its text contains at least one of: an amount with a currency symbol or ISO 4217 code, or one of the words invoice, receipt, statement, bill, subscription, renewal, order confirmation, payment, facture, rechnung, factura.

A file that is not a candidate is not silently dropped. If it is plainly not a financial document, say how many such files were skipped. If it looks like one but nothing can be read from it, it belongs in "could not be read" with its path and the reason.

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

**An unlabelled numeric date.** Thermal receipts print dates like `01/02/2025` with nothing to say which number is the month. Decide it in this order, and stop at the first one that answers: a value above 12 in either position settles it for that document; the same vendor's labelled documents settle it for the rest; the file name settles it, since a scanner stamps `scan_20250204` in an unambiguous order. If none of those answer, record the date as unknown rather than picking one, and say so in the report. Whichever convention you settle on, state it once in the report and apply it to every document in the run.

**Which amount to use.** Use the total payable, not the subtotal and not any single line. If the document shows tax separately, the total is the figure including tax. If a document shows an amount due of zero because it was already paid, use the amount charged, not the zero.

**Credit notes and refunds.** Record these with a negative total and `kind` of credit note. They are subtracted from a vendor's total later, never counted as another occurrence of the charge.

**Currency.** Read it from, in order of reliability: an explicit ISO 4217 code, the symbol together with the country in the address, the tax name, the language. An explicit code beats a tax name that disagrees with it: a document reading `GST` and `EUR 23.80` is recorded as EUR, and the disagreement is worth a line in the report. If the currency genuinely cannot be determined, record it as unknown rather than guessing. Never convert between currencies. A vendor billing in two currencies is reported as two separate recurring charges, because converting with today's rate would misstate what was actually paid.

Use the actual extracted records from this step for every step that follows. Do not re-read the files or work from memory of what they probably said.

### Step 3: Group the documents by payee

The same vendor is printed differently on every document, and card statements mangle it further. Normalise before grouping:

1. Lowercase the name.
2. Replace every character that is not a letter or a digit with a space.
3. Remove these noise words: pos, ach, debit, credit, card, purchase, payment, pmt, ref, reference, transaction, txn, visa, mastercard, amex, eft, online, pre, auth, authorized, withdrawal, deposit, transfer, the, and, inc, llc, ltd, gmbh, bv, sa, pty, limited, corp, corporation, co, company, subscription, renewal, monthly, annual, invoice, receipt, store.
4. Remove locale and marketplace words, which attach to many vendors and so group nothing: ca, com, net, org, io, uk, us, eu, de, mktp.
   - **A payment route is not the vendor.** When the name begins with paypal, sq or sp, drop that leading word and group on what follows, so `PAYPAL *SPOTIFY` becomes `spotify` and meets `Spotify AB`.
   - **A shortened brand is the brand.** Read amzn as amazon and gsuite as google before comparing, so `AMZN Mktp CA*2H4XY1` becomes `amazon` and meets `Amazon.ca`.
5. Remove any token containing a digit, which removes card trailers, store numbers and reference numbers.
6. Remove single-letter tokens.

Then group in two passes.

**Pass one, names that plainly match.** Two documents belong together when their normalised names share enough words: the number of words they have in common divided by the number of distinct words across both is 0.6 or higher, or one normalised name is wholly contained in the other. Compare a candidate against each existing member of a group, never against the group's combined words, because a group that pools its words drifts: once `adobe` and `adobe systems software` are together, the pool no longer contains `adobe creative cloud`.

**Pass two, names that share a word and behave the same.** A single shared leading word is not enough on its own, or `Microsoft 365` would swallow `Microsoft Azure`. Merge two groups that share their leading word only when their money agrees as well: either the amounts match within one percent, or the combined dates fall on one consistent cycle by the test in step 4. `google workspace` and `google gsuite argoro` merge on this rule. `shell` fuel receipts and `shell energy` bills do not, because neither their amounts nor their cycle line up.

`references/vendor-normalization.md` holds the worked examples, including the ones that must not merge.

Report the groups and their member counts before analysing them. A user who is there can correct a grouping, and a corrected grouping is used as given from then on.

### Step 4: Decide which groups are recurring

A vendor group is recurring when it has three or more records whose dates fall into a repeating cycle. Work out the cycle from the gaps between consecutive dates, taking the most common gap:

- 26 to 35 days apart: monthly
- 6 to 8 days: weekly
- 12 to 16 days: fortnightly
- 85 to 95 days: quarterly
- 175 to 190 days: half-yearly
- 350 to 380 days: yearly

**Missing paperwork is normal, and there is no limit on it.** A gap counts as a whole number of multiples of the cycle when it is within 7 days of a multiple, or within 10 percent of it, whichever is larger. A 200-day gap against a monthly cycle is 10 days short of 210, inside 10 percent, so it is six missing documents rather than a broken series. A series stays recurring as long as the most common gap is one cycle and every other gap is a multiple of it. Say in the report how many documents appear to be missing rather than claiming the charge stopped.

Two records are not enough to call something recurring. List those separately as "possible, only two occurrences" so the user can judge, and apply the dormancy test in step 7 to them as well.

A two-record group still needs a cycle for that test. Where its single gap falls in a band, that is the cycle. Where the gap is a whole multiple of a band instead, for example 60 days against monthly, name the base cycle and say a document appears to be missing between them.

### Step 5: Find duplicates before counting anything

The same bill is frequently saved twice: once from email and once scanned, often with different file names.

Within a vendor group, treat two records as the same underlying charge when their totals differ by no more than one percent, with a floor of one cent so that very small amounts are not held to an impossible standard, **and** their dates are within 5 days of each other.

Mark the second one as a probable duplicate. Exclude it from the totals and the cycle calculation, and list it in a separate duplicates section naming both files, so the user can see what was set aside and overrule it. Never delete a file and never describe a file as deleted.

A genuine repeat charge inside the duplicate window is possible, for example two identical same-day purchases. When the two documents carry **different invoice or order numbers**, treat them as separate charges rather than duplicates, and say so. A transaction reference on a card receipt is not an invoice number: `REF 987362` beside an invoice numbered `2026-38788` is the same charge seen twice, not two charges. An invoice and a receipt for the same amount within the window are one event, and the invoice is the record to keep. The kept record's date is the one that counts, including for the cycle.

### Step 6: Measure the change in price

For each recurring group, after duplicates are excluded:

- Report the earliest amount and its date, and the latest amount and its date.
- Report the change between them in both the currency amount and as a percentage.
- Report the total of every occurrence, with credit notes subtracted.
- Flag any single step up of 15 percent or more between consecutive occurrences, with the two dates and the two amounts, because that is the kind of increase people do not notice.
- Where a flagged step spans a gap with missing documents, say so. The rise may have arrived in stages that are not on the NAS.

### Step 7: Find charges that stopped

Measure age from the day the review runs, not from the newest document in the folder.

A group is dormant when its most recent occurrence is more than two full cycles old. Apply this to recurring groups and to two-occurrence groups alike: a subscription that billed twice and stopped is exactly what a cancellation looks like, and excluding it would hide the most useful finding in the report. Mark which of the two it came from.

Report the vendor, the last amount, the last date, and how many cycles have passed.

Say what this means in both directions, because the skill cannot tell which happened: either the service was cancelled, or it is still being billed and the paperwork has stopped arriving on the NAS. Do not state that money is still being spent.

### Step 8: Produce the report

Show the results on screen in this order:

1. **Recurring charges**, sorted by total paid descending.
2. **Price increases**, every step of 15 percent or more.
3. **Stopped appearing**, the dormant charges.
4. **Possible, only two occurrences**, where a pattern is forming but is not yet a pattern.
5. **Largest one-off purchases**, the five biggest documents whose vendor group holds exactly one record. Not the two-occurrence groups, which have their own section, and not credit notes, which have theirs. The biggest number in the folder should never be missing from the report because it only happened once.
6. **Credit notes with no matching charge**, each with its vendor, amount and file, so a refund never vanishes.
7. **Worth a look**, the suggestions from step 9.
8. **Probable duplicates set aside**, naming both files.
9. **Could not be read**, each with its path and the reason.

Open with the scope block, and put everything that is not one of the nine sections in it: the folder, the number of documents read, the span of the dates found in them (the dates of the documents, not the window searched), the date convention used, any window exclusions, and any signal that contradicted another, such as a tax name disagreeing with a currency code.

The step 1 and step 3 checkpoints go above that block, before the report proper. In an unattended run they are the record of what was found and how it was grouped, so they are kept rather than dropped.

Then ask whether to save the full record as a CSV. Save it only after the user agrees, and ask where to put it. Name the file `recurring-charge-review-<date>.csv` using the date the review was run.

The CSV has one row per document, including the ones excluded as duplicates and the ones that could not be read, with these columns:

`vendor_as_printed, vendor_group, date, total, currency, kind, period, cycle, is_duplicate, duplicate_of, source_file`

`vendor_group` is the shortest normalised name in the group, so one group has one label whatever its members print. A document that could not be read carries its path, a `kind` of unreadable, and empty fields elsewhere: the CSV has a row for every file, so its row count matches the number of documents read.

One row per document, never one row per group, so that the figures can be checked against the original files and filtered by tax year in a spreadsheet.

### Step 9: Suggest what to do about it

Read `references/alternatives.md` and follow it. In short: suggest only from its table, never state a price for an alternative, one suggestion per vendor, nothing at all for a vendor you cannot categorise, and keep suggestions in their own section after everything measured.

The suggestion worth making first, whenever it applies, is a cloud storage subscription on a machine whose entire purpose is storage.

## When something is missing or unclear

Say so and stop rather than filling the gap:

- No candidate documents: name the folder searched and the file types looked for, and stop. Do not search elsewhere without being asked.
- A document that cannot be read, or whose total cannot be found: list it under "could not be read" with its path and the reason. Never guess an amount.
- A date that cannot be determined: keep the record, mark the date unknown, and exclude it from cycle detection while still counting it in the vendor total.
- No recurring pattern anywhere: report that none could be established yet and show the one-off purchases and two-occurrence groups, rather than presenting a single purchase as a subscription.
- A signal that contradicts another, such as a tax name that disagrees with a currency code: record the more reliable one, and say in the report that they disagreed.

Never present an estimate as a reading. Every figure in the report comes from a document, and every figure can be traced to the file it came from.

## What needs confirmation

Ask first, and wait for an answer, before:

- Saving the CSV or any other file, including where it goes.
- Searching any folder other than the one the user named.
- Treating a grouping the user has corrected as anything other than final.

With nobody there to answer, state the question and carry on with the report. Nothing is written to disk in that case.

The skill never deletes, moves, renames or overwrites a file, and never sends or shares anything outside the NAS. There is no situation in which it should offer to.

## Reference material

- `references/vendor-normalization.md`: worked examples of names that group and names that must not.
- `references/document-rules.md`: which date and which total to take from each kind of document, with the awkward cases.
- `references/report-format.md`: the exact on-screen layout and CSV column meanings.
- `references/alternatives.md`: the only list suggestions may come from, and the rules for making one.
