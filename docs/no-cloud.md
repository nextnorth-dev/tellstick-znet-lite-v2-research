# Local access and network-isolation research

The results below cover stock local interfaces and selected LocalOwner paths
on one modified firmware 1.3.2 gateway.

## What stock firmware exposes

The stock application has a local HTTP API protected by signed JWTs, RF433
event/command paths, and internal Z-Wave controller code. The stock new-token
approval flow depends on Telldus Live OpenID. A valid JWT can be used on some
local API operations, but the existence of a local interface does not establish
that onboarding, every device operation, or event delivery works without the
cloud.

UDP 42314 accepts a listener registration and keeps one event-recipient
address. A later registration replaces it. The examined handler has no
authentication, per-client ownership, or application-level delivery
acknowledgement; RF433 event reception and command delivery were not verified
through this service. See [UDP 42314](udp-42314.md).

## Experimental local-owner observations

An experimental LocalOwner overlay was exercised on one modified 1.3.2
gateway. The following selected paths were observed physically:

- Local HTTPS login, local approval of a pending stock token, stock JWT
  issuance, and authenticated system, device, and sensor reads.
- A cold boot with temporary IPv4 egress-block rules on both active network
  interfaces. Local login, token approval, JWT issuance, selected API reads,
  and dashboard access succeeded during that single test.
- Z-Wave inclusion through the stock controller, a captured `nodeAdded`
  callback, node appearance in the stock API after daemon restart, and a
  captured physical-button-origin state change. On/Off switching of one
  connected test load was an **OWNER-REPORTED PHYSICAL OBSERVATION**.
- WMR-252-specific RF433 control with pairing indication and On/Off of its
  connected load (**OWNER-REPORTED PHYSICAL OBSERVATION**). This does not
  establish general pairing or receiver-origin state feedback.

These are narrowly scoped results, not compatibility guarantees. The
dashboard and LocalOwner code remain experimental research. The owner-control
path has not had an independent security review or an untouched-stock
installation and rollback rehearsal. See the [physical verification
matrix](../HARDWARE_VERIFICATION.md) and [evidence register](evidence-status.md).

## What remains unverified

- Universal or long-duration absence of outbound traffic, IPv6 isolation, and
  repeatability across boots or devices.
- Untouched-stock Telldus Live end-to-end login. A modified R1-derived runtime
  completed registration and a fresh callback on November 3; that cloud-enabled
  run is separate from the LocalOwner isolation tests. See
  [registration evidence](later-runtime-results.md#telldus-live-registration-and-openid-november-3).
- RF433 sensor-origin events, generic pairing across device models, and
  receiver-origin state feedback.
- Sensor-to-MQTT forwarding. A `sensorValueUpdated` callback was exercised in
  the production daemon using simulated input only; no RF was emitted and no
  persistent sensor reading was created.
- Home Assistant's stock-API state polling is implemented, but sustained
  polling behavior, callback forwarding, and complete bidirectional
  synchronization have not been fully verified.
- General Z-Wave command delivery, independently verified post-exclusion
  controller membership, association management,
  controller-NVM backup/recovery, and radio behavior under egress isolation.
  Later Live captures record exclusion and removal callbacks, separately from
  LocalOwner and cloud-isolation tests; see [later results](later-runtime-results.md).
- Recovery from failure of the experimental overlay.
