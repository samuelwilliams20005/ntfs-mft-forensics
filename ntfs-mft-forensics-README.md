# NTFS Master File Table (MFT) Forensic Analysis

Byte-level decoding of the NTFS boot sector and Master File Table, comparing findings against The Sleuth Kit's automated output to validate manual analysis.

## Objective
- Locate and decode the partition table and NTFS boot sector by hand
- Understand and manually parse MFT entry structure
- Validate manual decoding against The Sleuth Kit's `istat` output
- Compare NTFS's data structures against FAT

## Tools
`dd` · a hex editor · `The Sleuth Kit (fls, istat)`

## Process
1. **Partition table decoding** — used `dd` to extract the boot sector, then manually decoded the partition table entries in a hex editor (boot flag, CHS addressing, partition type, starting LBA, length).
2. **Locating the MFT** — extracted bytes 48–55 of the NTFS boot sector to find the MFT's starting cluster.
3. **MFT entry structure** — manually decoded a 1024-byte MFT entry field by field (signature, fixup array, attribute offsets, flags, entry size) and cross-referenced it against the standard NTFS MFT layout.
4. **Validation with TSK** — ran `fls` to list all files/directories on the volume, then `istat` on two file entries to pull their full metadata (timestamps, attributes, resident vs non-resident data) and confirm it matched the manual hex analysis.

## Key finding
Manually decoded field values (entry size, attribute offsets, flags) matched exactly with TSK's `istat` output, confirming correct understanding of the MFT entry structure. Also identified that larger files are stored as **non-resident** attributes (data in external clusters) vs small files stored **resident** (data inline in the MFT entry) — a distinction that matters for recovery strategy.

## NTFS vs FAT
| | NTFS | FAT |
|---|---|---|
| Metadata | Comprehensive, MFT-based | Basic, directory entries |
| Journaling | Yes (`$LogFile`) | No |
| Free space tracking | `$Bitmap` | FAT table itself |
