# throwawaydeadd
saint-lascivious (Hayden Pearce) ©2023

A simple generic yet flexible `keepalived` check and/or notify script.

# Script
The basename and file extension will dictate the service to monitor and the function to perform respectively.

The service name MUST be an exact match, and the extension MUST be one of `check`, `CHECK`, `notify` or `NOTIFY`.

Example:
```
ln -s /etc/scripts/throwawaydeadd /etc/scripts/Example-Service.check
ln -s /etc/scripts/throwawaydeadd /etc/scripts/Example-Service.notify
```

To indicate that they should function primarily as a MASTER server rather than a BACKUP server, MASTER server(s) SHOULD create a trigger file.

Example:
```
touch /etc/scripts/.is_master_server
```

## Configure
```
MASTER_FLAG_FILE="/etc/scripts/.is_master_server"  # MASTER server trigger file.
MAX_STATE_AGE="4"  # The time in seconds before saved state considered stale.
STATE_DIR="/var/log/"  # Directory in which *.lock and *.state files are kept.
```

# Config
MASTER to BACKUP failover, with fast fall and slow rise.

## MASTER
Example `keepalived.conf` for master server.
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
        auth_pass sdWejwse
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

## BACKUP
Example `keepalived.conf` for backup server.
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
        auth_pass sdWejwse
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
