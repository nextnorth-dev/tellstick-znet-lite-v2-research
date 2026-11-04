# Evidence classes and status register

These labels describe the evidence behind each claim. Detailed observations
are in the topic pages and the [physical verification matrix](../HARDWARE_VERIFICATION.md).
Unresolved claims are marked as such.

## Evidence classes

The evidence labels are defined in [HARDWARE_VERIFICATION.md](../HARDWARE_VERIFICATION.md) and used consistently across the repository.

| Label | Meaning |
| --- | --- |
| **RUNTIME VERIFIED** | Supported by captured or recorded runtime evidence such as daemon logs, callbacks, HTTP responses, or state records. Synthetic input is always identified as synthetic. |
| **STATIC CODE VERIFIED** | Established from the extracted official 1.3.2 firmware without the matching live behavior. |
| **VERIFIED IN EMULATION ONLY** | Original code run under an emulator or with substituted hardware. |
| **NOT YET VERIFIED** | The required observation has not been made. |
| **HYPOTHESIS** | A proposed behavior still needing a discriminating test. |
| **PHYSICALLY OBSERVED** | Direct physical observation during testing, without an independent instrumented capture of that result. |
| **SOURCE-RECONSTRUCTED** | Derived from decompilation or recovery rather than original source; a reading aid, never a vendor-source claim. |

A hash match identifies an artifact; it is not a signature, provenance proof, or evidence that the artifact boots.

## Status register

### Hardware, flash, boot, recovery

| Claim | Status | Basis and limit |
| --- | --- | --- |
| AR9331 (reported as AR9330 by the kernel) host, 64 MB DDR2, 16 MB Winbond W25Q128, U-Boot 1.1.4 (AP121), OpenWrt r49388, Linux 4.1.23, five TP-Link MTD partitions | **RUNTIME VERIFIED** | UART capture; see [hardware](hardware.md), [UART and U-Boot](uart-and-uboot.md), [firmware](firmware.md). A sanitized boot log is in [logs](logs/boot-log-sanitized.txt). |
| Interactive U-Boot, including an HTTP recovery command | Prompt **RUNTIME VERIFIED**; command listing from help output | The recovery server's behavior and accepted images were **not tested** by this repository. |
| Full-flash backup compared with official image | Selected regions compared, including an exact SquashFS payload match | **No restore from it has been attempted here.** Backup alone is not a tested recovery route. |
| `sysupgrade` path, `PART_NAME=firmware`, Z-Wave controller outside the AR9331 MTD map | **STATIC CODE VERIFIED** | Controller-NVM survival across a rewrite is an expectation, not a tested guarantee. See [update and recovery](update-and-recovery.md). |

### Stock firmware 1.3.2

| Claim | Status | Basis and limit |
| --- | --- | --- |
| UDP 30303 discovery exposes sensitive data; UDP 42314 has one replaceable listener, no handler-level auth, no command acknowledgement | **STATIC CODE VERIFIED**; listener and registration **RUNTIME VERIFIED** | RF delivery via 42314 **NOT YET VERIFIED**. See [UDP 42314](udp-42314.md). |
| HTTP API uses signed tokens; approval normally relies on Telldus Live OpenID | **STATIC CODE VERIFIED**; anonymous rejection **RUNTIME VERIFIED** | [Authentication](authentication.md), [interfaces](interfaces.md). |
| Vendor update chain verifies size, SHA-1 and a GPG signature before `sysupgrade` | **STATIC CODE VERIFIED** | No vendor signature was verified and no signing key is held. Manual `sysupgrade` does not repeat the GPG check. |
| Embedded RemoteSupport SSH private key present in the firmware | **STATIC CODE VERIFIED** | Key not reproduced; no third-party infrastructure was contacted. See [remote support](remote-support.md). |
| Initial privileged access from factory-locked stock | **NOT ESTABLISHED** | The bounded tests documented in [authentication](authentication.md) do not demonstrate a supported initial privileged-access path and do not prove that no vulnerability exists. |

### Experimental local control

