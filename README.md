# TellStick ZNet Lite V2 research

Hardware and firmware research on the Telldus TellStick ZNet Lite V2, focused
on firmware 1.3.2. The findings combine static analysis of the vendor image
with bounded tests on an owned device. This project is independent of Telldus.

This is a research repository, not a firmware distribution or supported
installation. It contains no firmware images, flash dumps, credentials,
private keys, or device-specific configuration.

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
  persistent stock-API listing. A physical button press produced a
  captured state update; On/Off switching of the connected test load was
  **PHYSICALLY OBSERVED**. The WMR-252-specific RF433 load
  switching is also **PHYSICALLY OBSERVED**. These bounded
  results do not establish general radio support.
- The stock plugin loader's package-signature and startup path was exercised
  with a locally signed test plugin. A controlled MQTT publication reached an
  authenticated subscriber. This did not test arbitrary authenticated uploads
  or radio-event forwarding.
- A separate Home Assistant experiment completed owner-approved token
  issuance, selected stock-API reads, and entity creation. Polling stock API
  state reads is implemented. An On command switched on
  one connected test plug during testing (**PHYSICALLY OBSERVED**); sustained
  polling behavior, callback forwarding, complete bidirectional
  synchronization, and a connected-device Off command have not been fully
  verified.
- A production-daemon sensor callback was tested with simulated input. No RF
  signal was emitted; physical sensor-origin events and sensor-to-MQTT delivery
  remain unverified.
- A modified R1-derived runtime completed Telldus Live registration and a fresh
  OpenID callback on November 3. Later RF433/Z-Wave Live captures and separate
  R1/rc5 image validation are described in [later results](docs/later-runtime-results.md).
  RC5 was not flashed or boot-tested in the retained release records.
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

## Documentation

- [Evidence status](docs/evidence-status.md) — claim labels and evidence limits
- [Hardware verification](HARDWARE_VERIFICATION.md) — physical test matrix
- [Interfaces](docs/interfaces.md) and [authentication](docs/authentication.md)
- [RF433](docs/rf433.md) and [Z-Wave local paths](docs/zwave-local.md)
- [Local access and network isolation](docs/no-cloud.md)
- [Testing](docs/testing.md), [later runtime results](docs/later-runtime-results.md),
  and [references](docs/references.md)
- [Security policy](SECURITY.md)

## Scope and safety

Firmware modification and direct flash access can damage a device or erase its
configuration. Findings apply to the examined hardware and firmware unless
stated otherwise; tests used owned or explicitly authorized equipment. No
recovery procedure is claimed as tested unless the evidence says so. Report
sensitive issues privately as described in [SECURITY.md](SECURITY.md).

The LocalOwner material is experimental research. OpenTZnet Lite and the
LocalOwner product integration are separate development tracks and are not
released from this repository.

## Attribution and license

The research documentation is covered by the repository's
[CC BY 4.0 license](LICENSE), subject to its stated exclusions. Separate code
and third-party license boundaries remain applicable; this documentation does
not grant rights to vendor firmware or other projects' code.

Telldus and TellStick are trademarks of their respective owners.
