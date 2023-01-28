# Install Osmocom GSM lab with Ansible

This is a Ansible playbook and a guide to set up a lab for virtual [Osmocom](https://osmocom.org/) GSM network. Both CS core and RAN.

The [Osmocom](https://osmocom.org/) project is an umbrella project regarding Open source mobile communications. This includes software and tools implementing a variety of mobile communication standards, including GSM, DECT, TETRA and others.

The guide is tested on a Ubuntu 20.04 running in Virtual Box. The virtual machine is refered as "osmo-server". One user, named "osmo" have to be added. I recomand to use "Bridged network" for the VM (for both ssh access from host and internet access for the VM).

On osmo-server:

Install packages required for Ansible: 
```
apt install python python-apt
```

Give osmo user access to sudo: 
```
visudo /etc/sudoers.d/osmo
```
>osmo ALL=(ALL) NOPASSWD:ALL

On the host:
```
apt install ansible
```

`~/.ssh/config`
>Host osmo-server<br>
>    Hostname <VM IP><br>
>    User osmo<br>

Copy your public SSH key to the VM
```
ssh-copy-id osmo-server
```

Download git repo:
```
git clone git@github.com:kjeb1/osmogsm-install.git
```

An alternative to use SSH is to install Ansible on the osmo-server and run the playbook from there. Then the hosts file should look like this:
```
[osmo]
osmo-server ansible_host=localhost ansible_connection=local
```

Test Ansible: 
```
cd osmogsm-install
ansible osmo-server -i hosts -m ping
```
    osmo-server | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }


Run the playbook for setup and config of the GSM lab (first dryrun with --check)
```
ansible-playbook -i hosts osmo.yml --check
```

```
ansible-playbook -i hosts osmo.yml

PLAY [osmo] ********************************************************************

TASK [Install packages for building] *******************************************
ok: [osmo-server]

TASK [Clone newest libosmocore from gitea.osmocom.org] *************************
changed: [osmo-server]

TASK [Clone OsmocomBB from gitea.osmocom.org] **********************************
changed: [osmo-server]

TASK [Clone Osmo-bts from Github] **********************************************
ok: [osmo-server]

TASK [Clone libosmo-gprs from Github] ******************************************
changed: [osmo-server]

TASK [Build libosmocore] *******************************************************
ok: [osmo-server] => (item=autoreconf -i)
ok: [osmo-server] => (item=./configure)
ok: [osmo-server] => (item=make)
ok: [osmo-server] => (item=make install)
ok: [osmo-server] => (item=ldconfig -i)

TASK [Build libosmo-gprs] ******************************************************
ok: [osmo-server] => (item=autoreconf -i)
ok: [osmo-server] => (item=./configure)
ok: [osmo-server] => (item=make)
ok: [osmo-server] => (item=make install)

TASK [Build mobile from osmocom-bb] ********************************************
ok: [osmo-server] => (item=autoreconf -i)
ok: [osmo-server] => (item=./configure)
ok: [osmo-server] => (item=make)
ok: [osmo-server] => (item=make install)

TASK [Build virtphy from osmocom-bb] *******************************************
ok: [osmo-server] => (item=autoreconf -i)
ok: [osmo-server] => (item=./configure)
ok: [osmo-server] => (item=make)
ok: [osmo-server] => (item=make install)

TASK [Install osmo packages] ***************************************************
ok: [osmo-server]

TASK [Start HLR] ***************************************************************
changed: [osmo-server]

TASK [Start STP] ***************************************************************
changed: [osmo-server]

TASK [Start MGW] ***************************************************************
changed: [osmo-server]

TASK [Start MSC] ***************************************************************
changed: [osmo-server]

TASK [Start BSC] ***************************************************************
changed: [osmo-server]

TASK [Copy service file for virtual BTS] ***************************************
ok: [osmo-server]

TASK [Copy configfile for virtual BTS] *****************************************
changed: [osmo-server]

TASK [Change basic config for virtual BTS] *************************************
ok: [osmo-server]

TASK [Start BTS] ***************************************************************
changed: [osmo-server]

TASK [Copy configfile for virtual MSs] *****************************************
changed: [osmo-server]

TASK [Configure IMEI for mobile] ***********************************************
ok: [osmo-server]

TASK [Configure IMSI for mobile] ***********************************************
ok: [osmo-server]

TASK [Configure SIM for mobile] ************************************************
ok: [osmo-server]

TASK [Create directory for sms inbox] ******************************************
ok: [osmo-server]

PLAY RECAP *********************************************************************
osmo-server : ok=24   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

## Create users

Login to HLR<br>
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

## Start virtual RAN and phone

### Start Virtphy (the virtual RAN)

```
root@osmo-server:~# virtphy 
Fri Jan 27 14:30:29 2023 DVIRPHY virtphy.c:244 Virtual physical layer starting up...
Fri Jan 27 14:30:29 2023 DVIRPHY virtphy.c:253 Virtual physical layer ready, waiting for l23 app(s) on /tmp/osmocom_l2
Fri Jan 27 14:31:10 2023 DMAIN virt_l1_model.c:41 MS 0000: allocated
Fri Jan 27 14:31:10 2023 DL1C l1ctl_sock.c:138 Accepted client (fd=6) from server (fd=5)
Fri Jan 27 14:31:10 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 1)
Fri Jan 27 14:31:10 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 1)
Fri Jan 27 14:31:10 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 1)
Fri Jan 27 14:31:10 2023 DL1C virt_prim_fbsb.c:56 MS 0000: Rx L1CTL_FBSB_REQ (arfcn=868, flags=0x7)
Fri Jan 27 14:31:10 2023 DL1C virt_prim_fbsb.c:125 MS 0000: Tx L1CTL_FBSB_CONF (res: 0)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:463 MS 0000: Rx L1CTL_CCCH_MODE_REQ (mode=2)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:724 MS 0000: Tx L1CTL_CCCH_MODE_CONF (mode: 2)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 1)
Fri Jan 27 14:31:11 2023 DL1C virt_prim_fbsb.c:56 MS 0000: Rx L1CTL_FBSB_REQ (arfcn=868, flags=0x7)
Fri Jan 27 14:31:11 2023 DL1C virt_prim_fbsb.c:125 MS 0000: Tx L1CTL_FBSB_CONF (res: 0)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:404 MS 0000: Rx L1CTL_PARAM_REQ (ta=0, tx_power=7)
Fri Jan 27 14:31:11 2023 DL1C virt_prim_rach.c:81 MS 0000: Rx L1CTL_RACH_REQ (ra=0x00, offset=2 combined=1)
Fri Jan 27 14:31:11 2023 DL1C virt_prim_rach.c:122 MS 0000: Tx L1CTL_RACH_CONF (fn: 13582, arfcn: 868)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:404 MS 0000: Rx L1CTL_PARAM_REQ (ta=0, tx_power=7)
Fri Jan 27 14:31:11 2023 DL1C virt_prim_rach.c:81 MS 0000: Rx L1CTL_RACH_REQ (ra=0x0d, offset=123 combined=1)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 2)
Fri Jan 27 14:31:11 2023 DL1C l1ctl_sap.c:404 MS 0000: Rx L1CTL_PARAM_REQ (ta=0, tx_power=7)
Fri Jan 27 14:31:12 2023 DL1C l1ctl_sap.c:404 MS 0000: Rx L1CTL_PARAM_REQ (ta=0, tx_power=15)
Fri Jan 27 14:31:12 2023 DL1C l1ctl_sap.c:375 MS 0000: Rx L1CTL_DM_REL_REQ
Fri Jan 27 14:31:12 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 1)
Fri Jan 27 14:31:12 2023 DL1C l1ctl_sap.c:702 MS 0000: Tx L1CTL_RESET_CONF (reset_type: 1)
Fri Jan 27 14:31:12 2023 DL1C virt_prim_fbsb.c:56 MS 0000: Rx L1CTL_FBSB_REQ (arfcn=868, flags=0x7)
Fri Jan 27 14:31:13 2023 DL1C virt_prim_fbsb.c:125 MS 0000: Tx L1CTL_FBSB_CONF (res: 0)
Fri Jan 27 14:31:13 2023 DL1C l1ctl_sap.c:517 MS 0000: Rx L1CTL_NEIGH_PM_REQ (list with 0 entries): IGNORED
```

### Start Mobile station (MS)

```
root@osmo-server:~# mobile -c /etc/osmocom/mobile.cfg 
Copyright (C) 2010-2015 Andreas Eversberg, Sylvain Munaut, Holger Freyther, Harald Welte
Contributions by Alex Badea, Pablo Neira, Steve Markgraf and others

