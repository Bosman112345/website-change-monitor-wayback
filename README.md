# Website Change Monitor: Price and Text Changes via Wayback

**Run it on Apify:** https://apify.com/rickivory/wayback-what-changed

**Track competitor price changes without the alert noise.** Give it a URL. It reads every archived Wayback Machine snapshot in the window you choose, compares the cleaned visible text, and returns only the changes that are real: lines added, lines removed, and prices that moved, with links to both snapshots.

It does not flag a page as "changed" because its markup, tracking code or a rotating widget changed.

## Who it's for
- **Competitive pricing research.** When did a competitor change their plans, and from what to what?
- **Terms and policy tracking.** What changed in a terms of service, refund or privacy policy?
- **AI agents** answering "when did X change on this page?" in one call with structured output.

## Real example: stripe.com/pricing (run on Nov 9, 2026)
Default input, last 3 changes, 25 snapshots scanned. The Actor ignored 4 lines from a recommendations widget that blinked in and out between snapshots, and returned 2 real changes:

1. **Oct 30, 2026.** Stripe restructured its payouts pricing: a new "Dispute appealed fee: $15.00 for each dispute you appeal", new per-rail payout rates (ACH next day $0.50, ACH same day $1.00, real-time $2.50, Fedwire $25.00), and coverage going from "50+ countries" to "160+ countries".
2. **Nov 5 to Nov 6, 2026.** A wording fix: "Smart Dispute fee" became "Smart Disputes fee".

## How it works
1. Looks up distinct archived captures of each URL with the Internet Archive CDX API.
2. Fetches the archived HTML and strips scripts, styles and, by default, navigation and footers.
3. Drops flickering content: lines that appear and disappear 3 or more times across the scanned snapshots (rotating widgets, A/B tests, personalised banners) are ignored.
4. Compares the visible text of every snapshot in the window. Identical snapshots are skipped.
5. Reports each real change with dates and links to both snapshots.
6. Pairs price changes only when the surrounding words match ("Pro plan $29/mo" to "Pro plan $35/mo"), so unrelated numbers are never reported as a price move.

## Input example
```json
{
  "urls": ["https://stripe.com/pricing"],
  "mode": "lastChanges",
  "lastChanges": 3,
  "maxSnapshotsToScan": 25
}
```

## Output example (abridged)
```json
{
  "type": "diff",
  "url": "https://stripe.com/pricing",
  "from": { "date": "2026-11-05T13:09:06Z" },
  "to":   { "date": "2026-11-06T13:23:09Z" },
  "summary": "1 line(s) added, 1 removed",
  "added":   ["There's no Smart Disputes fee for lost disputes. The dispute received fee still applies."],
  "removed": ["There's no Smart Dispute fee for lost disputes. The dispute received fee still applies."],
  "flickeringLinesIgnored": 4
}
```

## Limits
- Coverage depends on the Internet Archive. If it did not capture a page, there is nothing to compare.
- The Archive sometimes captures another country's version of a page; currency switches are flagged, not reported as price changes.
- Polite by design: one request at a time, with delays and backoff on rate limits. Large runs take time.

## Pricing (pay per event, on Apify)
Per snapshot scanned, per real change found, and optionally per snapshot row. You pay for changes, not for noise. A max cost per run stops the scan cleanly.

Archived content is provided by the Internet Archive (https://archive.org). This project is not affiliated with or endorsed by the Internet Archive.
