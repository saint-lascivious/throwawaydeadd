# [throwawaydeadd](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/throwawaydeadd "Link to throwawaydeadd script")

**Author:** saint-lascivious (Hayden Pearce) ©2023

**License:** GNU General Public License Version 3

## Overview

A simple and highly flexible `keepalived` check and/or notify script particularly useful for managing failover scenarios in a high-availability environment.

## Features

- **Light Weight:** Principle of least operation. Get in, do work, get out.

- **Simple:** Monitor (and optionally take action on) a service as easily as renaming a file or creating a symbolic link.

- **Check & Notify:** `throwawaydeadd` operates as a `keepalived` check or notify script depending on the extension used for the script or symbolic link.

- **Actions:** Perform one of many `systemctl` actions on the service or host based on the current state of the service being monitored.

- **Low Dependency:** Simple bash shell construction requires only `systemctl` and some basic built-ins.

- **Configurable:** Optional external configuration can be provided from a file with the same name as the check and or notify script/symbolic link, located within the same working directory as the script or symbolic link.

## [Check/Notify Script](https://github.com/saint-lascivious/throwawaydeadd/tree/master/script "Link to throwawaydeadd keepalived check/notify script folder")

### Install throwawaydeadd

All commands given are assumed to be run as `root` or equivalent (with `sudo`).

For ease of use with a real world sample all examples going forward will assume that the service to be monitored is Pi-Hole's service, named `pihole-FTL`.

The basename and file extension will dictate the service to monitor and the function to perform respectively. The service name MUST be an exact match, and the extension MUST be one of `check`, `CHECK`, `notify` or `NOTIFY`. It is not necessary (nor is it forbidden) to provide the `.service` section of the service.

1. Create the directory `throwawaydeadd` and its symbolic links will live:

   `mkdir /etc/throwawaydeadd`

2. Navigate to that directory:

   `cd /etc/throwawaydeadd`

3. Download the `throwawaydeadd` script using `wget`:

   `wget https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/throwawaydeadd`

4. Review or modify the script if desired, and ensure it is executable:

   `chmod +x throwawaydeadd`

5. Create link(s):

   `ln -s /etc/throwawaydeadd/throwawaydeadd /etc/throwawaydeadd/pihole-FTL.check`

   `ln -s /etc/throwawaydeadd/throwawaydeadd /etc/throwawaydeadd/pihole-FTL.notify`

### Configure throwawaydeadd

To indicate that they should function as a MASTER server rather than a BACKUP server at a system/host rather than script/symbolic link level, MASTER server(s) MAY create a dotfile trigger in the script directory.

Example:

```
touch /etc/throwawaydeadd/.is_master_server
```

External configuration may be supplied via a configuration file with the same name as the service being monitored, located within the script directory.

- **IS_MASTER:** May be set to `true` on a case by case basis via this KEY=VALUE pair, or system wide via an `.is_master_server` dotfile trigger located within the script directory.

   The default value is `false`.

- **LOG_DIR:** The directory in which `throwawaydeadd` log files are kept.

   The default value is `/var/log/throwawaydeadd`.

- **LOG_VERBOSITY:** A numeric value from `0` (zero) to `4` (four), representing:

   | Verbosity Level | Log Type |
   | :---: | :---: |
   | `0` | No logging. |
   | `1` | Errors. |
   | `2` | Warnings. |
   | `3` | Info messages. |
   | `4` | Debug messages |

   The default value is `0` (zero).

- **EXPECTED_SERVICE_STATE:** The expected service state for check operations.

   If a service is found to be in a state other than the expected state then the check will fail.

   The default value is `active`.

