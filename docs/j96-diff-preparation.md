# J96 / iPad mini 4 comparison preparation

Prepared 2026-10-25. Documentation-only preparation; no device boot, build, storage operation or hardware qualification performed.

## Scope and preserved baseline

Target: iPad mini 4 Wi-Fi, iPad5,1 / J96 / A1538, Apple A8 / T7000.
Fork: https://github.com/tcballard/hardware
Working branch: j96/diff-prep
Orchard hardware baseline: 9bcaef15ba0250503497d3754fa1254e48f96013
Public iOMARCHY baseline: 9cd661f90edb5fa3d1b45512623522a5d3b14024

Keep the original XPS checkout, uncommitted changes and known-good boot artifacts intact. Use separate checkouts/output directories for experiments.

The public iOMARCHY README still says hardware boot untested. The subsequent 21 October session reported native ARM64 Linux command execution on the physical device. Treat these as different evidence sets: the public repository is available for comparison, but the successful session's exact local changes, binaries, hashes and transcript must be recovered before asserting reproducibility. The earlier session export is not evidence for the later successful run.

## First confirmed finding: identical base revisions

| Component | iOMARCHY config/sources.lock | Orchard lock file |
| --- | --- | --- |
| Linux | 6831bc701a6ce059e71e5aaa9488c9195bea6927 | same, kernel/upstream.lock |
| m1n1 | d5a10ac52a6468484854419a6c5130f1d62073eb | same, boot/m1n1/upstream.lock |

Orchard uses its own mirrors and identifies HoolockLinux as original upstream. A zero base-revision difference does not imply identical built artifacts: patches, configuration, toolchains, device trees, boot arguments and payload composition remain to compare.

Sources:
- https://github.com/tcballard/iOMARCHY/blob/9cd661f90edb5fa3d1b45512623522a5d3b14024/config/sources.lock
- https://github.com/OrchardLinux/hardware/blob/9bcaef15ba0250503497d3754fa1254e48f96013/kernel/upstream.lock
- https://github.com/OrchardLinux/hardware/blob/9bcaef15ba0250503497d3754fa1254e48f96013/boot/m1n1/upstream.lock

## Recover the successful XPS baseline first

Record locally, without publishing identifiers or proprietary inputs:
- Repository HEAD, branch, git status, tracked working diff and untracked source file inventory.
- Actual Linux and loader revisions; applied patches and generated kernel configuration.
- SHA-256 of the executed kernel, DTB, initramfs, loader, Pongo image and combined payload.
- Exact assembly/launch command, payload order, boot arguments and tool versions.
- Timestamped successful transcript demonstrating Linux and /init execution and command output.
- Which transport actually succeeded: Pongo or the separately investigated patched iBoot route.
- Device-tree data supplied by the loader, especially framebuffer address, stride, format and dimensions.
- Local artifact provenance and retained upstream CI download. The public README says the pinned m1n1 CI archive expires 2026-11-29; preserve the existing download.

Use git rev-parse HEAD, git status --short and git diff --binary in the actual source directories. Use sha256sum on the actual successful artifacts. Store raw logs, ADT, ECID, credentials, firmware and calibration privately; publish only reviewed, redacted evidence.

## Diff map

| Area | iOMARCHY baseline | Orchard reference | Question |
| --- | --- | --- | --- |
| Source pins | config/sources.lock | kernel/upstream.lock; boot/m1n1/upstream.lock | Confirm actual successful build uses the matching pins |
| Kernel build | scripts and config in pinned iOMARCHY tree | kernel/build; kernel/configs/j71b/config; kernel/patches/j71b/ | Separate reusable changes from A10/J71b assumptions; retain A8 4 KB pages |
| Device tree | actual J96 DTB and t7000-j96.dts include chain | kernel/devicetree/j71b/ and pinned kernel source | Compare effective DTBs, not just source fragments |
| Display handoff | actual loader-populated J96 framebuffer | boot/m1n1/patches/j71b/; recovery boot arguments | Does the loader enable/fill simple-framebuffer correctly? |
| Payload assembly | scripts/assemble.sh; config/bootargs | boot/assemble.py; image/recovery.py | Order, compression, addresses, entry format and initramfs |
| Recovery | initramfs/; successful USB shell setup | boot/initramfs/j71b/recovery/ | Minimal RAM-only J96 recovery with reliable console first |
| Runtime | current shell bring-up | runtime/j71b/; devices/j71b/ | Identify reusable protocol and J71b-only hardware sequences |
| Host adapter | scripts/boot.sh and successful local variant | OrchardLinux/cli docs/adapter-contract.md | J96 discovery, identity, boot and connection adapter |
| Desktop later | no established desktop baseline | omarchy-a desktop/configuration/login-session | Software-rendered labwc/Quickshell after display/input work |

For textual comparison, use git diff --no-index between the corresponding files; exit 1 means differences, not a tool failure. For configs, use the Linux scripts/diffconfig utility. Decompile copied DTBs with dtc -I dtb -O dts, then compare normalized copies. List initramfs contents and compare /init, executable modes, ELF architecture and USB gadget setup. Hash comparisons establish byte identity only, not hardware functionality.

Do not transplant J71b memory addresses, GPIO mappings, touch firmware, charging policy or NVMe write-enable options to J96.

## Hardware test order and acceptance evidence

1. Reproduce the saved successful RAM boot unchanged. Capture Linux identity and a command round trip.
2. Isolate framebuffer handoff. Record one change, expected observation and rejection criterion; save exact artifact hashes.
3. Establish usable input/USB diagnostics; distinguish physical buttons from touchscreen support.
4. Adapt minimal recovery and the host/device protocol to J96. Keep hardware activation device-specific.
5. Attempt the software-rendered desktop with documented performance and input limitations.
6. Investigate persistent storage separately after validating the device-specific storage stack.

Neither a successful USB upload nor a loader message proves kernel execution. A build passing CI does not qualify hardware. Every experiment should preserve a rollback to the exact known-good payload.

## Boundaries and upstream contribution

- Read applicable repository instructions before code work, including dependencies. Earlier iOMARCHY work identified restrictions on AI changes in upstream m1n1; verify the current instructions before proposing loader edits and do not fork/copy code to evade them.
- J96 exists in the Linux source tree; it is not a packaged Orchard supported device.
- The framebuffer node starts disabled in the shared A8 source and expects loader-populated properties. This is an investigation lead, not a diagnosed failure.
- Orchard's J71b kernel README says combined read/write NVMe mode is not yet hardware-proven.
- Resize APFS currently requires iOS/iPadOS 16+; our reported 15.8.3 device is outside that requirement.
- No storage writes, APFS resizing or installation are part of this first comparison.
- Contribute small independently evidenced changes: J96 metadata/build configuration, RAM recovery/runtime, CLI adapter, then desktop validation. Do not advertise support before physical tests.

## Next-session instruction

Locate the successful 21 October XPS artifacts and logs, read local instructions, and preserve their state. Compare them against this pinned Orchard baseline, starting with patch/configuration/recovery differences because the Linux and m1n1 base commits already match. Produce a concrete diff report separating identical, reusable, J71b-only and unresolved elements. Keep initial hardware experiments RAM-only and change one variable at a time. Do not treat stale public README status or historical build manifests as the successful-run manifest.