License GPLv2+: GNU GPL version 2 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

<0011> app_mobile.c:278 Mobile '1' initialized, please start phone now!
<0015> telnet_interface.c:88 Available via telnet 127.0.0.1 4247
<0001> gsm48_rr.c:1925 Changing CCCH_MODE to 2
<0001> gsm48_rr.c:1732 Complete set of SI5* for BA(0)
<0005> gsm48_mm.c:2512 follow-on proceed not supported.
<0002> gsm322.c:3838 Event unhandled at this state.
```


## Examples, management and testing

### HLR

`telnet 0  4258`
```
OsmoHLR> subscriber id 1 show
    ID: 1
    IMSI: 001010000000001
    MSISDN: 555501
    2G auth: COMP128v1
             KI=00000000000000000000000000000000
```

### BTS

`telnet 0 4241`
```
OsmoBTS> show bts 0
BTS 0 is of FIXME type in band DCS1800, has CI 0 LAC 1, BSIC 63 and 1 TRX
  Description: (null)
  Unit ID: 1800/0/0, OML Stream ID 0x00
  NM State: Oper 'Enabled', Admin 'Unlocked', Avail 'OK'
  Site Mgr NM State: Oper 'Enabled', Admin 'unknown 0x0', Avail 'OK'
  Paging: Queue size 200, occupied 0, lifetime 0s
  AGCH: Queue limit 12, occupied 0, dropped 0, merged 0, rejected 0, ag-res 1, non-res 2
  CBCH backlog queue length: 0
  Paging: queue length 0, buffer space 200
  OML Link state: connected.
  TRX 0
    phy 0 
  Features:
    006 OML Alerts                              
    009 Fullrate speech V1                      
    010 Halfrate speech V1                      
    011 Fullrate speech EFR                     
    012 Fullrate speech AMR                     
    013 Halfrate speech AMR                     
  base transceiver station:
   Received paging requests (Abis):        6 (0/s 0/m 6/h 0/d)
   Dropped paging requests (Abis):        0 (0/s 0/m 0/h 0/d)
   Sent paging requests (Um):        4 (0/s 0/m 4/h 0/d)
   Received RACH requests (Um):        3 (0/s 0/m 3/h 0/d)
   Dropped RACH requests (Um):        0 (0/s 0/m 0/h 0/d)
   Received RACH requests (Handover):        0 (0/s 0/m 0/h 0/d)
   Received RACH requests (CS/Abis):        3 (0/s 0/m 3/h 0/d)
   Received RACH requests (PS/PCU):        0 (0/s 0/m 0/h 0/d)
   Received AGCH requests (Abis):        3 (0/s 0/m 3/h 0/d)
   Sent AGCH requests (Abis):        3 (0/s 0/m 3/h 0/d)
   Sent AGCH DELETE IND (Abis):        0 (0/s 0/m 0/h 0/d)
