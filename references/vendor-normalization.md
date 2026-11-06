# Vendor normalisation, worked examples

Grouping is where this skill is right or wrong. Everything downstream, the cycle, the price change, the total, depends on deciding which documents belong to the same payee.

The procedure is in `SKILL.md`, step 3. This file is the set of cases it has to get right, and the cases where it must refuse to group.

## Names that must group

| As printed on the documents | Normalised | Why they group |
|---|---|---|
| `Adobe Inc.`, `ADOBE SYSTEMS SOFTWARE`, `Adobe *Creative Cloud` | `adobe`, `adobe systems software`, `adobe creative cloud` | One normalised name is contained in the others |
| `AMZN Mktp CA*2H4XY1`, `Amazon.ca`, `AMAZON MKTP CA` | `amzn mktp ca`, `amazon ca`, `amazon mktp ca` | Route token and reference stripped, remainder agrees |
| `SQ *THE COFFEE BAR`, `The Coffee Bar Ltd` | `sq coffee bar`, `coffee bar` | `sq` is a Square payment route, not a vendor |
| `GOOGLE*GSUITE_argoro`, `Google Workspace` | `google gsuite argoro`, `google workspace` | Same payee, same monthly cycle, amounts agree |
| `Hydro One`, `HYDRO ONE NETWORKS INC` | `hydro one`, `hydro one networks` | Containment after removing `inc` |
| `PAYPAL *SPOTIFY`, `Spotify AB` | `paypal spotify`, `spotify ab` | PayPal is the route; group on what follows |

## Names that must not group

| As printed | Why grouping them would be wrong |
|---|---|
| `Apple.com/Bill` monthly at 2.99 and `Apple Store` once at 1,899.00 | Same payee, two unrelated spending patterns. The subscription is recurring; the hardware purchase is not. Split them, and say why |
| `Microsoft 365` at 11.29 monthly and `Microsoft Azure` at a varying amount | Different products on different cycles. The normalised names share `microsoft` only, which is below the threshold once `microsoft` is the single common token |
| `Shell` fuel purchases and `Shell Energy` gas bills | One is irregular retail, the other is a monthly utility. Same word, different vendor |
| `Bell Canada` and `Bell Media` | Similar names, no amount or cycle agreement |

The deciding question when similarity is borderline: do the amounts and the cycle agree? Two documents that share a word but share nothing else are two vendors.

## Why bare numbers are removed

A card statement line carries a store number, a terminal id, or the last four digits of the card. Those change between occurrences of the same charge, so leaving them in would split one subscription into twelve one-off purchases.

`STARBUCKS #4471` and `STARBUCKS #0928` are the same vendor. `Invoice 2026-00831` and `Invoice 2026-00947` are the same vendor too, which is why the word `invoice` is in the noise list alongside the numbers.

## Why single letters are removed

Separator artefacts. `AMZN*MKTP` becomes `amzn mktp`, but `A&W` becomes `a w`, and a lone `a` or `w` matching some other vendor's stray letter would create false overlap.

## The one case that needs a human

A normalised name that appears with two distinct, stable amounts on two distinct cycles is almost always two products from one company. Report both patterns under the one payee, keep their figures separate, and let the user decide whether they want them merged. Do not merge silently, and do not split silently.
