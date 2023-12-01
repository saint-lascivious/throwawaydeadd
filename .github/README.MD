# [throwawaydeadd](https://raw.githubusercontent.com/saint-lascivious/throwawaydeadd/master/script/throwawaydeadd "Link to throwawaydeadd script")
saint-lascivious (Hayden Pearce) ©2023

A simple generic yet flexible `keepalived` check and/or notify script.

# [Script](https://github.com/saint-lascivious/throwawaydeadd/tree/master/script "Link to script folder")

## Install script
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

All examples going forward will assume that the service to be monitored is named `Example-Service`.

Example:
```
ln -s /etc/scripts/throwawaydeadd /etc/scripts/Example-Service.check
ln -s /etc/scripts/throwawaydeadd /etc/scripts/Example-Service.notify
```

To indicate that they should function as a MASTER server rather than a BACKUP server, MASTER server(s) MAY create a trigger file.

Example:
```
touch /etc/scripts/.is_master_server
```

## Configure script
External configuration may be supplied via a configuration file in the `/etc/scripts/` directory, with the same name as the service being monitored.

Example `/etc/scripts/Example-Service.conf` shown here with default values:
```
IS_MASTER="false"

MASTER_IS_BACKUP_ACTIVE="none"
MASTER_IS_BACKUP_INACTIVE="start"
MASTER_IS_BACKUP_FAILED="restart"

BACKUP_IS_BACKUP_ACTIVE="none"
BACKUP_IS_BACKUP_INACTIVE="start"
BACKUP_IS_BACKUP_FAILED="restart"

MASTER_IS_MASTER_ACTIVE="none"
MASTER_IS_MASTER_INACTIVE="start"
MASTER_IS_MASTER_FAILED="restart"

BACKUP_IS_MASTER_ACTIVE="none"
BACKUP_IS_MASTER_INACTIVE="start"
BACKUP_IS_MASTER_FAILED="restart"
```

Possible options for `SERVER_IS_ROLE_STATE` variables may be ONE (1) of the following.

Options performed on services in the form of `systemctl $option Example-Service`:
`disable`, `enable`, `mask`, `reenable`, `reload`, `reload-or-restart`, `reset-failed`, `restart`, `start`, `stop` or `unmask`.

Options to restart or shutdown the host server in the form of `systemctl $option`:
`reboot` or `poweroff` respectively.

Null options:
`none` or blank/unset, do nothing.

*:
Any other string is rejected and will result in an abort with `exit 1`.

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
    router_id MASTER
    script_user root
    enable_script_security
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
    router_id BACKUP
    script_user root
    enable_script_security
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
