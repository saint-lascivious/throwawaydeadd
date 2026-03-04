```bash
     _   _                                            _             _    _ 
    | |_| |_  _ _ _____ __ ____ ___ __ ____ _ _  _ __| |___ __ _ __| |__| |
    |  _| ' \| '_/ _ \ V  V / _` \ V  V / _` | || / _` / -_) _` / _` / _` |
     \__|_||_|_| \___/\_/\_/\__,_|\_/\_/\__,_|\_, \__,_\___\__,_\__,_\__,_|
                                              |__/
```

# [throwawaydeadd](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/throwawaydeadd "throwawaydeadd - An oddly complex VRRP check/notify script") - An oddly complex VRRP check/notify script

**Author:** saint-lascivious (Hayden Pearce) (C) 2026

**License:** GNU General Public License Version 3.0 or later (GPL-3.0+)

## Description

A wildly convoluted VRRP tracking script that accepts the service name and mode as command-line arguments, with optional configuration file support.

The script checks if the specified service is running in "check" mode and performs actions based on VRRP state changes in "notify" mode, using a simple configuration file format to define actions for each state and role.

In check mode, service availability is cached per-service with a configurable timeout to avoid redundant process checks during frequent script invocations. Stale cache entries are automatically cleaned up.

In notify mode, service actions are executed via the detected init system: SysV (service), systemd (systemctl), OpenRC (rc-service), runit (sv), or macOS launchd (launchctl). The detection order is configurable.

## Features

- **Simple:** Monitor and optionally take action on a service by providing the service name and operation mode.

- **Multi-Init System Support:** Works with SysV, systemd, OpenRC, runit, and macOS launchd with automatic detection. Probably.

- **Check & Notify Modes:** Operates as a keepalived check script (check mode) or notify script (notify mode) depending on arguments provided.

- **State-Based Actions:** Perform configured actions on services based on the current VRRP role (MASTER/BACKUP) and state (MASTER/BACKUP/FAULT).

- **Efficient Caching:** Service state caching with configurable timeouts reduces redundant process checks during frequent invocations.

- **Low Dependency:** POSIX shell construction with minimal external dependencies.

- **Configurable:** Optional external configuration via `.conf` files with the same name as the monitored service.

- **Security:** Config file validation prevents sourcing of malformed files with strict permission and ownership checks.

## Installation

### Quick Start

All commands should be run as `root` or with `sudo`.

For examples, we'll monitor the Pi-hole `pihole-FTL` service:

1. Download the script:

   ```bash
   sudo wget -O /usr/local/bin/throwawaydeadd https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/throwawaydeadd
   ```

2. Make it executable:

   ```bash
   sudo chmod +x /usr/local/bin/throwawaydeadd
   ```

3. Or use the built-in installer:

   ```bash
   throwawaydeadd install /usr/local/bin
   ```

### Configuration

Configuration is optional. If no configuration file is found, the script proceeds with defaults.

#### Configuration File Location

The script searches for configuration files in the following order:

1. Explicitly specified via `--config` option
2. In the script's directory with the service name: `${SERVICE}.conf`
3. In the script's directory: `throwawaydeadd.conf`

#### Configuration Options

**IS_MASTER** (case-insensitive)

Declares whether this instance should be considered a MASTER or BACKUP server. Used in notify mode to determine which actions to perform based on VRRP state changes.

- Valid values: `yes`, `true`, `1`, `no`, `false`, `0`
- Default: `false`

**Example:**
```bash
IS_MASTER=true
```

---

**State Action Variables**

Define actions to perform when VRRP state changes. Format: `${ROLE}_IS_${STATE}=<action>`

- `ROLE`: `MASTER` or `BACKUP` (determined by `IS_MASTER` setting)
- `STATE`: `MASTER`, `BACKUP`, or `FAULT` (passed by keepalived)
- `action`: Service action to execute

**Valid Actions:**
- `none` - Perform no action (default)
- `start` - Start the service
- `stop` - Stop the service
- `restart` - Restart the service
- `reload` - Reload the service configuration

**Example:**
```bash
MASTER_IS_MASTER=none
MASTER_IS_BACKUP=stop
BACKUP_IS_MASTER=start
BACKUP_IS_BACKUP=none
```

---

**INIT_SYSTEM_ORDER** (default: `service,systemctl,rc-service,sv,launchctl`)

Comma-separated list of init systems to attempt in order. The script will use the first available system.

**Example:**
```bash
INIT_SYSTEM_ORDER=systemctl,service,rc-service
```