- **[$SERVER]_IS_[$ROLE]_[$SERVICE-STATE]:** The action to perform, depending on the current service state, when the VRRP state changes from MASTER to BACKUP or vice versa.

   The format for this variable is `MASTER` or `BACKUP` `_IS_` `MASTER` or `BACKUP` `_ACTIVE`, `_INACTIVE`, `_FAILED`, `_DISABLED` or `_MASKED`.

   See the example configuration file below for a full list of default values.

   May be ONE (1) of the following actions to be performed on services in the form of `systemctl $ACTION $SERVICE`:

   | Action | Description |
   | :---: | :---: |
   | `restart` | Stop and then start one or more units specified on the command line. If the units are not running yet, they will be started. |
   | `start` | Start (activate) one or more units specified on the command line. |
   | `stop` | Stop (deactivate) one or more units specified on the command line. |
   | `enable` | Enable one or more units or unit instances. This will create a set of symlinks, as encoded in the [Install] sections of the indicated unit files. After the symlinks have been created, the system manager configuration is reloaded (in a way equivalent to daemon-reload), in order to ensure the changes are taken into account immediately. Note that this does not have the effect of also starting any of the units being enabled. If this is desired, combine this command with the --now switch, or invoke start with appropriate arguments later. Note that in case of unit instance enablement (i.e. enablement of units of the form foo@bar.service), symlinks named the same as instances are created in the unit configuration directory, however they point to the single template unit file they are instantiated from. |
   | `disable` | Disables one or more units. This removes all symlinks to the unit files backing the specified units from the unit configuration directory, and hence undoes any changes made by enable or link. Note that this removes all symlinks to matching unit files, including manually created symlinks, and not just those actually created by enable or link. Note that while disable undoes the effect of enable, the two commands are otherwise not symmetric, as disable may remove more symlinks than a prior enable invocation of the same unit created. |
   | `mask` | Mask one or more units, as specified on the command line. This will link these unit files to /dev/null, making it impossible to start them. This is a stronger version of disable, since it prohibits all kinds of activation of the unit, including enablement and manual activation. Use this option with care. This honors the --runtime option to only mask temporarily until the next reboot of the system. The --now option may be used to ensure that the units are also stopped. This command expects valid unit names only, it does not accept unit file paths. |
   | `unmask` | Unmask one or more unit files, as specified on the command line. This will undo the effect of mask. This command expects valid unit names only, it does not accept unit file paths. |
   | `reenable` | Reenable one or more units, as specified on the command line. This is a combination of disable and enable and is useful to reset the symlinks a unit file is enabled with to the defaults configured in its [Install] section. This command expects a unit name only, it does not accept paths to unit files. |
   | `reload` | Asks all units listed on the command line to reload their configuration. Note that this will reload the service-specific configuration, not the unit configuration file of systemd. If you want systemd to reload the configuration file of a unit, use the daemon-reload command. In other words: for the example case of Apache, this will reload Apache's httpd.conf in the web server, not the apache.service systemd unit file. |
   | `reload-or-restart` | Reload one or more units if they support it. If not, stop and then start them instead. If the units are not running yet, they will be started. |
   | `reset-failed` | Reset the "failed" state of the specified units, or if no unit name is passed, reset the state of all units. When a unit fails in some way (i.e. process exiting with non-zero error code, terminating abnormally or timing out), it will automatically enter the "failed" state and its exit code and status is recorded for introspection by the administrator until the service is stopped/re-started or reset with this command. |
   | `try-restart` | Stop and then start one or more units specified on the command line if the units are running. This does nothing if units are not running. |
   | `try-reload-or-restart` | Reload one or more units if they support it. If not, stop and then start them instead. This does nothing if the units are not running. |

   ONE (1) of the following actions performed in the form of `systemctl $ACTION`:

   | Action | Description |
   | :---: | :---: |
   | `reboot` | Shut down and reboot the system. |
   | `halt` | Shut down and halt the system. This is mostly equivalent to systemctl start halt.target --job-mode=replace-irreversibly --no-block, but also prints a wall message to all users. This command is asynchronous; it will return after the halt operation is enqueued, without waiting for it to complete. Note that this operation will simply halt the OS kernel after shutting down, leaving the hardware powered on. Use systemctl poweroff for powering off the system (see below). |
   | `poweroff` | Shut down and power-off the system. This is mostly equivalent to systemctl start poweroff.target --job-mode=replace-irreversibly --no-block, but also prints a wall message to all users. This command is asynchronous; it will return after the power-off operation is enqueued, without waiting for it to complete. |
   | `daemon-reexec` | Reexecute the systemd manager. This will serialize the manager state, reexecute the process and deserialize the state again. This command is of little use except for debugging and package upgrades. Sometimes, it might be helpful as a heavy-weight daemon-reload. While the daemon is being reexecuted, all sockets systemd listening on behalf of user configuration will stay accessible. |
   | `daemon-reload` | Reload the systemd manager configuration. This will rerun all generators (see systemd.generator(7)), reload all unit files, and recreate the entire dependency tree. While the daemon is being reloaded, all sockets systemd listens on behalf of user configuration will stay accessible. |
   | `sleep` | Put the system to sleep, through suspend, hibernate, hybrid-sleep, or suspend-then-hibernate. The sleep operation to use is automatically selected by systemd-logind.service(8). By default, suspend-then-hibernate is used, and falls back to suspend and then hibernate if not supported. Refer to SleepOperation= setting in logind.conf(5) for more details. This command is asynchronous, and will return after the sleep operation is successfully enqueued. It will not wait for the sleep/resume cycle to complete. |
   | `suspend` | Suspend the system. This will trigger activation of the special target unit suspend.target. This command is asynchronous, and will return after the suspend operation is successfully enqueued. It will not wait for the suspend/resume cycle to complete. |
   | `hibernate` | Hibernate the system. This will trigger activation of the special target unit hibernate.target. This command is asynchronous, and will return after the hibernation operation is successfully enqueued. It will not wait for the hibernate/thaw cycle to complete. |
   | `suspend-then-hibernate` | Suspend the system and hibernate it after the delay specified in systemd-sleep.conf. This will trigger activation of the special target unit suspend-then-hibernate.target. This command is asynchronous, and will return after the hybrid sleep operation is successfully enqueued. It will not wait for the sleep/wake-up or hibernate/thaw cycle to complete. |
   | `hybrid-sleep` | Hibernate and suspend the system. This will trigger activation of the special target unit hybrid-sleep.target. This command is asynchronous, and will return after the hybrid sleep operation is successfully enqueued. It will not wait for the sleep/wake-up cycle to complete. |

   Or ONE (1) of the following options:

   | Action | Description |
   | :---: | :---: |
   | `none` | No action is performed. |

   Note that not all options may be possible on all host servers.

   Any other string is rejected and will result in an abort with `exit 1`.

