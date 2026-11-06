# Report format

The on-screen report and the CSV carry the same records. The report is for reading, the CSV is for checking and filtering.

## On screen

Open with one line of scope. The date range is the span of the dates found in the documents, not the window that was searched:

```
Read 412 documents in /volume1/documents/finance, dated 2023-10-30 to 2026-10-29.
388 readable, 24 could not be read.
Dates on the thermal receipts read as day/month/year.
```

If a window excluded documents that are sitting in the folder, add a line saying how many and how recent:

```
The three-year window left out 14 documents, the newest dated 2023-10-28.
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

Where documents are missing from a series, say so under the table rather than in it:

```
Adobe: 4 documents appear to be missing. Every gap is a whole number of months.
```

### 2. Price increases worth noticing

Every rise between consecutive occurrences, largest first. Rises of 15 percent or more are marked, which is emphasis and not a filter:

```
Adobe            Jan 2025   14.99 -> 19.99   +33.4%   *
Dropbox          Jan 2026  119.88 -> 143.88  +20.0%   *
Hydro One        Mar 2026   96.40 ->  99.95   +3.7%
Fastmail         Jan 2026   59.00 ->  61.00   +3.4%
```

Rounding, a tax rate change and a part-month first invoice all move an amount without the price moving, so anything under one percent is left out.

### 3. Stopped appearing

```
Vendor        Cycle     Last charged        Last amount   Cycles since
Audible       monthly   14 Feb 2026          16.95        8
Hetzner       monthly   04 Feb 2025          23.80        21     (from two occurrences)
```

Followed by one sentence, every time, without exception:

> These stopped appearing in the documents on the NAS. That means either the service ended, or it is still being billed and the paperwork is no longer arriving here.

### 4. Possible, only two occurrences

```
Vendor        Amounts              Dates                          Apart
Fastmail      59.00, 59.00         12 Mar 2026, 12 Apr 2026       31 days
```

A pattern that is forming. Two records are not a cycle, so these are not counted as recurring and their totals are not in section 1.

### 5. Largest one-off purchases

```
WestJet Airlines    641.30   02 Jun 2026   receipt
Apple Store       1,899.00   03 Aug 2025   receipt
```

The five biggest documents belonging to no recurring group. Without this section the largest number in the folder can be missing from the report because it only happened once.

### 6. Credit notes with no matching charge

```
Staples 212 Saskatoon   -41.10   28 May 2026   2026/staples-credit-note.png
```

A refund with nothing to subtract it from still gets named, so the money does not vanish from the report.

### 7. Worth a look

The suggestions, made under the rules in `alternatives.md`. Three sentences each at most, no prices, one per vendor.

### 8. Probable duplicates set aside

```
Adobe  22.99  on 02 Oct 2026
  kept    /volume1/documents/finance/2026/adobe-oct.pdf
  set aside /volume1/documents/inbox/scan_20261002_0001.pdf
```

Both paths every time, so the decision can be overruled.

### 9. Could not be read

```
/volume1/documents/finance/2024/scan_1183.pdf   total unreadable, the figure is cut off
/volume1/documents/inbox/IMG_4417.HEIC          no text recognised
```

Path and reason. Never an estimate. A file that was plainly not a financial document at all is not listed here; it is counted in one line with the scope ("9 files skipped, not financial documents").

## The CSV

One row per document, including rows set aside as duplicates and rows that could not be read. The row count matches the number of documents read.

| Column | Meaning |
|---|---|
| `vendor_as_printed` | The vendor exactly as it appears on the document, empty when nothing could be read |
| `vendor_group` | The shortest normalised name in the group, so one group carries one label |
| `date` | ISO 8601, `YYYY-MM-DD`, or `unknown` |
| `total` | Decimal, two places, negative for a credit note, empty when nothing could be read |
| `currency` | ISO 4217, or `unknown` |
| `kind` | invoice, receipt, statement, credit note, order confirmation, or unreadable |
| `period` | Billing period if stated, otherwise empty |
| `cycle` | monthly, weekly, fortnightly, quarterly, half-yearly, yearly, or empty |
| `is_duplicate` | `yes` or `no` |
| `duplicate_of` | The `source_file` of the record this duplicates, otherwise empty |
| `source_file` | Full path on the NAS |

One row per document rather than per vendor, for three reasons: every figure can be checked against the file it came from, a tax year is a spreadsheet filter on `date`, and a grouping the user disagrees with can be corrected by editing `vendor_group` without re-reading anything.

Encoding is UTF-8. Fields containing a comma or a quote are quoted. The header row is always present, even when there are no records.
