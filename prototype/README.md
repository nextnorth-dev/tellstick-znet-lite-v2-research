# LocalOwner prototype

This independently written Python 2.7 prototype explores local owner
authentication for TellStick ZNet Lite V2 firmware 1.3.2. This is the published
offline/emulation snapshot, not the complete later deployed implementation.

`localowner/LocalOwnerAuth.py` adds owner login, session-bound approval,
logout, rate limiting, and CSRF checks. On HTTPS token requests it redirects
the stock approval link through local login; token issuance and other API
responses remain with the stock `ApiManager`. An owner verifier must be
provisioned explicitly. `LocalHttpsServer.py` configures CherryPy TLS and
secure session cookies. `local-ui.html` provides a small interface for API
reads and basic device actions; `provision_owner.py` creates the verifier.

Offline tests of this snapshot use original firmware modules with synthetic
settings and devices; they do not exercise physical radio behavior. Related,
extended LocalOwner code was deployed on a modified gateway, where login,
approval, JWT, API, dashboard, and selected device-control paths were tested.
The deployed extensions include remember-me, RF433 administration, Z-Wave
diagnostics, and an MQTT configuration summary absent from this snapshot.
Its exact byte-identical deployment is not established. See
[test methods and results](../docs/testing.md) and the
[physical verification matrix](../HARDWARE_VERIFICATION.md). Later R1/RC5
cloud-enabled firmware and subsequent LocalOwner development are separate
environments; none is silently substituted for this prototype.

Installation on untouched firmware, other versions, general radio support,
certificate trust across network configurations, upgrade persistence, and
recovery after a failed boot remain open. See [LocalOwner deployment
findings](../docs/hardware-readiness.md) and [update and recovery
paths](../docs/update-and-recovery.md).

Original code in this directory is licensed under [MIT](../LICENSE-CODE).
No Telldus code is relicensed.
