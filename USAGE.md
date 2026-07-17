# protonvpn-random

`protonvpn-random` is a small Bash tool for the **Proton VPN command-line interface**.
It invokes the `protonvpn` command and does **not** control the Proton VPN graphical desktop application.

The script connects to configured Proton VPN destinations and can automatically change the connection after a randomized amount of time.

> The tool is composed of small, independent Bash functions that work together as one configurable workflow.

## Features

- Preferred destination with a configurable probability in `RANDOM` mode
- Random selection from a destination pool in `RANDOM` mode
- Ordered traversal of the pool in `ROUTE` mode
- Randomized connection-switch interval
- Configurable Secure Core probability
- Retries failed connections with increasing delays
- Granular desktop-notification controls
- Optional terminal information and countdown output
- Configuration validation and defined exit codes
- Command-line options can override matching script settings for a run

## Requirements

The following commands must be available:

- `bash`
- `protonvpn`
- `notify-send`
- `tput`

The Proton VPN Linux CLI must already be installed, configured and signed in.
The current official CLI also relies on NetworkManager and a compatible keyring and is not intended for headless systems.

## Quick start

Make the script executable:

```bash
chmod +x protonvpn-random
```

Run it:

```bash
./protonvpn-random
```

It can also be started explicitly with Bash:

```bash
bash protonvpn-random
```

Show the available command-line options:

```bash
./protonvpn-random -h
```

Supported command-line options override the corresponding values in the script for that run. They do not permanently rewrite the configuration section.

## Configuration overview

Open the script in a text editor and edit the user-configuration section near the top.

### Preferred destination

```bash
MAIN_LOCATION="US"
PROBABILITY_MAIN_LOCATION=50
```

`MAIN_LOCATION` can be a country, city or individual server identifier.
In `RANDOM` mode, `PROBABILITY_MAIN_LOCATION` controls how often it is selected:

- `0` - never use the preferred destination
- `50` - use it for approximately half of the connections
- `100` - always use it

### Destination pool

```bash
RANDOM_POOL=(
    "US-FREE#36"
    "US-GA#29-TOR"
    "New York"
    "NO"
)
```

Countries, cities and individual server identifiers may be mixed.
In `RANDOM` mode, an entry from this pool is selected when `MAIN_LOCATION` is not selected.
In `ROUTE` mode, the entries are processed in their written order.

Do not add `MAIN_LOCATION` to the pool unless you intentionally want to increase its overall probability in `RANDOM` mode.

### Selection mode

```bash
MODE="RANDOM"
```

Available values:

- `RANDOM` - select pool entries randomly; `MAIN_LOCATION` can be selected according to its probability
- `ROUTE` - process pool entries one after another in their written order

The switch time remains randomized in both modes.

### Automatic switching

```bash
SWITCH_VPN_ONOFF="ON"
SECONDS_SWITCH_VPN_MIN=3600
SECONDS_SWITCH_VPN_MAX=7200
```

- `SWITCH_VPN_ONOFF="ON"` - continue changing the VPN connection
- `SWITCH_VPN_ONOFF="OFF"` - connect once and exit the script

When switching is disabled, the established VPN connection remains active after the script exits.
The script selects a random waiting time between the minimum and maximum values.
The minimum must not be greater than the maximum.

### Secure Core

```bash
PROBABILITY_SECURECORE=50
```

- `0` - disabled
- `50` - requested for approximately half of the connection attempts
- `100` - requested for every connection attempt

Not every city or individual server supports Secure Core. An incompatible pool entry can therefore cause a failed connection attempt. The script reports the failure and continues according to its retry and selection logic.

### Notifications and terminal output

```bash
NOTIFY_ALLOFF="NO"
NOTIFY_VPN_ERROR_ONOFF="ON"
NOTIFY_VPN_OK_ONOFF="ON"
NOTIFY_EXIT_ONOFF="ON"
NOTIFY_ASK_ONOFF="ON"
PROTON_INFO_ONOFF="ON"
START_MESSAGE_ONOFF="ON"
```

Setting `NOTIFY_ALLOFF="OFF"` disables the individual desktop-notification categories.
The individual notification and terminal-output switches use `ON` or `OFF`.

### Failed connections

```bash
MAX_CON_FAILS=2
INCREASING_DELAY=10
```

`MAX_CON_FAILS` defines how many consecutive connection failures are tolerated before the script exits.
`INCREASING_DELAY` defines the delay increment between retries. With a value of `10`, consecutive failures wait approximately 10, 20, 30 seconds, and so on.

Exiting `protonvpn-random` does not automatically prove that an already active VPN connection was disconnected.

## Install as a command

Copy the script to `/usr/local/bin`:

```bash
sudo cp protonvpn-random /usr/local/bin/protonvpn-randomizer
sudo chmod 755 /usr/local/bin/protonvpn-randomizer
```

Start it from any directory:

```bash
protonvpn-randomizer
```

The configuration remains inside the copied script. Edit it with:

```bash
sudo nano /usr/local/bin/protonvpn-randomizer
```

## Start automatically after desktop login

Create the autostart directory:

```bash
mkdir -p "$HOME/.config/autostart"
```

Create:

```text
$HOME/.config/autostart/protonvpn-randomizer.desktop
```

Example:

```ini
[Desktop Entry]
Type=Application
Name=protonvpn-random
Comment=Automatically changes Proton VPN CLI connections
Exec=/usr/local/bin/protonvpn-randomizer
Terminal=true
StartupNotify=false
```

Use exactly one valid terminal setting:

- `Terminal=true` - show terminal information and the countdown
- `Terminal=false` - run without a terminal window

## Exit codes

- `0` - normal exit
- `1` - general failure
- `10` - missing dependency
- `11` - invalid configuration

Display the last exit code immediately after the script exits:

```bash
echo "$?"
```

## Kill switch during server changes

`protonvpn-random` delegates every connection change to the Proton VPN CLI. It does not implement its own firewall or kill switch.

Proton states that its kill switch blocks traffic when the VPN connection is lost and that, in most cases, it also protects while switching between VPN servers. This is not an unconditional guarantee for every platform, version or local configuration.

Enable the standard kill switch in the Linux CLI:

```bash
protonvpn config set kill-switch standard
```

Disable it:

```bash
protonvpn config set kill-switch off
```

For maximum assurance, test the actual behavior on the target system. A separately managed persistent firewall or kill-switch policy can provide stricter protection independently of the script lifecycle.

## Notes

The script depends on commands, options and destination identifiers provided by the installed Proton VPN CLI. Future CLI changes may require changes to `protonvpn-random`.

Official references:

- Proton VPN Linux CLI: <https://protonvpn.com/support/linux-cli>
- Proton VPN kill switch: <https://protonvpn.com/support/what-is-kill-switch>
