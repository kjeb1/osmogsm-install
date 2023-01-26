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

Alternative to use ssh is to install ansible on the osmo server and run the playbook from there. Then the hosts file should look like this:
```
[osmo]
osmo-server ansible_host=localhost ansible_connection=local
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

## Start virtual phone

Start Virtphy (the virtual RAN): 
```
virtphy
```

Start Mobile Station: 
```
mobile -c /etc/osmocom/mobile.cfg
```


# Management and testing on osmo-server

MS<br>
`telnet 0 4247`
```
OsmocomBB> show ms 1
MS '1' is up, service is limited
  IMEI: 000000000000001
     IMEISV: 0000000000000010
     IMEI generation: fixed
  automatic network selection state: A0 null
  cell selection state: C0 null
  radio resource layer state: idle
  mobility management layer state: MM idle, PLMN search

OsmocomBB> show cell 1
ARFCN  |MCC    |MNC    |LAC    |cell ID|forb.LA|prio   |min-db |max-pwr|rx-lev
-------+-------+-------+-------+-------+-------+-------+-------+-------+-------


```

HLR<br>
`telnet 0  4258`
```
OsmoHLR> subscriber id 1 show
    ID: 1
    IMSI: 001010000000001
    MSISDN: none
```

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
```
OsmoBTS>show bts 0
```

