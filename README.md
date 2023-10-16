# Wazuh

Wazuh is a security platform that provides unified XDR and SIEM protection for endpoints and cloud workloads. The solution is composed of a single universal agent and three central components: the Wazuh server, the Wazuh indexer, and the Wazuh dashboard.

wazuh.com

![wazuh logo](https://i.ibb.co/L1bqRk1/Wazuh.png)

## Goals

The principal goal of this Makejail is to help us install, configure and run wazuh-indexer (opensearch), wazuh-manager, logstash, filebeat and wazuh-dashboards (opensearch-dashboard + wazuh-kibana-app) quickly. Take on mind this container as is, is for testing/learning purpose and it is not recommended for production because it has a minimal configuration for run wazuh.

![wazuh dashboard 1](https://user-images.githubusercontent.com/11150989/204661974-141395d0-dda0-4573-8ea6-4d3b17ad2759.png)

![wazuh dashboard 2](https://user-images.githubusercontent.com/11150989/204662101-75880698-8cfd-4aa9-b0ac-e9bac011cd5c.png)

## Requirements

Before you can install wazuh using this Makejail you need some initial configurations.

### Enable Packet filter

We need to add some lines to `/etc/rc.conf`:

```console
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

`rdr-anchor` is necessary for use dynamic redirect from jails.

### Enable forwarding

```console
# sysrc gateway_enable="YES"
# sysctl net.inet.ip.forwarding=1
```
### Bootstrap a FreeBSD version

Before you can begin creating containers, AppJail needs to fetch and extract components for create jails. If you are creating FreeBSD jails it must be a version equal or lesser than your host version.

```console
# appjail fetch
```

### Create a virtualnet

```console
# appjail network add wazuh-net 10.0.0.0/24
```

It will create a bridge named `wazuh-net` on which the epair interfaces will be attached. By default, this Makejail will use NAT for Internet outbound. Do not forget to add a `pass` rule in your `/etc/pf.conf` because this Makejail will try to download and install packages and some other resources for the configuration of Wazuh services.

```
pass out quick on wazuh-net inet proto { tcp udp } from 10.0.0.2 to any
```

### Create a lightweight container system

Create a container named `wazuh` with a private IP address `10.0.0.2`. Take on mind that the IP address must be part of `wazuh-net` network.

```console
# appjail makejail -f gh+AppJail-makejails/wazuh -j wazuh -- --network wazuh-net --server_ip 10.0.0.2
```

When it is done you will see credentials info for connect to wazuh-dashboards via web browser and one password to agent enrollment.

```
################################################ 
 Wazuh dashboard admin credentials                
 Hostname : https://jail-or-host-ip:5601/app/wazuh   
 Username : admin                                 
 Password : @hkXudpIp93xbIOvD                          
################################################
 Wazuh agent enrollment password                
 Password : @sXDudSIKJKfMTmCroHGvirVPE80=
################################################
```

Keep it to another secure place.

## License

This project is licensed under the BSD-3-Clause license.
