# Authentication in TellStick ZNet Lite V2 firmware 1.3.2

The examined stock firmware exposes a local HTTP API protected by signed JWTs.
The stock web approval flow for a new token depends on Telldus Live OpenID;
this is an approval dependency, not a requirement on every request made with
an already valid token. These are findings about firmware 1.3.2, not a claim
that every device operation can be performed offline.

## Token and API behavior

`api.ApiManager` accepts token requests and issues or renews JWTs.
`web.base.Server` uses a web session for the approval interface. In the
examined image, `tellduslive.web.WebRequestHandler` checks the returned OpenID
identity against the device's registered account.

For `/api/system/info`, the server may skip web-session authentication while
still requiring a valid API token. Loopback tests using the original API and
CherryPy modules exercised missing, empty, malformed, expired, and valid
synthetic token cases. The unauthenticated cases returned 401. On one modified
test gateway, an experimental local-owner configuration obtained a stock JWT
and used selected system, device, and sensor read routes. This verifies those
requests only; untouched-stock token approval was not tested. The later
modified R1-derived OpenID result is described below.

The firmware generates JWT signatures with its API token key. Device-specific
material protects the stored key but is not itself the signing key. Neither
key material nor credentials are included here.

## OpenID and transport limits

The examined Telldus login path uses OpenID discovery and a browser
redirect/callback. A certificate-verified HTTPS request to the discovery
endpoint succeeded using the firmware's Python 2.7 stack. On November 3,
2026, a modified R1-derived runtime received and applied Live registration,
then completed a fresh OpenID callback to authenticated Index according to
the browser/session record. The JSONL records callback execution and Boolean
registration state, not the redirect destination. R1 retains the stock
account checks with a targeted discovery compatibility correction. See
[later runtime results](later-runtime-results.md#telldus-live-registration-and-openid-november-3).
Untouched-stock end-to-end login, general clock-error handling, and RC5
runtime verification remain unverified; no general OAuth flow is claimed.

The original web startup uses HTTP and does not configure a Secure session
cookie. An anonymous session cookie was observed on the owned test gateway
with `HttpOnly`, but without `Secure` or `SameSite`; its value was not retained.
These observations apply to the tested firmware and routes only.

## Experimental local-control research

The published LocalOwner snapshot retains the stock token issuer and has
offline/emulated tests. Related, extended LocalOwner code was deployed on a
modified 1.3.2 gateway; selected login, approval, JWT, read, and device-control
paths were observed there. Exact source-snapshot deployment is not established.
See the [prototype description](../prototype/README.md),
[physical verification matrix](../HARDWARE_VERIFICATION.md), and
[evidence register](evidence-status.md).

## Dropbear and owner SSH

The original ROM `/etc/config/dropbear` sets `PasswordAuth on`,
`RootPasswordAuth on`, and port 22. Its first-boot `10_dropbear` script sets
`RootPasswordAuth off` through UCI. A ROM default therefore does not establish
the later active setting. The init script maps disabled `PasswordAuth` to
Dropbear `-s` (password login disabled) and disabled `RootPasswordAuth` to `-g`
(root password login disabled). Neither option alone prohibits authorized
public-key root login; root-login policy is a separate setting.

Earlier reports describe active root password authentication as on; another
reported observation says off, but a dated primary snapshot for that off value
was not located. No single active-UCI value is established across these runs.
Effective runtime policy additionally depends on the launched process options.
The November 3 verification records a fresh, non-multiplexed public-key root
login and a port-22 listener; no password was retested. Owner access was
independent of the embedded Telldus RemoteSupport private key, which was not
used for that login. See [remote support](remote-support.md) for the distinct
outbound-tunnel authentication boundary.
