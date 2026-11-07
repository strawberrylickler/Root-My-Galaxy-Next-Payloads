# SM-X820 / X820XXSBDZI2 target profile

Model `SM-X820`, product `gts10pwifi`, Android 16 system build
`X820XXSBDZI2`, kernel `6.1.145-android14-11-abX820XXSBDZI2`.

Build fingerprint:
`samsung/gts10pwifixx/gts10pwifi:16/BP4A.251205.006/X820XXSBDZI2:user/release-keys`.

The profile was generated from the supplied firmware's raw ARM64 Image and
validated raw BTF. Its symbol map and supporting evidence are in the adjacent
`x820_analysis` directory. The P0 table contains 32 candidate rows and was
read back against all 256 source qwords.

The raw BTF reports `sizeof(struct mm_struct) == 0x3c0`. On-device
`/proc/slabinfo` reports the `mm_struct` cache object size as 1024 bytes with
32 objects across 8 pages, so `MM_STRUCT_SZ` is set to the actual 0x400-byte
slab stride used by KernelSnitch.

The prior local app candidate (104,128 bytes, SHA-256
`6e117f5df475fbdedf8b028e608970b0e38c17cc9fce43d541459004249712f7`) used
the S24 `SLIDE_PSELECT_WORD_SHIFT=3`, fake-lock owner link, fixed 50 ms attempt
delay, and waiter CPU 6 pin. A repo-backed tablet run after that candidate was
published still reached a `sched_setattr` OOPS with `refcount_t: addition on
0`; the exact downloaded digest was not captured. The local profile now also
copies the S24 stable-race bank geometry and candidate offsets. The experimental
local build is 104,128 bytes with SHA-256
`94c8058ed09c7d6f2d28d79cc1fb5fb29d4bf2f77a6957c7e9ccc0c090d34d12`; it has
now been run through the app's GitHub source. All 24 attempts reached the
pselect physical-write stage but failed its write window (`ok=0`); no root was
obtained. The tablet stayed up without a fresh kernel OOPS. X820-specific
physical address, firmware offsets, and slab measurements remain unchanged.
Rebuild it from the analysis directory with
`powershell -ExecutionPolicy Bypass -File .\build_x820_app.ps1`.

The app payload and matching KernelSU v3.2.5 module/ksud pair are built. An
X820 compatibility entry is prepared in `support/targets-v3.json` for the
`indeedkerem-coder/Root-My-Galaxy-Payloads` `main` branch.

The compact waiter layout and `mm_struct` slab stride have been matched to X820
BTF and runtime slab data. Earlier candidates reached
`rt_mutex_adjust_prio_chain` OOPSes with refcount warnings. The latest tablet
run used payload SHA-256
`160a5dc358d751ed53c06e813db4ca50d7f521242ea34306041fd8bc5f5c1c9b`. Its
pselect windows and P0 writes passed, and tracefs resolved the kernel base to
`0xffffffc008000000`. The payload nevertheless selected `physical-alias` mode;
its configfs readback at `0xffffff800231acc0` returned no bytes, so it stopped
before root. The S24 profile sets `APP_TRACEFS_KASLR_DIRECT` when tracefs has
resolved the kernel base. X820 now follows that path and uses the canonical
kernel address, avoiding the still-unverified X820 physical load translation.
This follow-up build is not yet hardware-validated. KernelSU module load also
remains unvalidated.

The subsequent 14:03 run used that exact `04d09d263d9e9e37e9820e1e46583b62690a9c5579d3ea4489c7eff8377cf048` artifact but regressed earlier: the app passed `SLIDE_P0_OFFSET=0`, which took a forced-offset branch before the tracefs-direct logic. The log shows `source=forced`, `data_mode=physical-alias`, an unverified fops trigger (`ret=1 errno=0`), and an abort because the stack writer had run. No root or KernelSU was obtained. The tablet was not rerun.

The forced-offset branch now resolves the live tracefs base first and commits it as canonical when available, even when it disagrees with the app's cached P0 offset. The rebuilt artifact is 104,128 bytes, SHA-256 `c16c9122350f3625304c9614754eecc4381eaaff12d7f62df1b990c6d12289dc`. It compiles, but has not been uploaded or tested on the tablet.

The 14:09 run used that exact artifact. It reached the pselect writer window with canonical tracefs addressing, but the CFI readback at `0xffffffc00a31acc0` returned EOF (`ret=0`) instead of the fake fops pointer. The run stopped before root; the device remained responsive. The local capture files are in `x820_analysis/RootMyGalaxy-live-latest-*`.

The current candidate also enables the PI-right-only fake waiter layout for X820's cached-offset path, keeping the oracle tree and fake task unchanged. It builds to 104,128 bytes with SHA-256 `b71c5436108d865f120e74606eb540b9ca3b6e6287bab5ca6f725add654c5a06`. It is not yet hardware-validated.

## 2026-11-07 fresh-page timing candidate

The 16:09 tablet run used the strict pselect check and correctly stopped at
`ret=0`, `triggered=0`; the fake-fops read and physical R/W stages were not
entered. The first fresh-page candidate then exposed a bookkeeping mismatch:
the live tracefs base resolver did not mark the in-process session as fresh,
so all 24 attempts stopped before the FOPS route. The resolver now marks a
live tracefs lookup fresh. The payload still uses up to eight fresh FOPS page
attempts and the existing 70/60/80/40/90/50/30/20 ms delay sequence. This
follow-up candidate has not been run on the tablet.

