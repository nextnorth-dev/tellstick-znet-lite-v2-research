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
requests only; stock OpenID account login and untouched-stock token approval
were not tested.

The firmware generates JWT signatures with its API token key. Device-specific
material protects the stored key but is not itself the signing key. Neither
key material nor credentials are included here.

## OpenID and transport limits

The examined Telldus login path uses OpenID discovery and a browser
redirect/callback. A certificate-verified HTTPS request to the discovery
endpoint succeeded using the firmware's Python 2.7 stack. A complete account
login, callback, authenticated-cookie exchange, and clock-dependent
verification were not tested. This research does not claim a general OAuth
flow.

The original web startup uses HTTP and does not configure a Secure session
cookie. An anonymous session cookie was observed on the owned test gateway
with `HttpOnly`, but without `Secure` or `SameSite`; its value was not retained.
These observations apply to the tested firmware and routes only.

## Experimental local-control research

The repository includes an experimental LocalOwner prototype that was used
for limited tests on one modified firmware 1.3.2 gateway. It retains the stock
token issuer; selected login, approval, JWT, and read operations were observed
on that gateway. This is research evidence, not a supported installation,
security review, or product release. See the [physical verification matrix](../HARDWARE_VERIFICATION.md)
and [evidence register](evidence-status.md) for scope and limitations.
