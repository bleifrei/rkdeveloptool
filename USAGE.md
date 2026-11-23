# `el` — Erase LBA range

```
rkdeveloptool el <BeginSec> <CountSec|-1>
```

Erases `CountSec` 512-byte sectors starting at sector `BeginSec` on the
currently selected storage (see `cs`). The device must be in **Loader** or
**Maskrom** mode; in Maskrom mode download a loader first (`db <loader>`),
otherwise the erase command is not understood by the device.

The erase is sent to the device in chunks of 32768 sectors (16 MiB); if any
chunk fails, the command stops and reports failure — sectors erased before
the failing chunk stay erased.

## Arguments

| Argument   | Meaning                                                        |
|------------|----------------------------------------------------------------|
| `BeginSec` | First sector to erase (0-based).                               |
| `CountSec` | Number of sectors to erase, or `-1` / `0xFFFFFFFF` = to the end of the flash. |

Numbers are parsed like C literals: decimal (`4096`), hex (`0x1000`) or
octal with a leading `0` (`010` = 8). Note the octal rule — `0100` is 64,
not 100.

## Examples

```sh
# Erase sectors 0x2000..0x3FFF (0x2000 sectors = 4 MiB)
rkdeveloptool el 0x2000 0x2000

# Erase from sector 0x8000 to the end of the flash
rkdeveloptool el 0x8000 -1
rkdeveloptool el 0x8000 0xFFFFFFFF   # same thing

# Erase the whole flash (similar to `ef`, but via LBA erase)
rkdeveloptool el 0 -1
```

## Limits

- Sector addresses are 32-bit, so the addressable range is 2³² × 512 B = 2 TiB.
- Sector size is always 512 bytes, independent of the underlying media's
  page/block size.
- An explicit `CountSec` that runs past the end of the flash is not checked
  by the tool; the range is sent as-is, and sectors that fit may be erased
  before the device reports failure. Check the size with `rfi` first.
- To find the flash size in sectors, run `rkdeveloptool rfi` and read the
  `Flash Size: ... Sectors` line.