| Claim | Status | Basis and limit |
| --- | --- | --- |
| LocalOwner HTTPS login, local token approval, stock JWT/API reads, dashboard, remember-me/logout | **RUNTIME VERIFIED** | Related extended code on one dedicated modified gateway; byte-identical deployment of the public prototype snapshot is not established. |
| Cloud isolation | Transport disable and one dual-interface IPv4 egress-block cold boot **RUNTIME VERIFIED**; config-upload suppression **STATIC CODE VERIFIED** | No long-duration or universal zero-vendor-traffic proof. Radio response under isolation is **NOT YET VERIFIED**. |
| Z-Wave inclusion, `nodeAdded`, physical-button callback, dashboard ON/OFF of one plug | Inclusion, callback, and recorded state **RUNTIME VERIFIED**; dashboard-command physical load change **PHYSICALLY OBSERVED** | Earlier LocalOwner plug test only. Later Live captures record exclusion/application removal; final controller membership remains unverified. See [later Z-Wave results](later-runtime-results.md#later-z-wave-captures-november-2). |
| RF433 WMR-252 control; generic record CRUD (disposable Arctech record, no RF sent) | Adapter acceptance and generic record operations **RUNTIME VERIFIED**; physical pairing indication and load On/Off **PHYSICALLY OBSERVED** | Later Nexa Live control is recorded separately; general model coverage, receiver-origin feedback and sensor-value RF input remain unverified. Incoming Arctech frames do not identify sender/action. See [RF433 results](later-runtime-results.md#rf433-reception-and-separate-command-paths). |
| Signed MQTT test plugin | **RUNTIME VERIFIED** on one dedicated gateway | A controlled plugin import was followed by an authenticated subscriber receiving a test publication. A later live daemon restart under dual-interface IPv4 isolation also had subscriber-confirmed delivery. Other starts have publish-call logs only; cold-boot subscriber receipt remains unverified. The one-shot plugin and private broker details are not included. |
| Z-Wave `isFailed` semantics | Code path **STATIC CODE VERIFIED**; late-callback overwrite **HYPOTHESIS** | `failed=True` must not be read as unreachable. |
| Home Assistant integration experiment | Token flow, selected reads, and entity creation **RUNTIME VERIFIED**; test-plug On **PHYSICALLY OBSERVED** | Polling stock API state reads is implemented; sustained polling behavior, callback forwarding, connected-device Off delivery, and complete bidirectional synchronization **NOT YET VERIFIED**. |
| `sensorValueUpdated` callback | **RUNTIME VERIFIED** with controlled simulated input in the production daemon | No RF was emitted and no persistent sensor reading was created. Physical sensor-origin input and sensor-to-MQTT delivery are **NOT YET VERIFIED / NOT IMPLEMENTED**. See [hardware verification](../HARDWARE_VERIFICATION.md) and [testing](testing.md). |

### Later runtime and firmware evidence

| Claim | Status | Basis and limit |
| --- | --- | --- |
| Modified R1-derived registration and OpenID callback, November 3 | **RUNTIME VERIFIED** | JSONL registration/callback observations plus browser/session record; not untouched-stock login |
| R1 and rc5 OpenID-handler identity | Artifact bytecode hashes match recorded deployed handler | Does not establish byte identity with original stock or rc5 runtime verification |
| Later rc4 installation | Recorded installed-ROM hash match | Normal boot/install evidence, not failed-boot recovery |
| rc5 build/tests/native acceptance | Recorded tests and native validator passed; image SHA-256 independently matched | No rc5 flash or boot test recorded |
| Controlled RemoteSupport tunnel reproduction | **NOT YET VERIFIED** from located primary evidence | Earlier narrative and static-review status conflict; [evidence gap](remote-support.md#controlled-verification-evidence-gap) retained explicitly |

See [later runtime results](later-runtime-results.md) for exact artifact
identity, dates, test counts, and capture boundaries; [authentication](authentication.md#dropbear-and-owner-ssh)
separates ROM/first-boot SSH policy from runtime and owner public-key login.