The 16:25 run passed the fresh-session check and completed all eight page
attempts. Attempts 1-4, 6, and 7 entered the pselect path but returned
`ret=0`; attempts 5 and 8 failed the PI-chain precheck. No FOPS mutation or
root was confirmed. The next candidate adds the S24 pselect synchronization,
blocked-syscall confirmation, and age guard settings to X820. It builds locally
but has not been run on hardware.

The 16:29 run showed the synchronization guards passing on six attempts, but
all eight pselect calls still timed out at 100 ms and returned zero ready file
descriptors. X820's pselect timeout is now aligned with S24 at 500 ms so the
selected descriptors remain under the syscall for the full S24 race window.
This build is an untested candidate.

The 16:39 run reached verified physical R/W and entered the KernelSU helper
handoff, but aborted because the fixed UMH data scratch span contained old
bytes (`work=0 data=1`). The payload now searches the reserved scratch tail of
the reclaimed order-3 page for a zeroed, page-contained span and uses that
address consistently for the UMH pointers. It remains untested on hardware.

The 16:33 run used the 500 ms timeout. The S24 blocked-syscall and guard
checks passed, but the FOPS pselect still returned `ret=0` for all eight
pages. The next local candidate sweeps the eight FOPS delays from 75 to 140 ms
within the 150 ms age guard. It is experimental and untested on hardware.

The subsequent run again missed all eight FOPS attempts, although a previous
100 ms attempt reached CFI and physical R/W. The latest candidate makes 16
fresh-page attempts clustered around 100 ms, retaining the strict positive
pselect result requirement. This increases the kernel-race exposure and is
not validated on hardware.

## 2026-11-07 root-stage run

The app run using `indeedkerem-coder/Root-My-Galaxy-Payloads @ main @ a19e842`
passed the p0, fake-fops, and physical read/write gates, then failed while
publishing the fake work item to `system_unbound_wq`. The three pool-workqueue
counter writes passed; the exact write/readback of `worker_pool.worklist.prev`
failed. No root was obtained. The retained app log for this 15:17 run does not
contain a kernel OOPS; the older retained OOPS belongs to the earlier 14:47
attempt, not this run.

X820's raw BTF confirms `worker_pool.lock=0x0`, `worklist=0x28`, and
`nr_idle=0x3c`. Android 14 / Linux 6.1 documents `worklist` as protected by
`pool->lock`. The current UMH handoff edits the list through physical writes
without acquiring that lock, so the insertion can race worker-pool activity.
This needs a lock-safe handoff before another hardware attempt; a blind retry
or offset change is not justified by the captured evidence.

The local runner now requires four consecutive 1 ms observations of an
unlocked pool with an empty worklist and idle workers. It rechecks the lock and
both list links after writing the scratch records and again after reserving the
workqueue counters; if the pool is visibly active, it rolls back before
publishing the work item. A failed qword write logs its target, requested
value, write/readback results, and observed list pointers. This reduces
contention exposure but does not atomically acquire `pool->lock`, so the race
is not eliminated and this remains an experimental candidate, not a confirmed
root fix.

## 2026-11-07 p0 probe retry candidate

The 18:33 run (`b731ad3`) passed the p0 gate slot once, then the p0 probe slot
missed its pselect write window on its only attempt. The supervisor correctly
refused an unsafe cross-process retry after the p0 oracle had been dirtied. The
captured X820 build used one physical-slot attempt, while the S24 stable target
uses eight independent attempts at 50 ms. X820 now uses that same retry setting
for its physical p0 slots. This may improve the chance of reaching slide
discovery, but it does not address the separate workqueue-lock concern above
and has not been validated on hardware.

The local stable payload artifact is
`artifacts/gts10pwifi-X820XXSBDZI2/cve-2026-43499-app.so` with SHA-256
`2086a55f314a6ec00f5f180a124678e4ed65f95d44f7e71a00c149f15cbc40be`.

The 19:03 run downloaded that exact SHA-256 from the configured GitHub source.
Tracefs resolved the kernel base successfully. The p0 retry setting did not
affect this run because tracefs bypassed the physical p0 scan. The payload then
made 16 fresh-page FOPS attempts with delays clustered from 95 to 105 ms; every
guarded `pselect` returned zero ready descriptors. No CFI verification, kernel
R/W, or root handoff occurred. The next local candidate broadens those 16
delays across 75 to 140 ms while remaining within the existing 150 ms age
guard. It is untested on hardware and does not resolve the separate workqueue
lock concern above. Its local artifact SHA-256 is
`fbfe372e771b8fe9beab203fdf1dfd61cf244082de9156117433dd73642e1a07`.

## 2026-11-07 pselect readiness fix

The 19:20 and 19:24 runs both exhausted all 16 FOPS page attempts. Each
`pselect` returned `ret=0` after the 500 ms timeout, so the strict write-window
check correctly refused to enter CFI verification. The route created a
`timerfd` but never armed it, which meant no selected descriptor could become
readable.

The new candidate watches the read end of a pipe instead. The consumer writes
one byte only after `sched_setattr` succeeds, giving `pselect` a real
readiness event tied to completion of the route trigger. The positive-return
check remains in place, followed by the existing exact fake-fops CFI readback.
The app payload rebuilt successfully as a 104,128-byte artifact with SHA-256
`7a3ba623a52f3b824ade8e5dff73696a7de1a77a8d42d5333c8a4e4a30159d4b`.
This candidate has not yet been run on the tablet.