```

```
OsmoBTS> show trx 
TRX 0 of BTS 0 is on ARFCN 868
Description: (null)
  RF Nominal Power: 0 dBm, reduced by 0 dB, resulting BS power: 0 dBm
  NM State: Oper 'Enabled', Admin 'Unlocked', Avail 'OK'
  RSL State: connected
  Baseband Transceiver NM State: Oper 'Enabled', Admin 'Unlocked', Avail 'OK'
  IPA stream ID: 0x00
```

### MS

`telnet 0 4247`
```
OsmocomBB> show ms 1
MS '1' is up, service is normal
  IMEI: 000000000000001
     IMEISV: 0000000000000010
     IMEI generation: fixed
  automatic network selection state: A2 on PLMN
                                     MCC=001 MNC=01 (Test, Test)
  cell selection state: C3 camped normally
                        ARFCN=868(DCS) MCC=001 MNC=01 LAC=0x0001 CELLID=0x0000
                        (Test, Test)
  radio resource layer state: idle
  mobility management layer state: MM idle, normal service
```

```
OsmocomBB> show cell 1
ARFCN  |MCC    |MNC    |LAC    |cell ID|forb.LA|prio   |min-db |max-pwr|rx-lev
-------+-------+-------+-------+-------+-------+-------+-------+-------+-------
 868DCS|001    |01     |0x0001 |0x0000 |no     |normal |-110   |   7   |-63
```

### MSC

`telnet 0 4254`
```
OsmoMSC> subscriber msisdn 555501 paging 
% paging subscriber
```

```
OsmoMSC> subscriber msisdn 555501 sms sender msisdn 555502 send 'Test SMS'
```

```
root@osmo-server:~# cat .osmocom/bb/sms.txt 
[SMS from 555502]
'Test SMS'
```

### BSC

`telnet 0 4242`<br>
Do a paging from the MSC and then soon after,
```
OsmoBSC> show subscriber all 
 IMSI             TMSI      LAC    Use
 001010000000001  9e234e00      1  1
```

### MGW

`telnet 0 4243`

### STP

`telnet 0 4239`


### tcpdump / Wireshark

```
root@osmo-server:~# tcpdump -i any -w gsm.pcap not tcp port 22
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked v1), capture size 262144 bytes
1737 packets captured
2172 packets received by filter
0 packets dropped by kernel
```
dst/src 127.0.0.1 is tcp based core traffic<br>
dst/src 185/187 is sctp based core traffic<br>
dst 239.193.23.1 is virtual radio traffic<br>
[Wireshark](images/gsm.pcap.png)
