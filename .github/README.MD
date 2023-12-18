# [throwawaydeadd](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/throwawaydeadd "Link to throwawaydeadd script")
saint-lascivious (Hayden Pearce) ©2023

A simple generic yet flexible `keepalived` check and/or notify script.

### Features
- Simple filename based operation

Monitor (and optionally take action on) a service as easily as renaming a file or creating a symbolic link.

`throwawaydeadd` operates as either a check or notify script depending on the extension used for the script or symbolic link.

- Low dependencies

Simple bash shell construction.

Requires only some basic `bash` builtins, `systemctl` and `date` commands.

- Configurable

External configuration can be provided from a file with the same name as the check and or notify script/symbolic link, located within the same working directory as the script or symbolic link.

- State and lock file enhanced logic

Reduced potential for state flapping or "ping-pong".

Prevents multiple simultaneous executions of the same script or "timeout runaway" which can be typical in high run frequency scenarios.

- Action packed

Perform one of many `systemctl` actions depending on the current state of the service being monitored, up to/including (re)starting or stopping that service or even the host server itself.

- Get hooked

Optionally execute one or more scripts of your choice from specified directories both prior to and/or after performing a `systemctl` action.

# [Script](https://github.com/saint-lascivious/throwawaydeadd/tree/master/script "Link to script folder")

## Install throwawaydeadd
All commands given are assumed to be run as `root` or equivalent (with `sudo`).

Create the directory `throwawaydeadd` and its symbolic links will live:
```
mkdir /etc/scripts
```

Navigate to that directory:
```
cd /etc/scripts
```

Download the `throwawaydeadd` script using `wget`:
```
wget https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/throwawaydeadd
```

Review or modify the script if desired, and ensure it is executable:
```
chmod +x throwawaydeadd
```

The basename and file extension will dictate the service to monitor and the function to perform respectively.

The service name MUST be an exact match, and the extension MUST be one of `check`, `CHECK`, `notify` or `NOTIFY`.

It is not necessary (nor is it forbidden) to provide the `.service` section of the service.

All examples going forward will assume that the service to be monitored is named `Example-Service`.

Example:
```
ln -s /etc/scripts/throwawaydeadd /etc/scripts/Example-Service.check
ln -s /etc/scripts/throwawaydeadd /etc/scripts/Example-Service.notify
```

To indicate that they should function as a MASTER server rather than a BACKUP server at a system/host rather than script/symbolic link level, MASTER server(s) MAY create a dotfile trigger in the same directory as the script or symbolic link.

Example:
```
touch /etc/scripts/.is_master_server
```

If desired, create and populate the directory structure from which pre, post and failed service action hook scripts will be run from.

Those paths are `./hooks/pre_action`, `.hooks/post_action` and `.hooks/failed_action` for pre, post and failed action hooks respectively. If found to be executable, any scripts in those directories will be executed in alphanumeric order.

Please note that post_action hooks will only be triggered if that action succeeds.

Example:
```
mkdir -p ./hooks/pre_action
mkdir -p ./hooks/post_action
mkdir -p ./hooks/failed_action
```

Please note that this step is optional and only required if you wish to run one or more custom scripts prior to and/or after performing a service action other than null/"none".

## Configure throwawaydeadd
External configuration may be supplied via a configuration file in the `/etc/scripts/` directory, with the same name as the service being monitored.

Example [/etc/scripts/Example-Service.conf](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/Example-Service.conf "Link to example Example-Service.conf") shown here with default values:
```
IS_MASTER="false"

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

PRE_ACTION_HOOK="pre_hook"
POST_ACTION_HOOK="post_hook"

MAX_STATE_AGE="4"
STATE_DIR="/var/log"
```

`IS_MASTER` may be set to `true` on a case by case basis via this KEY=VALUE pair or system wide via an `.is_master_server` dotfile trigger located within the same directory as the script or symbolic link, the default value is `false`.

`SERVER_IS_ROLE_SERVICE-STATE` variables may be ONE (1) of the following.

Options performed on services in the form of `systemctl $option Example-Service`:
`restart`, `start`, `stop`, `enable`, `disable`, `mask`, `unmask`, `reenable`, `reload`, `reload-or-restart`, `reset-failed`, `try-restart` and `try-reload-or-restart`.

Options performed in the form of `systemctl $option`:
`reboot`, `poweroff`, `halt`, `daemon-reexec`, `daemon-reload`, `suspend`, `hibernate`, `suspend-then-hibernate` and `hybrid-sleep`.

Note that not all options may be possible on all host servers.

Null options:
`none` or blank/unset, do nothing and exit with a success status.

Any other string is rejected and will result in an abort with `exit 1`.

`MAX_STATE_AGE` is the time in seconds a service state derived from a state file should be considered fresh, the default value is `4` (FOUR) seconds.

`STATE_DIR` is the directory in which `.state` and `.lock` files should be kept, the default value is `/var/log`.

# [Config](https://github.com/saint-lascivious/throwawaydeadd/tree/master/config "Link to config folder")
MASTER to BACKUP failover, with fast fall and slow rise.

The following examples assume that the service to be monitored is `Example-Service`, that there is a MASTER running on `192.168.1.10` and a BACKUP running on `192.168.1.20`, and that the virtual router IP address is `192.168.1.99` on the `eth0` interface.

The example service is checked with an interval of `5` seconds, and a timeout of `4` seconds. The advertisement interval is `1` second. The priority of an instance will fall after `2` successive failures (`10` seconds), and only rise again after `12` successive successes (`60` seconds).

The MASTER has a starting priority of `150`, and the BACKUP has a starting priority of `145`. The weight value is `-10`.

The default shared password is `default`.

The Virtual Router ID (`99`) reflecting the final digits of the Virtual IP Address (`192.168.1.99`) is strictly for convenience/memory association.

Once again, any/all instances of `Example-Service`/`EXAMPLE_SERVICE` used in the examples should be replaced with the exact (case sensitive) service name to whose state you wish to attach your virtual IP address.

## [MASTER](https://github.com/saint-lascivious/throwawaydeadd/tree/master/config/MASTER "Link to MASTER config folder")
Example [keepalived.conf](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/config/MASTER/keepalived.conf "Link to example MASTER keepalived.conf") for MASTER server:
```
global_defs {
    enable_script_security
    max_auto_priority 99
    router_id MASTER
    script_user root
}

vrrp_script check_Example-Service {
    script "/etc/scripts/Example-Service.check"
    interval 5
    weight -10
    timeout 4
    rise 12
    fall 2
}

vrrp_instance EXAMPLE_SERVICE {
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
        check_Example-Service
    }
    notify /etc/scripts/Example-Service.notify
}
```

## [BACKUP](https://github.com/saint-lascivious/throwawaydeadd/tree/master/config/MASTER "Link to BACKUP config folder")
Example [keepalived.conf](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/config/BACKUP/keepalived.conf "Link to example BACKUP keepalived.conf") for BACKUP server:
```
global_defs {
    enable_script_security
    max_auto_priority 99
    router_id BACKUP
    script_user root
}

vrrp_script check_Example-Service {
    script "/etc/scripts/Example-Service.check"
    interval 5
    weight -10
    timeout 4
    rise 12
    fall 2
}

vrrp_instance EXAMPLE_SERVICE {
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
        check_Example-Service
    }
    notify /etc/scripts/Example-Service.notify
}
```
