# Local owner authentication prototype

This directory contains independently written Python 2.7 code for an **experimental local-owner integration** with TellStick ZNet Lite V2 firmware 1.3.2. A version of the experiment was installed on one owned, modified gateway and selected authentication, API, dashboard, and device-control paths were exercised there. This is research evidence, not a supported installation, security patch, or released product. It has not been validated on an untouched-stock gateway or across other firmware versions.

`localowner/LocalOwnerAuth.py` adds an owner login, a session-backed token approval page, logout, rate limiting, and CSRF checks. It changes the approval URL in successful HTTPS `PUT /api/token` responses so a client opening that URL is directed through local login to the pending approval. Other stock API responses and JWT handling remain with `ApiManager`. A verifier must be provisioned explicitly; there is no default password. `localowner/LocalHttpsServer.py` provides the experimental CherryPy HTTPS startup with TLS and Secure cookies. `localowner/local-ui.html` is a small browser UI for system information, devices, sensors, token requests, and basic on/off actions. `provision_owner.py` generates a verifier file for offline review.

The physical experiment observed local owner login, approval of a pending stock token, stock JWT issuance, and selected authenticated system/device/sensor reads on one modified gateway. Related LocalOwner dashboard and device-control observations are bounded to the specific tests in the [physical verification matrix](../HARDWARE_VERIFICATION.md); they do not establish general hardware support. The tests also use an independently obtained and extracted copy of official firmware 1.3.2, a MIPS user-mode emulator, synthetic settings, and fake devices. Those tests exercise the original stock `ApiManager`, API routes, JWT implementation, and CherryPy package but do not simulate physical radio behavior. See [test instructions and results](../docs/testing.md).

Important limits before any broader deployment:

- Untouched-stock installation, other firmware versions, and compatibility with arbitrary third-party clients have not been established.
- The stock Telldus Live approval provider remains unchanged. The experimental local flow does not replace or validate the complete Live account login/callback sequence.
- Physical HTTPS paths were exercised, but certificate trust, exposure across network configurations, and the complete firewall/security posture still require review.
- Overlay capacity, persistence through vendor upgrades, and a rehearsed independent recovery/rollback path have not been established.
- Physical RF433 and Z-Wave behavior was observed only for the specific devices and operations listed in the verification matrix. General model coverage, exclusion, and sensor-origin RF remain unverified.

These limits mean the code should be read as an experimental research implementation, not as a security patch or a supported way to modify a gateway. No installer or vendor firmware is published. Original source in this directory is under [MIT](../LICENSE-CODE); no Telldus code is relicensed.