---

**STATE_CACHE_DIR** (default: auto-detect)

Directory for state cache files. The script automatically detects and prefers RAM-based filesystems in this order:

1. `/dev/shm` (Linux tmpfs)
2. `/tmp`
3. `/run`
4. Current directory (fallback)

**Example:**
```bash
STATE_CACHE_DIR=/dev/shm
```

---

**STATE_CACHE_TIMEOUT** (default: `30` seconds)

How long to cache service state before re-checking. Set to `0` to disable caching.

**Example:**
```bash
STATE_CACHE_TIMEOUT=30
```

---

**STATE_CACHE_CLEANUP** (default: `300` seconds)

Remove cache files older than this many seconds.

**Example:**
```bash
STATE_CACHE_CLEANUP=300
```

---

**VERBOSITY** (default: `0`)

Logging verbosity level:

| Level | Description |
| :---: | :--- |
| `0` | No logging |
| `1` | Errors only |
| `2` | Errors and info messages |
| `3` | All messages including debug |

**Example:**
```bash
VERBOSITY=2
```

Can also be set via environment variable `THROWAWAYDEADD_VERBOSITY`.

#### Example Configuration File

Create `/etc/throwawaydeadd/pihole-FTL.conf`:

```bash
# Master/Backup role declaration
IS_MASTER=false

# Logging verbosity (0=none, 1=error, 2=info, 3=debug)
VERBOSITY=2

# Service state caching (seconds)
STATE_CACHE_TIMEOUT=30
STATE_CACHE_CLEANUP=300

# Init system detection order
INIT_SYSTEM_ORDER=systemctl,service,rc-service,sv,launchctl

# Actions based on VRRP role and state transitions
MASTER_IS_MASTER=none
MASTER_IS_BACKUP=stop
MASTER_IS_FAULT=restart
BACKUP_IS_MASTER=start
BACKUP_IS_BACKUP=none
BACKUP_IS_FAULT=restart
```

## Usage

### Basic Usage

**Check Mode** - Test if a service is running:

```bash
throwawaydeadd pihole-FTL check
```

Exits with `0` if service is running, `1` if not running.

**Notify Mode** - Execute actions based on VRRP state change, example:

```bash
throwawaydeadd pihole-FTL notify VRRP myinstance MASTER 150
```

Arguments passed by keepalived:
- `TYPE`: VRRP instance type
- `NAME`: VRRP instance name
- `STATE`: New VRRP state (MASTER, BACKUP, FAULT)
- `PRIORITY`: VRRP priority value

### Command Line Options

| Option | Long Form | Description |
| :---: | :---: | :--- |
| `h` | `--help` | Display help message |
| `v` | `--version` | Display version information |
| `s` | `--show` | Display warranty (w) or license conditions (c) |
| `c` | `--config` PATH | Path to configuration file |
| `f` | `--flush` | Remove all cached state files |
| `i` | `--install` PATH | Install script to location (default: /usr/local/bin/keepalived/throwawaydeadd) |
| `u` | `--uninstall` PATH | Uninstall script from location |

### Examples

```bash
# Check if pihole-FTL is running
throwawaydeadd pihole-FTL check

# Perform notify action with VRRP arguments
throwawaydeadd pihole-FTL notify VRRP myinstance MASTER 150

# Use explicit config file
throwawaydeadd pihole-FTL check --config /etc/throwawaydeadd/pihole-FTL.conf

# Set verbosity via environment variable
THROWAWAYDEADD_VERBOSITY=3 throwawaydeadd pihole-FTL check

# Flush all cached state files
throwawaydeadd flush

# Install to system
sudo throwawaydeadd install /usr/local/bin

# Uninstall from system
sudo throwawaydeadd uninstall /usr/local/bin/keepalived/throwawaydeadd

# Display warranty information
throwawaydeadd show w

# Display license conditions
throwawaydeadd show c
```

## keepalived Integration

### MASTER Configuration Example

Create `/etc/keepalived/keepalived.conf`:

```bash
global_defs {
    enable_script_security
    max_auto_priority 99
    router_id MASTER
    script_user root
}

vrrp_script pihole-FTL {
    script "/usr/local/bin/throwawaydeadd pihole-FTL check"
    interval 5
    timeout 4
    weight -10
    rise 12
    fall 2
}

vrrp_instance pihole-FTL {
    state MASTER
    interface eth0
    virtual_router_id 99
    priority 150
    advert_int 1
    unicast_src_ip 192.168.1.10
    unicast_peer {
        192.168.1.20
    }
    authentication {
        auth_type PASS
        auth_pass secret
    }
    virtual_ipaddress {
        192.168.1.99/24
    }
    track_script {
        pihole-FTL
    }
    notify "/usr/local/bin/throwawaydeadd pihole-FTL notify"
}
```

