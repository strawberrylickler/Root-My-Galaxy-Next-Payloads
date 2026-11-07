# Galaxy Tab S10+ Wi-Fi (SM-X820) — experimental

Exact target: product `gts10pwifi`, firmware `X820XXSBDZI2`, Android 16, kernel
`6.1.145-android14-11-abX820XXSBDZI2`. Do not use these files on another
model, firmware, or kernel.

The current app payload (`cve-2026-43499-app.so`, SHA-256
`7a3ba623a52f3b824ade8e5dff73696a7de1a77a8d42d5333c8a4e4a30159d4b`) has
not been run on the tablet. Earlier candidates caused kernel OOPSes or failed
to obtain root. Source review also identified a remaining workqueue lock race;
the current guard reduces exposure but does not eliminate it. Treat the payload
as experimental. A kernel OOPS, bootloop, or data loss is possible. Back up
important data and know how to restore stock firmware before testing.

The matching KernelSU daemon is `kernelsu/ksud-gts10pwifi-X820XXSBDZI2-kdp`
(SHA-256 `2b1a3f1ae33d3856fa5dbd9639f841a82410c251e9e3901ce7a647e42bc6105d`).
The KernelSU load path for this candidate has not been validated on the tablet.
The bootstrap helper is `cve-2026-43499-root` (SHA-256
`e719691efb77e22681a3ae801f09d6150caa7a4a8b5157b2d58091d399ab9bec`).

The manifest entry is in `support/targets-v3.json`. The app fetches the `.so`
and daemon from this repo. The build profile and evidence are in
`src/targets/gts10pwifi-X820XXSBDZI2/README.md`.
