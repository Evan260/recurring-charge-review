# Report format

The on-screen report and the CSV carry the same records. The report is for reading, the CSV is for checking and filtering.

## On screen

Open with one line of scope, so the reader knows what was and was not looked at:

```
Read 412 documents in /volume1/documents/finance, 2023-10-30 to 2026-10-29.
388 readable, 24 could not be read.
```

Then the sections, in this order.

### 1. Recurring charges, by total paid

```
Vendor            Cycle      Times   First            Latest           Change     Total paid
Adobe             monthly    41      9.99  Mar 2023   22.99  Oct 2026  +130.1%    743.61
Google Workspace  monthly    36      14.40 Nov 2023   21.60  Oct 2026  +50.0%     604.80
Dropbox           yearly     3       119.88 Jan 2024  143.88 Jan 2026  +20.0%     383.64
```

Sorted by total paid, descending. Totals are net of credit notes. Every currency is shown as it was billed, and a vendor billing in two currencies appears twice.

### 2. Price increases worth noticing

Only steps of 15 percent or more between consecutive occurrences:

```
Adobe            Jan 2025   14.99 -> 19.99   +33.4%
Dropbox          Jan 2026  119.88 -> 143.88  +20.0%
```

### 3. Dormant charges

```
Vendor        Cycle     Last charged        Last amount   Cycles since
Audible       monthly   14 Feb 2026          16.95        8
```

Followed by one sentence, every time, without exception:

> These stopped appearing in the documents on the NAS. That means either the service
> ended, or it is still being billed and the paperwork is no longer arriving here.

### 4. Probable duplicates set aside

```
Adobe  22.99  on 02 Oct 2026
  kept    /volume1/documents/finance/2026/adobe-oct.pdf
  set aside /volume1/documents/inbox/scan_20261002_0001.pdf
```

Both paths every time, so the decision can be overruled.

### 5. Could not be read

```
/volume1/documents/finance/2024/scan_0912.pdf   no total found
/volume1/documents/inbox/IMG_4417.HEIC          no text recognised
```

Path and reason. Never an estimate.

## The CSV

One row per document record, including rows set aside as duplicates.

| Column | Meaning |
|---|---|
| `vendor_as_printed` | The vendor exactly as it appears on the document |
| `vendor_group` | The normalised name the record was grouped under |
| `date` | ISO 8601, `YYYY-MM-DD`, or `unknown` |
| `total` | Decimal, two places, negative for a credit note |
| `currency` | ISO 4217, or `unknown` |
| `kind` | invoice, receipt, statement, credit note, order confirmation |
| `period` | Billing period if stated, otherwise empty |
| `cycle` | monthly, weekly, fortnightly, quarterly, half-yearly, yearly, or empty |
| `is_duplicate` | `yes` or `no` |
| `duplicate_of` | The `source_file` of the record this duplicates, otherwise empty |
| `source_file` | Full path on the NAS |

One row per document rather than per vendor, for three reasons: every figure can be checked against the file it came from, a tax year is a spreadsheet filter on `date`, and a grouping the user disagrees with can be corrected by editing `vendor_group` without re-reading anything.

Encoding is UTF-8. Fields containing a comma or a quote are quoted. The header row is always present, even when there are no records.
