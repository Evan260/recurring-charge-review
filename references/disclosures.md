# Disclosures

## Third-party components

None. This Skill is Markdown instructions. It contains no scripts, binaries, libraries, SDKs, APIs, models or datasets, and it makes no network calls. There is nothing here that carries a third-party licence, and no external service or data flow to declare.

The products named in `alternatives.md` are mentioned as alternatives a user might look into. None of them is used, bundled, called or depended on by this Skill.

## Principal functionality developed for this Hackathon

All of it. `SKILL.md` and every file under `references/` were written for this Hackathon, starting 30 October 2026, and have not been submitted anywhere before.

What existed beforehand is experience rather than material: the author builds commercial accounting software, and the judgement encoded in these rules comes from that work. The rules themselves, which date to take from an invoice, how to normalise a vendor name, when two documents are the same charge, were written here from scratch as instructions for an agent. No earlier Skill, document or submission was reused.

The behaviour described has been exercised in two unattended runs by an AI assistant following `SKILL.md` against a library of 177 generated documents. Those runs surfaced 26 ambiguities in the instructions, each of which was closed. The Skill has never run on UGREEN hardware.

## AI tools used, and their principal uses

**Claude (Anthropic), via Claude Code.** Used for four things:

1. **Drafting.** `SKILL.md`, `README.md` and the reference files were drafted with Claude, from the author's own rules and decisions, and edited by the author.
2. **Testing.** Claude acted as the assistant executing `SKILL.md` against the test library, twice, without sight of the answer key. Those runs are where the 26 ambiguities came from.
3. **Building the test library.** The generated invoices and receipts used for testing were produced with Claude's help. They are not part of this package.
4. **Review.** Claude audited the test harness against `SKILL.md` and found arithmetic and specification errors, which were fixed.

The Skill itself does not call Claude or any other model. At run time it uses whatever model the NAS provides.
