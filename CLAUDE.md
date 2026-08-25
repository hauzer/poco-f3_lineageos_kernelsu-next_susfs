# POCO F3 (alioth) — LineageOS + KernelSU-Next + SUSFS

Guide repo, **not** a source tree. There is no build system here — the actual ROM build happens in a
LineageOS source tree on a separate Ubuntu machine. Reference device: POCO F3 `alioth`
(M2012K11AG), LineageOS 22.2 / Android 15, kernel 4.19 non-GKI.

## What's Here

| File | What it is |
|---|---|
| `README.md` | The guide, and the actual deliverable. First-person, emoji section headers, GitHub `> [!TIP]`/`> [!NOTE]` callouts. Keep that voice when editing. |
| `kernelsu-next_susfs.patch` | Combined KernelSU-Next + SUSFS patch for the non-GKI 4.19 `alioth` kernel. |

## The Patch

Applied with `patch -p1` from the kernel root. It adds only `CONFIG_KSU=y` and `CONFIG_KSU_SUSFS=y`
to `arch/arm64/configs/vendor/xiaomi/alioth.config` — every SUSFS sub-option is `default y` in the
Kconfig the patch ships, so the full feature set is enabled without being listed in the device config.
Don't "fix" the device config by adding the sub-options; they're already on.

It does **not** vendor `include/linux/susfs.h`. That comes from the upstream susfs4ksu kernel patch,
applied separately per the README, so the kernel's SUSFS version is whatever that branch provides.

## Version Semantics

Kernel SUSFS version and susfs4ksu **module** version are independent, and an apparent mismatch
usually isn't one. Module tags read `v1.5.2+_R28`: `1.5.2+` is the **minimum** kernel SUSFS version
the module supports, `R28` is the module's own revision counter. A v1.5.5 kernel paired with an R28
module is the supported configuration. Read the module's stated requirement before calling a pairing
broken.

## Talking to the Device (adb)

adb is not on PATH — it lives at `$LOCALAPPDATA/Android/Sdk/platform-tools/adb.exe`, and Android
paths need `MSYS_NO_PATHCONV=1` (see Platform Quirks). `su` is only available to apps granted root in
the KernelSU-Next Manager, so `su: inaccessible or not found` means "not granted", not "not installed".

Push root scripts and run them with `sh` rather than inlining — `adb shell su -c "..."` mangles nested
quotes. Delete pulled files holding key material once you're done with them.

## Naming Things

**Spell it out. Never invent shorthand.** No ad-hoc acronyms, no invented labels, no cute names for
things that already have names. Repeating six extra words saves the reader a lookup; it costs nothing.

Label alternatives by what they do, not by letter or number — "the expired software keybox", not
"Option 2". Established vocabulary is not shorthand: KernelSU-Next, SUSFS, GKI, GApps, GSF, TEE,
keybox, adb and anything else already defined in the README stay as they are. The rule targets terms
invented mid-conversation that exist nowhere else.

## Git

**Never commit or push without an explicit request.**

Commit messages are plain descriptive one-liners — no ticket prefixes, no `Co-Authored-By` trailer,
no multi-line bodies. Match what's already in the log (`Update README.md`).

## CLAUDE.md Protocol

**Size limit:** ~200 lines — a review trigger, not a cliff. At ~200 do a pass; past 250 fix it first.

**Keep-test:** content belongs here only if *not* knowing it up front would cause a **wrong action**.
Everything explanatory or reference-shaped belongs in the README, which is what this repo exists to
publish.

**Why:** this file is auto-loaded *as instructions* in every session under its directory. The cost of
bloat is attention, not tokens — explanatory prose dilutes the hard rules sitting next to it.

**Committed:** `CLAUDE.md` is tracked and committed with the repo like any other file.

## Platform Quirks (Windows)

**No null redirections.** Never `/dev/null`, `> nul`, or `2> nul` in bash — on Windows that creates a
file literally named `nul` in the working directory. Omit the redirection entirely, or use `2>&1`.
`| Out-Null` is PowerShell-only.

**Path separators:** forward slashes in bash, backslashes in PowerShell. Most tools accept either.

**`MSYS_NO_PATHCONV=1`:** see the adb section above. The same trap applies to any `/`-prefixed
argument that isn't a real local filesystem path. The resulting error blames the parameter, not the
shell, so it reads like the value is invalid when it isn't.

**Temp files:** use the session scratchpad directory. Never create scratch files inside the repo — it
has no `.gitignore`, so anything dropped here shows up in `git status` immediately.

**PowerShell called from bash:** wrap the whole `-Command` argument in single quotes so bash doesn't
expand `$env:` variables before PowerShell sees them.
