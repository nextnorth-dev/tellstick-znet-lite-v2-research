# LocalOwner prototype

This independently written Python 2.7 prototype explores local owner
authentication for TellStick ZNet Lite V2 firmware 1.3.2. It was exercised on
one modified gateway; it is not a released product or supported installation.

`localowner/LocalOwnerAuth.py` adds owner login, session-bound approval,
logout, rate limiting, and CSRF checks. On HTTPS token requests it redirects
the stock approval link through local login; token issuance and other API
responses remain with the stock `ApiManager`. An owner verifier must be
provisioned explicitly. `LocalHttpsServer.py` configures CherryPy TLS and
secure session cookies. `local-ui.html` provides a small interface for API
reads and basic device actions; `provision_owner.py` creates the verifier.

Selected login, approval, JWT, API, dashboard, and device-control paths were
tested on the modified gateway. Offline tests use the original firmware
modules with synthetic settings and devices; they do not exercise physical
radio behavior. See [test methods and results](../docs/testing.md) and the
[physical verification matrix](../HARDWARE_VERIFICATION.md).

Installation on untouched firmware, other versions, general radio support,
certificate trust across network configurations, upgrade persistence, and
recovery after a failed boot remain open. See [LocalOwner deployment
findings](../docs/hardware-readiness.md) and [update and recovery
paths](../docs/update-and-recovery.md).

Original code in this directory is licensed under [MIT](../LICENSE-CODE).
No Telldus code is relicensed.
