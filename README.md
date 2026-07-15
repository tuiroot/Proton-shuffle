# protonvpn-random

`protonvpn-random` is a small Bash tool for the Proton VPN command-line interface.

It connects to configurable Proton VPN countries, optionally prefers a main location, enables Secure Core with a configurable probability, and changes the VPN connection after random time intervals.

The script uses the `protonvpn` CLI command directly. It does not control the Proton VPN desktop application.

## Features

* Random selection from a configurable country pool
* Optional preferred main country
* Configurable probabilities for the main country and Secure Core
* Random connection-switch intervals
* Automatic retries with increasing delays
* Proton VPN Kill Switch check before connecting
* Optional desktop notifications
* Configuration validation before startup
* Prevention of multiple simultaneous instances

## Quick start

Make the script executable:

```bash
chmod +x protonvpn-random
```

Run it:

```bash
./protonvpn-random
```

The Proton VPN CLI must already be installed, configured and logged in.

## Documentation

Detailed installation, configuration, notification, autostart and Kill Switch instructions are available in:

[USAGE.md](USAGE.md)

## Notes

`protonvpn-random` depends on the commands and output format provided by the Proton VPN CLI. Future CLI changes may require corresponding changes to the script.

The integrated Kill Switch handling uses Proton VPN's standard Kill Switch. It is not a replacement for a separate permanent firewall-based Kill Switch.
