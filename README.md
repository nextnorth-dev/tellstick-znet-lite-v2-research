# TellStick ZNet Lite V2 research

Independent hardware and firmware research on the Telldus TellStick ZNet Lite
V2, centered on official firmware 1.3.2. The work combines static analysis of
the vendor image with bounded tests on an owned device. It is not affiliated
with Telldus.

> **Research only.** This repository documents observations and experimental
> prototypes; it is not a firmware distribution, supported installation, or
> released product. No firmware image, flash dump, credential, private key, or
> device-specific configuration is distributed here.

## What the evidence supports

- The examined unit uses an AR9331-class MIPS host, U-Boot, OpenWrt, a SquashFS
  root filesystem with a JFFS2 overlay, and separate RF433 and Z-Wave interfaces.
  See [hardware](docs/hardware.md), [firmware](docs/firmware.md), and
  [UART and U-Boot](docs/uart-and-uboot.md).
- Stock firmware 1.3.2 exposes local discovery and event interfaces and a
  token-protected HTTP API. The exact test scope and limitations are documented
  in [interfaces](docs/interfaces.md), [authentication](docs/authentication.md),
  and [UDP 42314](docs/udp-42314.md).
- Selected local-control paths were exercised on one modified test gateway.
  Results do not establish compatibility with untouched devices or other
  firmware versions.
- Physical Z-Wave inclusion produced a captured `nodeAdded` callback and
  persistent stock-API listing. An owner-initiated button press produced a
  captured state update; On/Off switching of the connected test load is an
  **OWNER-REPORTED PHYSICAL OBSERVATION**. The WMR-252-specific RF433 load
  switching is also **OWNER-REPORTED PHYSICAL OBSERVATION**. These bounded
  results do not establish general radio support.
- The stock plugin loader's package-signature and startup path was exercised
  with a locally signed test plugin. A controlled MQTT publication reached an
  authenticated subscriber. This did not test arbitrary authenticated uploads
  or radio-event forwarding.
- A separate Home Assistant experiment completed owner-approved token
  issuance, selected stock-API reads, and entity creation. Polling stock API
  state reads is implemented. The owner reported that an On command reached
  one connected test plug (**OWNER-REPORTED PHYSICAL OBSERVATION**); sustained
  polling behavior, callback forwarding, complete bidirectional
  synchronization, and a connected-device Off command have not been fully
  verified.
- A production-daemon sensor callback was tested with simulated input. No RF
  signal was emitted; physical sensor-origin events and sensor-to-MQTT delivery
  remain unverified.
- One cold boot with scoped IPv4 egress blocks passed selected local checks.
  This does not prove universal or long-duration absence of outbound traffic.

The [evidence register](docs/evidence-status.md) and
[physical verification matrix](HARDWARE_VERIFICATION.md) distinguish static
analysis, emulation, physical observations, owner reports, and unverified
behavior.

## Technical map

- [Hardware and firmware](docs/hardware.md), [UART/U-Boot](docs/uart-and-uboot.md),
  [Wi-Fi research](docs/undocumented-wifi.md), and
  [firmware update/recovery](docs/update-and-recovery.md)
- [Local interfaces](docs/interfaces.md), [authentication](docs/authentication.md),
  and [UDP 42314](docs/udp-42314.md)
- [RF433](docs/rf433.md) and [Z-Wave code paths](docs/zwave-local.md)
- [Local access and network isolation](docs/no-cloud.md)
- [Security review](SECURITY_STATIC_REVIEW.md) and [test methods/results](docs/testing.md)

## Research boundaries

The official 1.3.2 image is an analysis reference, not a distributed artifact.
Firmware update, bootloader, and recovery findings are described in
[firmware](docs/firmware.md) and [update and recovery](docs/update-and-recovery.md).
No recovery procedure is
claimed as tested unless the evidence says so.

The LocalOwner material in this repository is experimental research, not a
released product. OpenTZnet Lite and the future LocalOwner product integration
are separate development tracks and are not release deliverables of this
Research repository.

## Documentation

- [Evidence status](docs/evidence-status.md) — claim labels and evidence limits
- [Hardware verification](HARDWARE_VERIFICATION.md) — physical test matrix
- [Interfaces](docs/interfaces.md) and [authentication](docs/authentication.md)
- [RF433](docs/rf433.md) and [Z-Wave local paths](docs/zwave-local.md)
- [Local access and network isolation](docs/no-cloud.md)
- [Testing](docs/testing.md) and [references](docs/references.md)
- [Security policy](SECURITY.md)

## Safety and scope

Firmware modification and direct flash access can damage a device or erase its
configuration. Findings apply only to the examined hardware and firmware
unless stated otherwise. Tests were limited to owned or explicitly authorized
equipment. Proprietary firmware, extracted root filesystems, credentials,
private keys, raw private captures, and device-specific configuration are not
included.

This repository is for research and interoperability. It does not provide
security patches, installation support, or a guarantee of compatibility or
recovery. Report sensitive issues privately as described in
[SECURITY.md](SECURITY.md).

## Attribution and license

The research documentation is covered by the repository's
[CC BY 4.0 license](LICENSE), subject to its stated exclusions. Separate code
and third-party license boundaries remain applicable; this documentation does
not grant rights to vendor firmware or other projects' code.

Telldus and TellStick are trademarks of their respective owners.
