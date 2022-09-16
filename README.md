# osmo-install

This is a guide to set up a lab for virtual Osmocom GSM. Both CS core and RAN. The setup is based on Ansible


On osmo server:
    apt install python python-apt


    visudo

>osmouser ALL=(ALL) NOPASSWD:ALL


On management client:

    apt install ansible


~/.ssh/config
>Host osmo1<br>
>    Hostname 192.168.56.100<br>
>    Username osmouser<br>

    ssh-copy-id osmo1


Test
    ansible osmo1 -i hosts -m ping


    osmo1 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }


    ansible-playbook -i hosts osmo.yml


# Management on osmo server

HLR
    telnet 0  4258

MSC
    telnet 0 4254

MGW
    telnet 0 4243

BSC
    telnet 0 4242

STP
    telnet 0 4239

BTS
    telnet 0 4241

MS
    telnet 0 4247


## Create users
Login to HLR
telnet 0 4258
    ena
    subscriber imsi 001010000000001
    subscriber imsi 001010000000002
    subscriber id 1 update aud2g comp128v1 ki 00000000000000000000000000000000
    subscriber id 1 update msisdn 555501
    subscriber id 2 update aud2g comp128v1 ki 00000000000000000000000000000000
    subscriber id 2 update msisdn 555502


## Start phones

Virtphy
virtphy has to be started before turning on the MS
    virtphy

Mobile
    mobile -c /etc/osmocom/ms1.cfg

    mobile -c /etc/osmocom/ms2.cfg