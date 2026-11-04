# First gateway test: readiness gates

This page began as a preflight checklist for the **experimental LocalOwner authentication prototype**. Later work installed the prototype on one owned firmware 1.3.2 gateway and verified selected login, token approval, API, dashboard, and control paths. A separate signed MQTT test plugin was also installed and persisted across daemon restarts. This research does not establish product readiness: broader deployment, upgrade persistence, and a tested recovery/rollback path remain open. See the [physical verification matrix](../HARDWARE_VERIFICATION.md) for the evidence and its limits.

## Baseline checks and remaining gates

1. Confirm the gateway's current LAN address through an owned network inventory. Confirm that the connection reaches the expected device rather than another host.
2. The pre-overlay test observed anonymous API rejection on stock HTTP. The later experimental overlay served selected owner-approved API reads over HTTPS. This does not establish the safety of every route or credential flow; do not send owner credentials over stock HTTP.
3. If an authorized read-only console is actually available, check `/etc/builddate`, the Python executable and package versions, service process, listeners, mount table, and free overlay space. Record checksums for the original startup file and any file that a future overlay would shadow. Do not modify the overlay in this phase.
4. Check logs for a normal boot and API request handling without issuing device commands. Confirm whether the stock web service remains on HTTP and whether the proposed HTTPS port is blocked as the reviewed image suggests.

## Why a second process is insufficient

The stock `ApiManager` keeps pending token requests in process memory. A separate CherryPy process would have a different pending-token table, even if it imported the same firmware modules. The experimental compatibility path therefore runs in the application process that handles `PUT /api/token`. Owner approval, token issuance, and selected authenticated reads were exercised on one modified gateway. Signed plugin loading in that process was separately verified using an MQTT test egg. These observations do not verify arbitrary authenticated uploads or establish safe deployment on other devices.

## Backup and rollback gate

Before any further LocalOwner deployment, an authorized operator needs an independent recovery channel and verified backups of the startup file, overlay upper layer, `/etc/telldus/Telldus.conf`, and any Z-Wave controller state that can actually be exported. These may contain credentials or network keys: preserve them in encrypted, access-controlled storage, verify checksums and restore readability, and keep values out of public logs. A 16 MiB full-flash backup has since been located outside this repository and matched against selected official firmware regions; a restore from it has **not** been tested. Overlay capacity, the deployed prototype's exact rollback behavior, and recovery after a failed boot remain open.

Any further prototype changes should use a removable overlay with an explicit return to original startup and web/API behavior. A serial or other independent administrative path must be tested before relying on rollback. Preserving `Telldus.conf` alone may not preserve controller nonvolatile memory. TLS behavior, firewall scope, and alternate owner-login routes require continued review before broader deployment.

The vendor-managed updater requires a valid detached signature before flashing, while the ordinary sysupgrade configuration list does not include the proposed startup shadow or Python plugin package. Therefore a future overlay installation must document how it is disabled, restored, or deliberately re-applied across a firmware upgrade. The board's inspected flash map does not show a redundant firmware slot. See [update and recovery paths](update-and-recovery.md) before planning either an overlay write or a custom image.

The original preflight ended before any plugin write. Later, separately authorized testing installed the MQTT plugin and the LocalOwner experiment and exercised selected physical paths. The cited tests do not establish a general rollback procedure, deployment across firmware versions, physical Z-Wave exclusion, or RF-origin sensor handling.
