# Later runtime and firmware results

These results extend the October 2026 firmware 1.3.2 research. The environments
are distinct: the original stock image, a modified R1-derived runtime with
Telldus Live enabled, the separately built Lite rc5 image, and LocalOwner
extensions. A matching module hash does not make those environments equivalent.

## Telldus Live registration and OpenID, November 3

A bounded observer wrapped the existing registration and callback handlers on
the modified R1-derived runtime. Its retained `observation.jsonl` records:

| November 3, 2026 (CET) | Observation |
| --- | --- |
| 11:45:38 | Telldus Live sent `registered`. |
| 11:45:49 | The existing handler applied it; `registered`, `email_present`, and `connected` were true. |
| 11:47:37 | The OpenID callback handler ran with those flags true; the observer recorded no failure template or reason. |
| 11:48:37 | The observer removed its hooks at its bounded timeout. |

The contemporaneous browser/session record reports that the fresh callback
returned to authenticated Index. The JSONL records callback execution and
runtime flags, not the redirect destination or session-cookie contents.
Together these establish a successful account callback on this modified
runtime, not untouched-stock end-to-end login or a general OAuth flow.

The deployed OpenID bytecode hash recorded in `final-runtime-check.json`
matches the handler extracted independently from both R1 and rc5:

```text
d1649d519525c3c1c39a55f28ea8ffe14808a6092015a7ed6692c566d40fcb6d
```

R1 contains a targeted OpenID compatibility correction: HTTPS discovery,
session-held transaction state, and constrained handling of Telldus's observed
OP-Identifier response. It retains the account-email check and delegates other
discovery cases to the original verifier. This is compatibility with the stock
authentication architecture, not byte identity with the original stock handler.
Five focused endpoint/identity/delegation regression tests passed. They use
substituted dependencies and do not replace the recorded browser result.

An earlier process displayed “not activated.” The handler reaches that result
when registration is false or the account email is empty. The successful later
run does not identify which condition failed in the earlier process. Repeated
callbacks and transactions spanning restarts are not fresh-login tests.

## RF433: reception and separate command paths

October daemon logs contain incoming `arctech` / `selflearning` frames. Those
entries do not identify the sender or action and are not physical sensor-value
tests. The WMR-252 LocalOwner route separately recorded adapter acceptance;
pairing indication and connected-lamp On/Off were owner-observed. Generic
RF433 record CRUD tests sent no RF. Stock API acceptance or saved state alone
is not a receiver acknowledgement.

The November 2 Nexa self-learning-dimmer capture contains 72 JSONL records:
27 RX, 34 TX, ten operator markers, and one diagnostic. Live processing handled
learn, On/Off, and dim commands at 64, 128, and 192, with application ACKs and
device events. The operator reported physical On/Off and dimmer response;
brightness was not measured. Neither the ACKs nor the events are receiver-side
RF acknowledgements. The commercial model was not independently identified.

These LocalOwner, stock API, and Live results do not verify RF transmission or
event forwarding through UDP 42314. That transport's physical evidence remains
listener registration and a version notice; its queue paths have offline tests.

## Later Z-Wave captures, November 2

Live application captures record normal inclusion, a separate
`addSecureNodeToNetwork` request, node-added and interview notifications,
device events, exclusion status messages, and `deviceremoved`. The secure
capture has 161 records, including eight On/Off commands and two exclusion
requests. A secure request and device-added event do not prove completed
security negotiation or key exchange; no serial or RF capture establishes it.

The second failed-node-removal retest has 16 records. It records a failed-node
probe request, `nodeInfo` reporting `isFailed:true`, a `removeFailedNode`
request, and an application removal event. Firmware code establishes the
NOP / `IS_FAILED_NODE_ID` and `REMOVE_FAILED_NODE` paths; the application log
does not capture the controller's reply bytes or independently establish its
final persistent membership table. Local-entry forgetting is a different
operation and can be undone by controller re-enumeration at daemon startup.

