# CHANGES.md — makerdiary-bootloader branch

What this branch adds on top of upstream `safijari/openpuck` main (16 files, +938/-10).
All changes are compile-time isolated behind `-DOPK_BOARD_MDK_USB_DONGLE=1`; ordinary
OpenPuck builds are unaffected.

## Makerdiary/GeeekPi nRF52840 MDK USB Dongle support

### bootloader/makerdiary/ (new tree)
- `README.md` — board docs: build recipe, first-install (factory S132 5.1.0 -> S140 6.1.1,
  app linked at 0x26000), bootloader build policy.
- `build_mdk_bootloader.sh` — clones Adafruit_nRF52_Bootloader pinned at commit
  `c87ea51b86b96f9b19458abfc6b37d4cb52e160b`, applies exactly two patches, builds the
  UF2 bootloader as `0.7.1-openpuck`.
- `0001-Makerdiary-reset-UICR.patch` — conservatively configure P0.18 as nRESET.
- `0002-OpenPuck-app-handoff.patch` — stop LFCLK before application handoff so OpenPuck
  can establish its own application clock policy.
- `flash_mdk_bootloader.sh` — flashing helper.

### tools/ (new)
- `make_mdk_first_install.py` — packages SoftDevice + application into a first-install
  UF2; refuses SoftDevice/app overlap and any block at or above the Makerdiary bootloader
  base 0xF4000; leaves InternalFS range 0xED000..0xF3FFF unpopulated.
- `verify_mdk_bootloader_hex.py` — validates bootloader hex: accepts programmed bytes only
  in 0xF4000..<0xFE000 plus the standard Adafruit UICR words at 0x10001014/0x10001018;
  rejects application, SoftDevice, settings-page, and unexpected UICR writes.

### Firmware (all under OPK_BOARD_MDK_USB_DONGLE)
- `OpenPuck/OpenPuck.ino` — mode-switch reboot drives P0.16 low to hard-reset the dongle
  via its wired P0.18 nRESET (the nRF52840 WDT survives NVIC_SystemReset); USB interface
  ordering tweak so puck slot HIDs stay on stable interface numbers when the MDK debug
  boot keeps CDC on IF0/IF1.
- `OpenPuck/board_config.h` — MDK board keeps Adafruit-format DFU/settings page
  (`OPK_HAS_ADAFRUIT_DFU 1`).
- `OpenPuck/fault_diag.cpp` — MDK clock diagnostic: refresh LFCLK/HFCLK source fields
  without mutating clock state while the board's clock policy settles.
- `OpenPuck/status_led.cpp` — status/wake LED handling for the MDK board.

### Build & CI
- `Makefile` — new targets: `build-mdk`, `build-mdk-bootloader`, `package-mdk-first-install`,
  `package-mdk-release`.
- `gen_uf2.sh` — adjusted for the MDK output paths.
- `.github/workflows/build.yml`, `release.yml` — MDK build/release jobs; CI installs the
  bootloader Python deps (adafruit-nrfutil, intelhex) explicitly.

### Docs
- `docs/BUILD_AND_DEPLOY.md` — new §4b (MDK dongle: build-mdk, first-install S140 flow,
  routine UF2 updates, Windows PATH notes for Arduino CLI).
