# Picasa Photo Viewer filename-order patch

A tiny patch that makes **Picasa Photo Viewer browse images by filename ascending when Windows Explorer's image order is unavailable**.

When an image is opened from a third-party file manager such as Directory Opus, Picasa can fall back to modification-date ordering for adjacent images. This project changes that fallback to filename ordering while retaining the comparisons that prioritize Explorer's displayed order.

![sample](res/example.png)

The repository contains version-specific patch files and documentation. You supply the original Picasa executable and a patching tool.

Confirmed working with Directory Opus on both supported versions.

## Supported executables

| Property | Picasa 3.1 | Picasa 3.9 |
| --- | --- | --- |
| File | `PicasaPhotoViewer.exe` | `PicasaPhotoViewer.exe` |
| Version | **3.1.70.73** | **3.9.141.259** |
| Architecture | **32-bit x86** | **32-bit x86** |
| Size | **4,367,936 bytes** | **4,806,984 bytes** |
| Original CRC32 | `DF5903EF` | `C6AE910F` |

Original SHA-256:

| Version | Original SHA-256 |
| --- | --- |
| 3.1.70.73 | `c56683a64139c73148ca7e30972b567c6212e550a6273b9e48bd80b4bebf98e5` |
| 3.9.141.259 | `d7cdc29430e0887115f09736fccc82fba79c115dc49150695ab7be0ae97ff371` |

Choose the patch for your exact executable version. BPS checks the source checksum and rejects incompatible files. IPS does not check the source, so verify its hash yourself. Other builds have not been investigated.

## Apply the patch

1. Download the portable Windows version of [Floating IPS (Flips)](https://github.com/Sir-Walrus/Flips/releases/tag/v198).
2. Open Flips, click **Apply Patch**, and select the BPS patch for your version from the table below.
3. Choose **All Files (*.*)** in the input dialog and select your original `PicasaPhotoViewer.exe`.
4. Save the output into a copy of your portable Picasa folder, keeping its adjacent `runtime` folder available.
5. Configure your file manager to launch the patched executable.

Close existing Picasa viewer sessions before testing. To test fallback ordering, also close Explorer windows displaying the same image folder: Picasa can discover a matching Explorer view even when another file manager launches it.

Keep the original executable for rollback. Editing the executable invalidates any previously valid embedded signature.

## Patch formats

| Version | BPS (recommended) | IPS |
| --- | --- | --- |
| 3.1.70.73 | [Download — 43 bytes](patches/PicasaPhotoViewer-3.1.70.73-filename-order.bps) | [Download — 25 bytes](patches/PicasaPhotoViewer-3.1.70.73-filename-order.ips) |
| 3.9.141.259 | [Download — 36 bytes](patches/PicasaPhotoViewer-3.9.141.259-filename-order.bps) | [Download — 16 bytes](patches/PicasaPhotoViewer-3.9.141.259-filename-order.ips) |

BPS checks source, output, and patch CRC32 checksums. IPS is an alternative for [Lunar IPS](https://fusoya.eludevisibility.org/lips/); verify the original hash yourself. For each version, both formats contain the same change. Choose one format. No Python or Ghidra is required to apply either patch.

## What changes

The patches bypass timestamp comparisons after equal Explorer-position checks and use the existing filename/path comparison instead. Version 3.1 changes seven bytes across two locations, covering the comparator and its inlined append optimization. Version 3.9 changes three bytes in a shared comparator used by both insertion paths.

| Version | File offset | Original bytes | Patched bytes |
| --- | --- | --- | --- |
| 3.1.70.73 | `0x00050519` | `8D 54 24 40` | `EB 15 90 90` |
| 3.1.70.73 | `0x00050BE7` | `8D 5F 08` | `EB 16 90` |
| 3.9.141.259 | `0x000400BF` | `DD 42 08` | `EB 17 90` |

Filename comparison is bytewise and case-sensitive. Natural numeric ordering and locale-aware sorting are not added; for example, `10.jpg` sorts before `2.jpg`.

Expected patched SHA-256:

| Version | Patched SHA-256 |
| --- | --- |
| 3.1.70.73 | `3f2dc3d0bb35d6d1cf063ac70ad826ca66d9b072d5128811c9135f8d5c21eb79` |
| 3.9.141.259 | `c347b941954587d2dd31d50500e8617a3b2f6cfbb1428488f5a63fc2ae3bb04f` |

See [manifest.json](patches/manifest.json) for patch hashes and compatibility information.

The patches were derived from Ghidra disassembly and decompiled code. Each patch is specific to its executable version identified above.
