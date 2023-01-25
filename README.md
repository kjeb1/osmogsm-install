# osmo-install

This is a guide to set up a lab for virtual Osmocom GSM network. Both CS core and RAN. The setup is based on Ansible

This guide is tested on a Ubuntu 20.04 running in Virtual Box. The virtual machine is refered as "osmo server". One user, named "osmo" have to be added. I recomand to use "Bridged network" for the VM.

On osmo server:

Install Ansible required packages: 
```
apt install python python-apt
```

Give "osmo" user access to sudo: 
```
visudo /etc/sudoers.d/osmo
```
>osmo ALL=(ALL) NOPASSWD:ALL

On management client or the host:
```
apt install ansible
```

`~/.ssh/config`
>Host osmo-server<br>
>    Hostname <VM IP><br>
>    User osmo<br>

```
ssh-copy-id osmo-server
```

Download git repo:
```
git clone git@github.com:kjeb1/osmo-install.git
```
    
    
Test Ansible: 
```
cd osmo-install
ansible osmo-server -i hosts -m ping
```

    osmo-server | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }


Run the playbook for setup the GSM lab
```
ansible-playbook -i hosts osmo.yml
```


# Management on osmo-server

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
subscriber id 1 show
subscriber id 2 show
```
## Start phones

Start Virtphy (virtual RAN): 
```
virtphy
```

Start Mobile Station: 
```
mobile -c /etc/osmocom/mobile.cfg
```

Login to Mobile Station and insert SIM-card
`telnet 0 4247`
```
ena
conf t
ms 1
shut
sim test
no shut
```

BTS
telnet 0 4241
show bts 0

Phone
telnet 0 4247
show ms
show cell 1
