# LocalOwner deployment findings

The LocalOwner prototype was exercised on one modified TellStick ZNet Lite V2
running firmware 1.3.2. The tested paths included HTTPS login, local approval
of a stock API token, JWT issuance, selected API reads, dashboard access, and
specific radio operations. A separate signed MQTT plugin also persisted across
daemon restarts. The [physical verification matrix](../HARDWARE_VERIFICATION.md)
records each result and its evidence type.

The token compatibility path runs in the stock application process because
`ApiManager` keeps pending requests in memory; a separate CherryPy process
would not share that state. Successful tests do not establish arbitrary
authenticated uploads or safe deployment on other devices.

## Open technical constraints

- The 16 MiB flash backup was compared with selected official firmware
  regions, but has not been restored. Rollback after a failed boot is untested.
- The stock updater requires a detached vendor signature. Its normal
  configuration-preservation list does not include the prototype's startup
  shadow or Python package; overlay persistence through an upgrade is unknown.
- The inspected flash map has no verified redundant firmware slot. Controller
  nonvolatile state may be separate from `Telldus.conf`.
- Untouched-stock installation, other firmware versions, broader TLS and
  firewall behavior, and physical Z-Wave exclusion remain unverified.

See [update and recovery paths](update-and-recovery.md) for the static update
analysis and recovery limits.
