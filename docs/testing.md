# Test methods and results

Physical results are from one owned TellStick ZNet Lite V2 running firmware
1.3.2 with related LocalOwner extensions active on a modified gateway. Later
R1-derived and Live-processing results are identified separately. They do not
establish behavior on an untouched-stock installation.

## Hardware tests

Targeted tests observed the stock UDP 30303 discovery response, UDP 42314
listener registration and version notice, and rejection of anonymous API
requests. No RF433 command or event was verified through UDP 42314.

The stock plugin loader was exercised on the physical gateway. A test egg
signed by a public key deliberately added to the device keyring was accepted;
the plugin's startup entry point ran, and an authenticated MQTT subscriber
received the controlled test publication. This does not establish arbitrary
web-upload behavior, trust of unknown signers, or radio-event delivery.
Further loader observations and their limits are in
[SECURITY_STATIC_REVIEW.md](../SECURITY_STATIC_REVIEW.md).

The `sensorValueUpdated` callback was exercised in the production daemon with
controlled simulated input. The observer received the queued callback, but
the test emitted no RF and created no persistent sensor reading. Physical
sensor-origin events and sensor-to-MQTT delivery remain unverified; forwarding
is not implemented in the tested one-shot plugin.

With the experimental LocalOwner overlay active, physical tests observed local
login, stock token approval/JWT use, and selected system/device/sensor API
reads. A scoped dual-interface IPv4 egress-block cold boot passed those local
checks once; it is not proof of universal or long-term zero egress. Separate
Z-Wave tests recorded inclusion, `nodeAdded`, and a physical-button state
update. The owner-reported On/Off switching of one connected load is an
**OWNER-REPORTED PHYSICAL OBSERVATION**. A WMR-252-specific RF433 path
produced a recorded adapter acceptance; the pairing indication and load
On/Off are **OWNER-REPORTED PHYSICAL OBSERVATION**. These results and their
individual limits are recorded in the
[hardware verification matrix](../HARDWARE_VERIFICATION.md).

## Later runtime and release verification

[Later runtime results](later-runtime-results.md) records the November 3
modified-R1 registration/OpenID callback, November 2 Live RF433/Z-Wave
captures, and separate R1/rc5 artifact acceptance and boot limits. The rc5
release record reports 33 Python tests passed, one framework-dependent skip,
three JavaScript suites passed, and seven runtime files compiled; it explicitly
records no rc5 flash or boot test. These do not supersede the earlier
prototype tests or establish untouched-stock login.

The MQTT plugin has subscriber-confirmed delivery during controlled import
and during a later live daemon restart under scoped IPv4 isolation. Cold-boot
publication has a startup log but no subscriber spanning the reboot.

## Offline and emulated tests

These results were recorded on 2026-10-24 and 2026-10-25 against official
firmware 1.3.2. The image, extracted root filesystem, and MIPS emulator are
not included.

| Test | Code exercised | Recorded result | Limit |
| --- | --- | --- | --- |
| Firmware imports | Original MIPS Python 2.7, CherryPy 3.7, API and OpenID provider via QEMU user-mode emulation | PASS | Not a full-system emulator |
| Public discovery TLS | Firmware Python/Requests/OpenSSL stack, certificate-verified HTTPS GET | PASS | Not a complete account login |
| CherryPy TLS | Original CherryPy serving a temporary loopback certificate | PASS | No real-address certificate |
| Local-owner integration | [`test_firmware_integration.py`](../tests/test_firmware_integration.py), original API/JWT modules with synthetic settings and devices | 9/9 PASS | No serial or radio hardware |
| End-to-end HTTPS | [`run_local_https_probe.sh`](../tests/run_local_https_probe.sh), CherryPy routing, local login, approval, JWT, API read, mocked device operation, logout | PASS | Loopback and synthetic account/devices |
| Local-only web-provider configuration | Same HTTPS probe with `LOCALOWNER_LOCAL_ONLY=1` | PASS | Live backend and physical application startup not exercised |
| Z-Wave command paths | [`test_zwave_offline.py`](../tests/test_zwave_offline.py), original node/controller bytecode with fake serial/message sink | 2/2 PASS | No radio network or transmission |
| Stock `system/info` authorization | [`test_original_system_info_http.py`](../tests/test_original_system_info_http.py), original CherryPy/API modules over loopback HTTP | 6/6 PASS | Valid case uses synthetic board values |
| Vendor upgrade verifier | [`test_stock_upgrade_verification.py`](../tests/test_stock_upgrade_verification.py), original `tellstick-upgrade.py:verifyFile` | 4/4 PASS | GPG subprocess mocked; no vendor signature or flash tested |
| UDP 42314 parser/listener | [`test_udp42314_original_offline.py`](../tests/test_udp42314_original_offline.py) | 7/7 PASS | Fake sockets; no serial/radio hardware |
| UDP 42314 queue/loopback checks | Original handler, protocol, message, adapter queue, and UDP server | 9/9 PASS | Separate offline harness not included; mock queue and loopback only |

The isolated integration cases covered owner login, rate limiting, CSRF,
session expiry/logout, synthetic JWT cases, and stock API routes. The
loopback HTTPS probe used temporary synthetic credentials and certificate
material and removed its temporary files. The five-test protocol/parser model
from an earlier phase applies stricter validation than the original firmware;
it does not establish that the firmware rejects the same input. A ready-made
radio-command sender is intentionally not published.

Tests under [`tests/`](../tests/) that use original firmware modules require
a separately obtained official image, extracted root filesystem, and
compatible user-mode MIPS Python runner. Emulated command acceptance,
adapter-queue activity, or HTTP success does not prove radio delivery.

Example offline invocation, with external paths kept outside the checkout:

```sh
export FIRMWARE_ROOTFS=/path/to/extracted-1.3.2-rootfs
export QEMU_MIPS=/path/to/qemu-mips

"$QEMU_MIPS" -L "$FIRMWARE_ROOTFS" \
  -E PYTHONHOME="$FIRMWARE_ROOTFS/usr" \
  -E PYTHONPATH="$PWD/prototype" \
  "$FIRMWARE_ROOTFS/usr/bin/python2.7" tests/test_firmware_integration.py
"$QEMU_MIPS" -L "$FIRMWARE_ROOTFS" \
  -E PYTHONHOME="$FIRMWARE_ROOTFS/usr" \
  "$FIRMWARE_ROOTFS/usr/bin/python2.7" tests/test_zwave_offline.py
"$QEMU_MIPS" -L "$FIRMWARE_ROOTFS" \
  -E PYTHONHOME="$FIRMWARE_ROOTFS/usr" \
  "$FIRMWARE_ROOTFS/usr/bin/python2.7" tests/test_udp42314_original_offline.py
```

The shell probes bind to loopback and create temporary synthetic TLS material.
Obtain firmware lawfully and keep firmware, extracted files, credentials, and
device-specific data outside the repository.