### BACKUP Configuration Example

```bash
global_defs {
    enable_script_security
    max_auto_priority 99
    router_id BACKUP
    script_user root
}

vrrp_script pihole-FTL {
    script "/usr/local/bin/throwawaydeadd pihole-FTL check"
    interval 5
    timeout 4
    weight -10
    rise 12
    fall 2
}

vrrp_instance pihole-FTL {
    state BACKUP
    interface eth0
    virtual_router_id 99
    priority 145
    advert_int 1
    unicast_src_ip 192.168.1.20
    unicast_peer {
        192.168.1.10
    }
    authentication {
        auth_type PASS
        auth_pass secret
    }
    virtual_ipaddress {
        192.168.1.99/24
    }
    track_script {
        pihole-FTL
    }
    notify "/usr/local/bin/throwawaydeadd pihole-FTL notify"
}
```

## Exit Codes

| Code | Meaning |
| :---: | :--- |
| `0` | Success (service running or action succeeded) |
| `1` | Failure (service not running or action failed) |
| `2` | Invalid usage (missing arguments or unknown options) |
| `3` | Configuration error (file not found, not readable, or invalid permissions) |

## Security Considerations

### Configuration File Security

- Config files are sourced directly into the shell environment
- Must contain only uppercase variable assignments (`VAR=val` format), comments, and blank lines
- Must not be world-writable or group-writable
- Must be owned by `root` or the current user
- The script validates file format before sourcing to prevent code injection

### Recommendations

- Store configuration files in `/etc/throwawaydeadd/` with restrictive permissions:
  ```bash
  sudo chown root:root /etc/throwawaydeadd/*.conf
  sudo chmod 600 /etc/throwawaydeadd/*.conf
  ```

- Only trusted users should have write access to config files

- Review configuration files before first use

## Logging

Logging output depends on the `VERBOSITY` setting and whether `logger` is available:

- **With logger available:** Messages are sent to syslog with tag `throwawaydeadd`
- **Without logger:** Messages are written to stderr

View logs with:
```bash
journalctl -t throwawaydeadd
# or
tail -f /var/log/syslog | grep throwawaydeadd
```

## Known Issues
- Caching may cause delayed detection of service state changes if `STATE_CACHE_TIMEOUT` is set too high
- Init system detection may fail on uncommon or custom init systems
- Configuration file validation is basic and may not catch all malformed files
- Actions are limited to basic service commands and may not cover all use cases
- Yes

## Troubleshooting

### Service Check Failing

1. Verify service name matches exactly:
   ```bash
   systemctl list-units --type=service | grep pihole-FTL
   ```

2. Check if service is actually running:
   ```bash
   systemctl status pihole-FTL
   ```

3. Enable debug logging:
   ```bash
   THROWAWAYDEADD_VERBOSITY=3 throwawaydeadd pihole-FTL check
   ```

### Notify Actions Not Executing

1. Verify config file exists and has correct actions:
   ```bash
   cat /etc/throwawaydeadd/pihole-FTL.conf
   ```

2. Check permissions:
   ```bash
   ls -l /etc/throwawaydeadd/pihole-FTL.conf
   ```

3. Test action manually:
   ```bash
   sudo systemctl restart pihole-FTL
   ```

### Cache Issues

Clear the state cache if you encounter stale state issues:

```bash
throwawaydeadd flush
```

## License

GNU General Public License v3.0 or later (GPL-3.0+)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

For the complete GPL-3.0+ license text, visit: https://www.gnu.org/licenses/gpl-3.0.html

## Version History

- **0.0.0 (Development)** - Initial release
  Wanna see something cool? Check out, this fukken' thing.
- **0.0.1-development**   - Obligatory ascii header
  Adds an obnoxious ascii text header in README and throwawaydeadd show_documentation(), as is required by nerd law.

## Support & Contributions

- **GitHub:** https://github.com/saint-lascivious/throwawaydeadd
- **Issues:** https://github.com/saint-lascivious/throwawaydeadd/issues
- **Contributions:** Pull requests welcome

## Author

**saint-lascivious** (Hayden Pearce)

- GitHub: https://github.com/saint-lascivious
- Copyright © 2026 Hayden Pearce. All rights reserved.