Example [/etc/throwawaydeadd/pihole-FTL.conf](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/pihole-FTL.conf "Link to example pihole-FTL.conf") shown here with default values:

```
IS_MASTER="false"

LOG_DIR=/var/log/throwawaydeadd
LOG_VERBOSITY=0

EXPECTED_SERVICE_STATE="active"

MASTER_IS_MASTER_ACTIVE="none"
MASTER_IS_MASTER_INACTIVE="start"
MASTER_IS_MASTER_FAILED="restart"
MASTER_IS_MASTER_DISABLED="enable"
MASTER_IS_MASTER_MASKED="none"
BACKUP_IS_MASTER_ACTIVE="none"
BACKUP_IS_MASTER_INACTIVE="start"
BACKUP_IS_MASTER_FAILED="restart"
BACKUP_IS_MASTER_DISABLED="enable"
BACKUP_IS_MASTER_MASKED="none"
MASTER_IS_BACKUP_ACTIVE="none"
MASTER_IS_BACKUP_INACTIVE="start"
MASTER_IS_BACKUP_FAILED="restart"
MASTER_IS_BACKUP_DISABLED="enable"
MASTER_IS_BACKUP_MASKED="none"
BACKUP_IS_BACKUP_ACTIVE="none"
BACKUP_IS_BACKUP_INACTIVE="start"
BACKUP_IS_BACKUP_FAILED="restart"
BACKUP_IS_BACKUP_DISABLED="enable"
BACKUP_IS_BACKUP_MASKED="none"
```

## [VRRP Config](https://github.com/saint-lascivious/throwawaydeadd/tree/master/config "Link to keepalived VRRP config folder")

MASTER to BACKUP failover, with fast fall and slow rise.

The following examples assume that the service to be monitored is `pihole-FTL`, that there is a MASTER running on `192.168.1.10` and a BACKUP running on `192.168.1.20`, and that the virtual router IP address is `192.168.1.99` on the `eth0` interface.

The example service is checked with an interval of `5` seconds, and a timeout of `4` seconds. The advertisement interval is `1` second. The priority of an instance will fall after `2` successive failures (`10` seconds), and only rise again after `12` successive successes (`60` seconds).

The MASTER has a starting priority of `150`, and the BACKUP has a starting priority of `145`. The weight value is `-10`.

The Virtual Router ID (`99`) reflecting the final digits of the Virtual IP Address (`192.168.1.99`) is strictly for convenience/memory association.

Once again, any/all instances of `pihole-FTL` used in the examples should be replaced with the exact (case sensitive) service name to whose state you wish to attach your virtual IP address.

### [MASTER](https://github.com/saint-lascivious/throwawaydeadd/tree/master/config/MASTER "Link to MASTER keepalived VRRP config folder")

Example [keepalived.conf](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/config/MASTER/keepalived.conf "Link to example MASTER keepalived.conf") for MASTER server:

```
global_defs {
    enable_script_security
    max_auto_priority 99
    router_id MASTER
    script_user root
}

vrrp_script check_pihole-FTL {
    script "/etc/throwawaydeadd/pihole-FTL.check"
    interval 5
    weight -10
    timeout 4
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
        auth_pass default
    }
    virtual_ipaddress {
        192.168.1.99/24
    }
    track_script {
        check_pihole-FTL
    }
    notify /etc/throwawaydeadd/pihole-FTL.notify
}
```

### [BACKUP](https://github.com/saint-lascivious/throwawaydeadd/tree/master/config/MASTER "Link to BACKUP keepalived VRRP config folder")

Example [keepalived.conf](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/config/BACKUP/keepalived.conf "Link to example BACKUP keepalived.conf") for BACKUP server:

```
global_defs {
    enable_script_security
    max_auto_priority 99
    router_id BACKUP
    script_user root
}

vrrp_script check_pihole-FTL {
    script "/etc/throwawaydeadd/pihole-FTL.check"
    interval 5
    weight -10
    timeout 4
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
        auth_pass default
    }
    virtual_ipaddress {
        192.168.1.99/24
    }
    track_script {
        check_pihole-FTL
    }
    notify /etc/throwawaydeadd/pihole-FTL.notify
}
```
