# postmach-privacy-hosts

The ad and email-tracker block list for [PostMach](https://machiilabs.com), the free, private Mac + iOS mail client. When the user turns on **Settings → Privacy → Block known ads and trackers**, PostMach skips fetching any remote image whose URL matches this list and collapses the empty slot. No request is ever made for a matched URL.

PostMach ships with a baked-in copy of this list, so the feature works offline and keeps working even if this repo disappears. While the setting is on, the app refreshes from this file at most once every 7 days via jsDelivr:

```
https://cdn.jsdelivr.net/gh/machiilabs/postmach-privacy-hosts@main/privacy-hosts.json
```

The fetch is an anonymous, cookie-less GET with no device or account identifiers.

## File format

```json
{
  "version": 1,
  "ads":      { "hosts": [], "hostPrefixes": [], "pathContains": [] },
  "trackers": { "hosts": [], "hostPrefixes": [], "pathContains": [] }
}
```

The client unions both sections — `ads` vs `trackers` is for human clarity only.

| Field | Matches when… | Example |
|---|---|---|
| `hosts` | the URL host equals the entry or is a subdomain of it | `"taboola.com"` matches `taboola.com` and `trc.taboola.com`, never `nottaboola.com` |
| `hostPrefixes` | any host label sequence starts with the entry | `"sli."` matches `sli.example.com` and `a.sli.example.com` |
| `pathContains` | the lowercased URL path contains the entry as a substring | `"/track/open"` matches `…/track/open.php` |

All entries are lowercased and trimmed by the client.

## Update procedure

1. Edit `privacy-hosts.json`. Bump `version` by 1 (integer, must stay ≥ 1).
2. Check the entry rules below — the client silently rejects the whole file if it is malformed, so a bad commit falls back to the previous list rather than breaking anyone.
3. Commit to `main`. That's the release: no app build, no App Store review.
4. Propagation: jsDelivr caches `@main` for up to ~12 hours, and clients only re-check every 7 days, so expect up to about 8 days before every user has the change. For an urgent fix, both delays still apply — there is no push channel, by design.

**Client validation limits** (a file violating any of these is rejected in full):

- Valid JSON matching the shape above; `version` between 1 and 1,000,000
- At most 2,000 entries per array; each entry at most 253 characters
- File size at most 256 KB

**Entry policy — read before adding a host:**

- Only *dedicated* ad or tracking domains. Never a shared image CDN, an ESP's general image host, or a sender's own domain — a false positive here blanks real newsletter images for every PostMach user.
- `pathContains` entries must be unambiguous across all hosts (they apply to every URL). `"/track/open"` is fine; `"/img"` is not.
- When unsure, leave it out. The user's "Load external images" default-off setting already blocks everything until they opt in; this list only needs to catch the well-known offenders after opt-in.

## Rollback

Revert the commit on `main`. Clients that already downloaded the bad version keep it until their next 7-day refresh picks up the revert; the baked-in seed entries are never removable by this file, only extendable.

## License

[MIT](LICENSE). Contributions welcome — open a PR with the entry, what mail it appeared in (sender domain is enough, no message contents), and why it can't be a false positive.
