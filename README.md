collins-cli
===========

CLI scripts for interacting with Collins API

## Overview

Main entry point is the ```collins``` binary.

    Usage: collins <command> [options]
    Available commands:
      query, find:        Search for assets in Collins
      modify, set:        Add and remove attributes, change statuses, and log to assets
      log:                Display log messages on assets
      provision, action:  Provision, control power status, allocate IPs, update IPMI info

## Searching - collins-find

    Usage: collins-find [options] [hostnamepattern]
    Query options:
        -t, --tag TAG[,...]              Assets with tag[s] TAG
        -T, --type TYPE                  Only show assets with type TYPE
        -n, --nodeclass NODECLASS[,...]  Assets in nodeclass NODECLASS
        -p, --pool POOL[,...]            Assets in pool POOL
        -s, --size SIZE                  Number of assets to return (Default: 9999)
        -r, --role ROLE[,...]            Assets in primary role ROLE
        -R, --secondary-role ROLE[,...]  Assets in secondary role ROLE
        -i, --ip-address IP[,...]        Assets with IP address[es]
        -S STATUS[:STATE][,...],         Asset status (and optional state after :)
            --status
        -a attribute[:value[,...]],      Arbitrary attributes and values to match in query. : between key and value
            --attribute
    
    Table formatting:
        -H, --show-header                Show header fields in output
        -c, --columns ATTRIBUTES         Attributes to output as columns, comma separated (Default: tag,hostname,nodeclass,status,pool,primary_role,secondary_role)
        -x, --extra-columns ATTRIBUTES   Show these columns in addition to the default columns, comma separated
        -f, --field-separator SEPARATOR  Separator between columns in output (Default:      )
    
    Robot formatting:
        -l, --link                       Output link to assets found in web UI
        -j, --json                       Output results in JSON (NOTE: This probably wont be what you expected)
        -y, --yaml                       Output results in YAML
    
    Extra options:
            --expire SECONDS             Timeout in seconds (0 == forever)
        -C, --config CONFIG              Use specific Collins config yaml for Collins::Client
        -h, --help                       Help
    
    Examples:
        Query for devnodes in DEVEL pool that are VMs
          cf -n develnode -p DEVEL
        Query for asset 001234, and show its system_password
          cf -t 001234 -x system_password
        Query for all decommissioned VM assets
          cf -a is_vm:true -S decommissioned
        Query for hosts matching hostname '^web6-'
          cf ^web6-
        Query for all develnode6 nodes with a value for PUPPET_SERVER
          cf -n develnode6 -a puppet_server -H

## Logging - collins-log

    Usage: collins-log [options]
        -a, --all                        Show logs from ALL assets
        -n, --number LINES               Show the last LINES log entries. (Default: 20)
        -t, --tags TAGS                  Tags to work on, comma separated
        -f, --follow                     Poll for logs every 2 seconds
        -s, --severity SEVERITY[,...]    Log severities to return (Defaults to all). Use !SEVERITY to exclude one.
        -C, --config CONFIG              Use specific Collins config yaml for Collins::Client
        -h, --help                       Help
    
    Severities:
      EMERGENCY, ALERT, CRITICAL, ERROR, WARNING, NOTICE, INFORMATIONAL, DEBUG, NOTE
    
    Examples:
      Show last 20 logs for an asset
        collins-log -t 001234
      Show last 100 logs for an asset
        collins-log -t 001234 -n100
      Show last 10 logs for 2 assets that are ERROR severity
        collins-log -t 001234,001235 -n10 -sERROR
      Show last 10 logs all assets that are not note or informational severity
        collins-log -a -n10 -s'!informational,!note'
      Show last 10 logs for all web nodes that are provisioned having verification in the message
        cf -S provisioned -n webnode$ | collins-log -n10 -s debug | grep -i verification

