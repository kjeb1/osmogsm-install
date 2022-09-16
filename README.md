# osmo-install

On osmo server:
    apt install python python-apt

    visudo
    user ALL=(ALL) NOPASSWD:ALL


On management client:

    apt install ansible

~/.ssh/config
>Host osmo1
>    Hostname 192.168.56.100
>    Username osmouser

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
    telnet 0 4255

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