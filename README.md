# wazuh-agent-makejail
Wazuh-agent-makejail is a [AppJail](https://github.com/DtxdF/AppJail) template ([AppJail-makejail](https://github.com/AppJail-makejails)) used by deploy a testing [Wazuh](https://wazuh.com/) agent on [FreeBSD](https://freebsd.org/). The principal goals are helps us to fast way install, configure and run wazuh-agent into a FreeBSD jail. It can be helpful for monitoring jail containers. Take on mind this container as is must be used by testing/learning purpose and it is not recommended for production because it has a minimal configuration for run wazuh.

![image](https://github.com/alonsobsd/wazuh-agent-makejail/assets/11150989/c72cdfda-7a38-40a8-8866-dac31682d407)

![image](https://github.com/alonsobsd/wazuh-agent-makejail/assets/11150989/0b0a3b1a-b14b-4763-970d-df27ff12100f)

## Requirements
Before you can install wazuh using this template you need a working wazuh-manager and some initial configurations. For deploy a wazuh single-node cluster you can use my [wazuh-makejail] (https://github.com/alonsobsd/wazuh-makejail).

#### Enable Packet filter
We need add somes lines to /etc/rc.conf

```sh
# sysrc pf_enable="YES"
# sysrc pflog_enable="YES"

# cat << "EOF" >> /etc/pf.conf
nat-anchor 'appjail-nat/jail/*'
nat-anchor "appjail-nat/network/*"
rdr-anchor "appjail-rdr/*"
EOF
# service pf reload
# service pf restart
# service pflog restart
```
rdr-anchor section is necessary for use dynamic redirect from jails

### Enable forwarding
```sh
# sysrc gateway_enable="YES"
# sysctl net.inet.ip.forwarding=1
```
#### Bootstrap a FreeBSD version
Before you can begin creating containers, AppJail needs fetch and extract components for create jails. If you are creating FreeBSD jails it must be a version equal or lesser than your host version. In this example we will create a 13.2-RELEASE bootstrap

```sh
# appjail fetch
```
#### Create a virtualnet
Create a virtualnet for add wazuh jail to it from wazuh-makejail

```sh
# appjail network add wazuh-net 10.0.0.0/24
```
it will create a bridge named wazuh-net in where wazuh-agent jail epair interfaces will be attached. By default wazuh-agent-makejail will use NAT for internet outbound. Do not forget added a pass rule to /etc/pf.conf because wazuh-agent-makefile will try to download and install packages and some another resources for configuration of it

```sh
pass out quick on wazuh-net inet proto { tcp udp } from 10.0.0.3 to any
```
Also, you need add a pass rule from wazuh-net network to wazuh-manager. In this example, wazuh-manager is running at 10.0.0.2

```sh
pass in inet proto { tcp udp } from 10.0.0.0/24 to 10.0.0.2
```

#### Create a lightweight container system
Create a container named agent01 with a private IP address 10.0.0.3. Take on mind IP address must be part of wazuh-net network

```sh
# appjail makejail -f gh+alonsobsd/wazuh-makejail -j agent01 -- --network wazuh-net --agent_ip 10.0.0.3 --agent_name agent01 --server_ip 10.0.0.2
```
When it is done, agent01 will try connect to wazuh-manager (10.0.0.2) for auth process

## License
This project is licensed under the BSD-3-Clause license.