## Modification - collins-modify

    Usage: collins-modify [options]
        -a attribute:value,              Set attribute=value. : between key and value. attribute will be uppercased.
            --set-attribute
        -d, --delete-attribute attribute Delete attribute.
        -S, --set-status status[:state]  Set status (and optionally state) to status:state. Requires --reason
        -r, --reason REASON              Reason for changing status/state.
        -l, --log MESSAGE                Create a log entry.
        -L, --level LEVEL                Set log level. Default level is NOTE.
        -t, --tags TAGS                  Tags to work on, comma separated
        -C, --config CONFIG              Use specific Collins config yaml for Collins::Client
        -h, --help                       Help
    
    Allowed values (uppercase or lowercase is accepted):
      Status (-S,--set-status):
        ALLOCATED, CANCELLED, DECOMMISSIONED, INCOMPLETE, MAINTENANCE, NEW, PROVISIONED, PROVISIONING, UNALLOCATED
      States (-S,--set-status):
        ALLOCATED ->
          CLAIMED, SPARE, RUNNING_UNMONITORED, UNMONITORED
        MAINTENANCE ->
          AWAITING_REVIEW, HARDWARE_PROBLEM, HW_TESTING, HARDWARE_UPGRADE, IPMI_PROBLEM, MAINT_NOOP, NETWORK_PROBLEM, RELOCATION, PROVISIONING_PROBLEM
        ANY ->
          RUNNING, STARTING, STOPPING, TERMINATED
      Log levels (-L,--level):
        EMERGENCY, ALERT, CRITICAL, ERROR, WARNING, NOTICE, INFORMATIONAL, DEBUG, NOTE
    
    Examples:
      Set an attribute on some hosts:
        collins-modify -t 001234,004567 -a my_attribute:true
      Delete an attribute on some hosts:
        collins-modify -t 001234,004567 -d my_attribute
      Delete and add attribute at same time:
        collins-modify -t 001234,004567 -a new_attr:test -d old_attr
      Set machine into maintenace noop:
        collins-modify -t 001234 -S maintenance:maint_noop -r "I do what I want"
      Set machine back to allocated:
        collins-modify -t 001234 -S allocated:running -r "Back to allocated"
      Set machine back to new without setting state:
        collins-modify -t 001234 -S new -r "Dunno why you would want this"
      Create a log entry:
        collins-modify -t 001234 -l'computers are broken and everything is horrible' -Lwarning
      Read from stdin:
        cf -n develnode | collins-modify -d my_attribute
        cf -n develnode -S allocated | collins-modify -a collectd_version:5.2.1-52
        echo -e "001234\n001235\n001236"| collins-modify -a test_attribute:'hello world'

## Actions - collins-action

    Usage: collins-action [options]
    Actions:
        -P, --provision                  Provision assets (see Provisioning flags).
        -S, --power-status               Show IPMI power status.
        -A, --power-action ACTION        Perform IPMI power ACTION on assets
    
    Provisioning Flags:
        -n, --nodeclass NODECLASS        Nodeclass to provision as. (Required)
        -p, --pool POOL                  Provision with pool POOL.
        -r, --role ROLE                  Provision with primary role ROLE.
        -R, --secondary-role ROLE        Provision with secondary role ROLE.
        -s, --suffix SUFFIX              Provision with suffix SUFFIX.
        -a, --activate                   Activate server on provision (useful with SL plugin) (Default: ignored)
        -b, --build-contact USER         Build contact. (Default: gabe)
    
    General:
        -t, --tags TAG[,...]             Tags to work on, comma separated
        -C, --config CONFIG              Use specific Collins config yaml for Collins::Client
        -h, --help                       Help
    
    Examples:
      Provision some machines:
        cf -Sunallocated -arack_position:716|collins-action -P -napiwebnode6 -RALL
      Show power status:
        cf ^dev6-gabe|collins-action -S
      Power cycle a bunch of machines:
        collins-action -t 001234,004567,007890 -A reboot

## TODO

I know the architecture is BRUTAL. This was all organically created; I need to refactor stuff out into libraries to facilitate code sharing between the utilities

* Implement IP allocation in collins-action
* Implement IPMI stuff in collins-action
* Share code between binaries
