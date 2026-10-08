# Contributing

This is a guide, so the contribution that matters is **being right**.

## Corrections are the most valuable thing you can send

Especially these:

- **A claim in here is wrong or out of date.** Software moves; this document is a
  snapshot. If a flag changed, a project was archived, or a default flipped, that
  is a high-value issue.
- **A procedure in here would destroy data.** Treat it as urgent and say so in the
  subject line.
- **You tested something this guide says it did not test.** The "not verified"
  callouts are honest gaps. Closing one is the single most useful contribution
  available.
- **An option is missing.** A transport or an MCP server that deserves a place in
  the comparison.

## Rules for a change to be merged

1. **No unsourced claims about someone else's software.** Link the upstream
   documentation, the changelog, or the code. "I think it does X" is not a claim,
   it is a rumour, and this document exists to be better than that.
2. **State what you verified and what you did not.** If you ran it, say where and
   what happened. If you read it but did not run it, say that instead. The
   distinction is the product.
3. **Prefer naming the trap over describing the happy path.** Anyone can write the
   happy path; they already found it by following the vendor's quick start.
4. **Do not add a project you maintain without saying so.** Disclose it. A guide
   that recommends something because the author wrote it is an advertisement.

## Style

- Markdown, one sentence per line is not required, long lines are fine.
- Keep the tone dry. No exclamation marks, no "simply", no "just".
- Write for someone who has already read the vendor documentation and is now
  stuck on the thing that documentation skips.
- Run the checks in `README`-adjacent tooling before opening a pull request; CI
  runs a markdown linter and a link checker, and a broken link is a broken claim.

## Reporting instead of fixing

Issues are welcome and often better than a patch. If you found something wrong but
do not want to write the correction, open an issue with: what you did, what you
expected, what happened, and the version you were on. That is enough.

## Code of Conduct

Participation is covered by the [Contributor Covenant](CODE_OF_CONDUCT.md). Reports
go through the private channel described there, not a public issue.

## Security

Do not open a public issue for anything that could lead to someone to lose data or
expose a vault. See [SECURITY.md](SECURITY.md).
