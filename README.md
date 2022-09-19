# osmo-install

This is a guide to set up a lab for virtual Osmocom GSM network. Both CS core and RAN. The setup is based on Ansible

This guide is tested on a Ubuntu 20.04 running in virtualbox. The virtual machine is refered as "osmo server". One user, named "user" have to be added.

On osmo server:

Install required packages: `apt install python python-apt`

Give "user" access to sudo: `visudo`
>osmouser ALL=(ALL) NOPASSWD:ALL

On management client or the host: `apt install ansible`

`nano ~/.ssh/config`
>Host osmo-server<br>
>    Hostname 192.168.56.100<br>
>    Username user<br>

`ssh-copy-id osmo-server`


Test Ansible: 
`ansible osmo1 -i hosts -m ping`


    osmo1 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }


`ansible-playbook -i hosts osmo.yml`


# Management on osmo server

HLR<br>
`telnet 0  4258`

MSC<br>
`telnet 0 4254`

MGW<br>
`telnet 0 4243`

BSC<br>
`telnet 0 4242`

STP<br>
`telnet 0 4239`

BTS<br>
`telnet 0 4241`

MS<br>
`telnet 0 4247`


## Create users
Login to HLR
`telnet 0 4258`
```
ena
subscriber imsi 001010000000001 create
subscriber imsi 001010000000002 create
subscriber id 1 update aud2g comp128v1 ki 00000000000000000000000000000000
subscriber id 1 update msisdn 555501
subscriber id 2 update aud2g comp128v1 ki 00000000000000000000000000000000
subscriber id 2 update msisdn 555502
```

## Start phones

Start Virtphy: `virtphy`

Start Mobile: `mobile -c /etc/osmocom/mobile.cfg`

Login to Mobile and insert SIM-card
`telnet 0 4247`
```
ena
conf t
ms 1
shut
sim test
no shut
```

