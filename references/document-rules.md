# Which date, which total, which document

Every figure in the report traces back to one document. These are the rules for reading that document correctly, and the cases where the obvious reading is wrong.

## The kinds of document, and what each one is evidence of

| Kind | What it proves | How it is counted |
|---|---|---|
| Invoice | An amount was billed on a date | One occurrence |
| Receipt | An amount was paid | One occurrence |
| Statement | A period's activity, often many charges | See "statements" below |
| Credit note | An amount was given back | Negative, never an occurrence |
| Order confirmation | An order was placed, not that it was billed | Only when no invoice or receipt exists for the same amount and date |

An invoice and a receipt for the same charge are the same event seen twice. They are caught by the duplicate rule in `SKILL.md` step 5, not counted as two months of a subscription.

## Dates

Use the invoice date or issue date.

- **Not the due date.** A monthly invoice issued on the 2nd and due on the 30th would otherwise drift a whole cycle and break the gap detection.
- **Not the printed or saved date.** A file copied onto the NAS in 2026 can hold a 2021 invoice. A PDF's own creation metadata is the date it was generated, which for a re-downloaded invoice is today.
- **A statement is dated by the end of its period**, so a statement covering 1 March to 31 March is dated 31 March.
- **Two dates and no labels:** take the earlier one. An invoice is issued before it is due.
- **No usable date:** keep the record with the date marked unknown. It still counts toward the vendor's total, and it is left out of cycle detection.

## Totals

Use the final amount payable, after tax and after discount.

- **Not the subtotal.** The subtotal excludes tax, and a price rise in a jurisdiction that changed its tax rate would be misread.
- **Not a single line item.** A three-line invoice has one total.
- **"Amount due: 0.00" after "Paid in full":** the charge is the amount billed, not the zero. The zero means it has already been settled.
- **A statement's "minimum payment" is not the total.** The charge is each line in the period, or the closing balance if the lines cannot be read.
- **Tips and surcharges are part of the total** when they appear above the final line.
- **Rounding:** record what the document says, to the cent. Never round for tidiness.

## Statements with many charges

A card or bank statement holds many vendors. Where the individual lines can be read, extract one record per line, each with its own vendor, date and amount, and keep the statement's path as the source for all of them.

Where the lines cannot be read, record one statement-level entry and say in the report that the statement's detail could not be read. Do not spread the closing balance across guessed vendors.

A statement line and a separate invoice for the same charge are a duplicate pair. The invoice is the better record, because it names the vendor properly and shows the tax. Keep the invoice and set the statement line aside.

## Currency

Read the currency from, in order of reliability: an explicit ISO 4217 code, the currency symbol together with the country in the address, the tax name, the language.

Tax names are the most reliable signal after an explicit code:

| Tax named on the document | Currency it implies |
|---|---|
| GST, HST, PST, QST | CAD |
| VAT with a European address | EUR or GBP, decided by the symbol and country |
| Sales tax with a US state | USD |
| GST with an Australian address | AUD |

A bare `$` is ambiguous between at least CAD, USD, AUD and several others. Decide it from the address, and if there is no address, record the currency as unknown.

**Never convert between currencies.** The report states what was actually paid. A vendor billing in two currencies becomes two recurring charges, each in its own currency, with a note that they appear to be the same service.

## Partial payments and instalments

Three payments of 100 against one invoice for 300 are one charge, not three occurrences. Where the documents name the same invoice number, group them, count one occurrence and total 300.

Where there is no shared reference, they will look like a three-month recurring charge of 100. Report the pattern, note that the amounts sum exactly to a single invoice in the same group, and let the user decide.

## Annual charges

A yearly subscription produces one document a year, so three years of history are needed before it is recurring at all. Until then it belongs in the "possible, only two occurrences" section.

This is the single most common reason a real subscription is missed, which is why the default window is three years rather than one.
