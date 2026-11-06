# What to do about a charge

A report that only says what you are paying stops one step short. This file is the list the skill is allowed to suggest from, and the rules for suggesting.

## The rules

**Suggest only from the table below.** Never invent an alternative from memory, and never name a product that is not listed here.

**Never state a price for an alternative.** Not a number, not "about ten dollars", not "cheaper". The skill's figures come from documents, and there is no document for a product the user has never bought. Say what the alternative is and let them look up what it costs today.

**Say what the trade is.** Every alternative costs something: effort, a migration, a feature that is missing, a backup you now own. A suggestion without its cost is an advert.

**Keep suggestions out of the figures.** They go in their own section at the end of the report, headed "Worth a look", after everything measured. Never inside the recurring charges table, and never phrased as something the skill found in the documents.

**Suggest nothing for a charge you cannot categorise.** An unmatched vendor gets no suggestion at all.

**Say it once.** One suggestion per vendor, the best one. A list of five options is a list the user has to research.

## The one worth leading with

**A cloud storage subscription, on a machine whose entire purpose is storage.** This is the only suggestion the skill should put first when it applies, because the alternative is already running in the user's house and already being paid for.

## The list

| What the charge is for | Self-hosted on the NAS | Hosted alternative | The trade |
|---|---|---|---|
| Cloud file storage and sync | Nextcloud, Seafile, Syncthing | Proton Drive, pCloud | You become responsible for backups and for off-site copies. A NAS in your house is one fire away from being no backup at all |
| Photo storage and sharing | Immich, PhotoPrism | Ente | Face and object search is good but not identical. Phone apps are less polished |
| Password manager | Vaultwarden | Bitwarden hosted, KeePassXC with your own sync | If your NAS is down and you are away from home, you are locked out unless clients cache |
| Document archive and OCR | Paperless-ngx | none listed | Needs real CPU for OCR, and the initial import is a weekend |
| Backup to the cloud | Duplicati, Restic, Kopia to cheap object storage | Backblaze B2, Borgbase | Object storage still costs money, usually much less. You own the restore test |
| Media streaming | Jellyfin | none listed | No licensed catalogue. This replaces your own library, not a streaming service |
| Office suite and documents | OnlyOffice, Collabora, LibreOffice | none listed | Real-time editing with outside collaborators is weaker |
| Email hosting | none recommended | Fastmail, Proton Mail, Migadu | Self-hosting email is the one thing on this list not worth doing. Deliverability is the reason |
| Website or blog hosting | Ghost, WordPress on the NAS | a static host | A machine at home on a domestic connection is not a web host |
| VPN subscription | WireGuard or Tailscale to reach your own network | a commercial VPN | These solve different problems. Reaching your own NAS is not the same as hiding your traffic |
| Note taking | Joplin with its own sync, Obsidian with Syncthing | none listed | Sharing and publishing notes gets harder |
| Creative software on subscription | none listed | Affinity, DaVinci Resolve, Krita, GIMP, Inkscape | File compatibility with collaborators is the usual blocker |
| Code hosting | Gitea, Forgejo | Codeberg | Continuous integration is on you |
| Accounting and bookkeeping | none listed | GnuCash, Wave | Moving mid-year means carrying over opening balances |
| Home automation | Home Assistant | none listed | A vendor's cloud features may stop working |
| Monitoring and dashboards | Grafana, Uptime Kuma | none listed | Alerting from a machine that is itself down needs a second machine |

## Matching a vendor to a row

Match on what the vendor sells, not on its name, and only when it is unambiguous. A company that sells several things gets matched by the amount and cycle of the charge, not by the brand.

When no row clearly fits, say nothing. "Worth a look: nothing obvious" is a better line than a bad suggestion.

## What a suggestion looks like

> **Worth a look**
>
> Dropbox, yearly, \$143.88 last time. You are paying for cloud storage on a machine built for storage. Nextcloud or Seafile run on this NAS and do the same job. The trade: backups become yours, and a NAS at home is not an off-site copy.

Three sentences. What it is, what the alternative is, what it costs you. No price for the alternative, no pitch.

## The cancellation case

For a dormant charge, say what the documents show and nothing more:

> Audible last billed 30 January 2026, nine cycles ago. If this was cancelled, nothing to do. If it is still billing, the invoices have stopped arriving here, and the place to check is the account itself.

Never say "cancel this". The skill cannot see a bank account and does not know whether the money is still moving.
