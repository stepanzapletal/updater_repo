# Static OTA manifests for HyperOS 2 Updater

This repository serves per-device JSON manifests from GitHub Raw. The patched
Updater reads `devices/<ro.product.mod_device>.json`; for this device that is
`devices/gale_global.json`. No backend service is needed.

## First run

`devices/gale_global.json` currently has `"latest": null`, so it reports the
installed ROM as current and offers no download. Replace `null` only after an
OTA ZIP is ready. Copy the structure in `devices/manifest.example.json` into
the `latest` field and replace every example value.

## Publishing an update

1. Upload the complete flashable OTA ZIP to an HTTPS host. GitHub Releases has
   a 2 GB per-file limit; use SourceForge, Hugging Face, or another host for
   larger ROMs.
2. Calculate the ZIP's lowercase MD5 and exact displayed size. `filesize` must
   end with `G`, `M`, or `K` (for example `4.8G`), and `filename` must be the
   actual ZIP filename.
3. Edit `devices/gale_global.json`. Set `latest.version`, `display_version`,
   `build_timestamp` (Unix seconds), `download_url`, `filename`, `filesize`,
   and `md5`. The updater downloads the ZIP, then uses its stock A/B install
   flow.
4. Add a changelog section and optional HTTPS image URLs. The schema example
   shows light/dark logos, section icons, portrait/landscape banners, and inline
   images. `branding.os_name` and each release's `display_version` control the
   header text.
5. Move the previous `latest` version and its changelog into `releases` under
   the exact version string. Commit and push to `main`; changes are available
   from Raw GitHub after propagation.

The adapter also accepts `mirrors` as extra direct HTTPS download URLs. Every
manifest must be valid JSON and reachable at the exact device filename.

## Files

- `devices/gale_global.json`: active device manifest, initially no update.
- `devices/manifest.example.json`: full example manifest schema.
- `assets/`: put branding and changelog artwork here; use HTTPS Raw URLs in
  the manifests.

## Dummy download test currently published in the manifest

The active `gale_global.json` temporarily advertises `DUMMY-TEST-1` and points
at `assets/dummy_update_test.zip`. This tiny ZIP contains only a warning text
file; it has no `payload.bin`, recovery script, or ROM contents. It is for
checking manifest retrieval, UI rendering, and the download/checksum path. The
native install step should fail because there is no update payload. Replace
`latest` with a real, correctly signed ROM package before using this updater
for actual updates.
