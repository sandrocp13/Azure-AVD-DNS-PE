# 06 - Screenshot redaction checklist

Use this checklist before publishing to GitHub.

## Must redact

- Subscription IDs
- Tenant IDs
- Public IP addresses
- Server names / hostnames
- User principal names and email addresses
- Tenant-specific DNS names not intended for public sharing
- Any secrets, keys, tokens, or connection strings

## Recommended redaction method

1. Capture screenshot at full quality.
2. Duplicate original into a `raw` backup folder (never publish raw).
3. Redact with opaque overlays (not blur) so data cannot be reconstructed.
4. Re-open redacted image at 200% zoom to verify no leakage.
5. Use consistent labels (`DC01`, `DC02`, `HUB-VNET`, etc.).

## File naming convention

- `01-hub-vnet-overview.png`
- `02-storage-networking-private-disabled.png`
- `04-dns-forwarder-privatelink-file.png`
- `05-nslookup-storage-private-ip.png`

## Final QA pass (required)

- [ ] Every screenshot referenced in docs exists in `assets/screenshots/`.
- [ ] No sensitive values visible at normal and zoomed views.
- [ ] Step numbers in docs match screenshot IDs.
- [ ] Captions describe purpose, not environment secrets.