Earlier physical-button callbacks and owner-observed load switching remain
valid separate observations. Their product-identification discrepancy is
preserved in the [hardware matrix](../HARDWARE_VERIFICATION.md); later captures
must not silently resolve it or be attributed to the earlier LocalOwner route.

## MQTT evidence boundaries

The signed test plugin loaded and ran its startup entry point. A controlled
import produced subscriber-confirmed broker delivery. A later live daemon
restart under dual-interface IPv4 egress isolation also had an independent
subscriber receipt. Other starts recorded successful publish calls only.
The isolated cold boot logged startup publication but had no subscriber
spanning the reboot. Sensor-to-MQTT forwarding was not implemented or tested.

## Firmware artifacts, installation, and recovery

| Artifact or experiment | Established result | Limit |
| --- | --- | --- |
| Stock Research R1 | Image integrity and on-device `sysupgrade -T` acceptance; its handler bytecode matches the tested modified runtime | The retained November 2 `device-test.json` explicitly says the candidate was not flashed. |
| Initial rc4 installation attempt | Acknowledgement and reboot left the previous ROM installed; the staging path was lost at the RAMFS transition | Reboot and upload filename did not establish installation. |
| Later owner rc4 installation | Retained rc5 validation records an installed rc4 ROM match, SHA-256 `0b6d5d621a69d4941443b8abbf2853b279832590d01512c96861acaf8d323d14` | Normal installation/boot evidence, not recovery from a failed boot. |
| Lite rc5 | Build/content checks, Python/JavaScript tests, and native image acceptance | Not flashed or boot-tested in the retained release records. |

The staging repair uses canonical `/tmp` storage: stock `supivot` carries that
mount into RAMFS but does not recreate the normal `/var` symlink. Installation
acceptance is separate from verification of a new boot and the compressed ROM.

Exact rc5 release identity:

```text
Artifact: OpenTZnet-Lite-v1.0.0-rc5.bin
Source commit: d93be21291fb7955356bd8e66272c45e2e71be7b
Release tag: v1.0.0-rc5
Size: 13369348 bytes
SHA-256: e06c5d39df6744103b5c58fc369fb765e42f3ce5545baf9bc8f32c6cc3f37348
R1 base SHA-256: 47b8c920f3eeafdf5e55a18486aa4fec59275b869dbeaf533264a5ae139e2766
```

The retained `VALIDATION.json` records 33 Python 2.7 tests passed and one
framework-dependent skip, three JavaScript suites passed, seven image runtime
files compiled with Python 2.7, unchanged kernel, and successful content checks.
The on-device native validator and `sysupgrade -T` accepted the exact image;
`physical-validation.json` records exit 0, no flash, and no reboot.

Artifact acceptance, normal boot, functional runtime verification, failed-boot
recovery, and full-flash restoration are separate claims. Neither the later
rc4 result nor rc5 acceptance proves the last two. The bootloader recovery
format and procedure remain unverified; no proprietary image is distributed here.

## Evidence provenance

The retained primary records include `observation.jsonl` and its observer,
the November 3 browser/session record, R1 `device-test.json`, rc5
`RELEASE.json`, `VALIDATION.json`, and `physical-validation.json`, the October
RF/isolated-MQTT logs, and the November 2 typed application captures. These
records are not a redistributable firmware bundle; private captures and
internal working files are not included. Reported browser/physical observations
are distinguished from machine-recorded fields above.

The retained capture identifiers are `433-nexa-dimmer-test1/capture.jsonl`,
`zwave-secure-test1/capture.jsonl`, and
`zwave-failed-node-removal-test2/capture.jsonl`. The October MQTT isolation
sequence is recorded in `wan-isolation-evidence-2026-10-28.txt`; incoming
Arctech frames occur in `logread-after-first-restart.raw` and
`logread-after-respawn.raw`. These names identify retained evidence, not files
shipped in this repository. Source history includes the development
protocol records `b3483c7`, `ae8cef9`, and `1d3fd00`, and the Lite installer
repair `0663cdc` followed by rc5 source `d93be21`. These identify separate
development histories, not additional commits in this Research repository.
