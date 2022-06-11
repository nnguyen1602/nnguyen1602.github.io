---
title: Network Security Exercises
date: 2020-06-12 00:15 +0200
author: hung
categories: [Project, Hobby]
tags: [project, hobby, docker, lab network]
---

## How to Create a LAB VNET with Docker

The system will look like the following diagram: ![](/assets/images/20190808/ITSEC_virtual_network.png)  
**Note: the ip addresses of the VMs are different from the above diagram (also even if you don't use docker but normal VMs tool like VirtualBox...), but the method stays the same.**

1.  `$ docker network create --driver=bridge --subnet=172.16.2.0/24 --ip-range=172.16.2.0/24 br0`
2.  `$ docker network create --driver=bridge --subnet=192.168.1.0/24 --ip-range=192.168.1.0/24 br1`
3.  `$ docker network create --driver=bridge --subnet=10.2.4.0/24 --ip-range=10.2.4.0/24 br1`
4.  
```shell
$ docker run --cap-add=NET_ADMIN -ti --privileged -P --name=router1 --net br0 --ip 172.16.2.4 pthung/itsec-lab /bin/bash  
$ export PS1="router1@172.16.2.4||192.168.1.5: \w \$ "
```
5.  `$ docker run --cap-add=NET_ADMIN -ti --privileged -P --name=router2 --net br0 --ip 172.16.2.5 pthung/itsec-lab /bin/bash  `
    `$ export PS1="router2@172.16.2.5||10.2.4.2: \w \$ "`
6.  Run 2 end machines:  
```shell
$ docker run -ti --cap-add=NET_ADMIN --privileged --name=vm1 --net br1 --ip 192.168.1.100 pthung/itsec-lab /bin/bash  
$ export PS1="machine1@192.168.1.100: \w \$ "  
$ docker run -ti --cap-add=NET_ADMIN --privileged --name=vm2 --net br2 --ip 10.2.4.37 pthung/itsec-lab /bin/bash  
$ export PS1="machine2@10.2.4.37: \w \$ "
```
7.  Add interfaces to routers:  
    `$ docker network connect br1 router1 --ip 192.168.1.5  `
    `$ docker network connect br2 router2 --ip 10.2.4.2`
8.  Setup iptabble for both routers:  
```shell
    $ iptables -t nat -A POSTROUTING --out-interface eth1 -j MASQUERADE  
    $ iptables -A FORWARD --in-interface eth0 -j ACCEPT  
    $ iptables -t nat -A POSTROUTING --out-interface eth0 -j MASQUERADE  
    $ iptables -A FORWARD --in-interface eth1 -j ACCEPT
```
9.  For router1: `$ route add -net 10.2.4.0 netmask 255.255.255.0 gateway 172.16.2.5 (ip address of router 2)`
10.  For router2: `route add -net 192.168.1.0 netmask 255.255.255.0 gateway 172.16.2.4 (ip address of router 1)`
11.  For vm1: `$ route add default gateway 192.168.1.5`
12.  For vm2: `$ route add default gateway 10.2.4.2`

Examples for firewall:

`$ iptables -A INPUT -s 192.168.1.0/24 -d 192.168.1.0/24 --protocol tcp --tcp-flags ALL SYN,ACK -j DROP`  
`$ iptables -A FORWARD -m conntrack --ctstate NEW -j DROP` (for routers)

## Firewall practice

1.  Check packet filter rule:

    `$ iptables -v -L`

2.  Drop all forward ICMP packet in router 2:

    `$ iptables -A FORWARD -p ICMP -j DROP`

3.  Flush all the rule in FORWARD:

    `$ iptables -F FORWARD`

4.  **Stateless:**

    Only allow outgoing TCP connection:

    `$ iptables -A FORWARD -d 10.2.4.0/24 -p TCP --tcp-flags SYN,ACK SYN -j DROP`

    Or with error message:

    `$ iptables -A FORWARD -d 10.2.4.0/24 -p TCP --tcp-flags SYN,ACK SYN -j REJECT`

5.  Stateful with conntrack

    Change policy of FORWARD table to DROP:

    `iptables -P FORWARD DROP`

6.  Only allow outgoing new TCP connection:
```shell
    $ iptables -A FORWARD -s 10.2.4.0/24 -m conntrack --ctstate NEW -j ACCEPT  
    $ iptables -A FORWARD -m conntrack --ctstate ESTABLISHED -j ACCEPT
```
7.  **NAT**

    Delete the route to 192.168.1.0/24 network on router 2:

    `$ route del -net 192.168.1.0/24 gw 172.16.2.4  `

    Create source NAT at router 1:

    `$ iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 172.16.2.4`

    Create destination NAT at router 1:

    `$ iptables -t nat -A PREROUTING -p TCP --dport 8080 -i eth0 -j DNAT --to 192.168.1.100:8180`

### Application Level Gateway recap:

Advantages:

*   user specific rule
*   Fine-gained control (communication peers, functions)
*   Content checking (detect malware, suspicious content)
*   No direct connection between sender/receiver
*   Logging

Disadvantages:

*   Applciation specific
*   expose to attacks

### Weakness in the concept of Firewall:

*   Mobile devices
*   Tunnels and encryption

### Other notes
`$ iptables -A INPUT -p tcp --dport 7001 -j ACCEPT `
`$ iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT`

## OpenVPN practice

**For RHEL**  
**Server:**

1.  Install openvpn:  
    `$ yum install openvpn`
2.  Get server template file:  
    `$ cd /usr/share/doc/openvpn-*/sample/sample-config-files/  `
    `$ cp server.conf /etc/openvpn`
3.  Generate keys and certificate and DH file with this: [How to Generate Own Certificates](/posts/how-to-generate-self-signed-certificates/)
4.  Change server.conf file:  
```shell
    push "redirect-gateway def1 bypass-dhcp"  
    push "dhcp-option DNS 8.8.8.8"  
    push "dhcp-option DNS 8.8.4.4"  
    user nobody  
    group nobody  
    tls-auth ta.key 0
```
5.  Make sure these 3 files are in /etc/openvpn/ : ca.crt server.crt server.key
6.  In /etc/openvpn/, generate ta key file:  
    `$ openvpn --genkey --secret ta.key`
7.  Enable and start server:  
    `$ systemctl start openvpn@server  `
    `$ systemctl enable openvpn@server`

**Client:**

1.  Install openvpn:  
    `$ yum install openvpn`
2.  Generate keys and certificate and DH file with this: [How to Generate Own Certificates](/posts/how-to-generate-self-signed-certificates/)
3.  Copy the ta key file from server to client
4.  Create client.ovpn file:
```shell
    client  
    dev tun  
    proto udp  
    remote ip-address-of-server openvpn-port  
    resolv-retry infinite  
    nobind  
    persist-key  
    persist-tun  
    verb 3  
    ca ca.crt  
    cert client2.crt  
    key client2.key  
    tls-auth ta.key 1
```

5.  Run:  
    `openvpn --config client.ovpn`


**Create config file for OPENVPN tap devices:**

1.  Config file for bridge client:  

```shell
    dev tap0  
    proto tcp-client  
    tls-client  
    remote 172.16.2.5  
    ca /sec/certificates/rootCA-cert.pem  
    cert /sec/certificates/192.168.1.100-cert.pem  
    key /sec/certificates/192.168.1.100-private-key.pem
```

2.  Start script for client: 

```shell
    #!/bin/bash  
    openvpn --mktun --dev tap0  
    ifconfig tap0 up  
    ifconfig tap0 10.2.4.212 netmask 255.255.255.0  
    openvpn --config /sec/openvpn/tap/bridgeclient.conf
```

3.  Config file for bridge server: 

```shell
    dev tap0  
    proto tcp-server  
    tls-server  
    ca /sec/certificates/rootCA-cert.pem  
    cert /sec/certificates/10.2.4.2-cert.pem  
    key /sec/certificates/10.2.4.2-private-key.pem  
    dh /sec/certificates/dh2048.pem
```
4.  (Not succesful yet) Start script for client:  

```shell
    #!/bin/bash  
    openvpn --mktun --dev tap0  
    ifconfig tap0 up  
    ifconfig tap0 promisc  
    ifconfig eth0 promisc  
    brctl addbr mybridge  
    brctl addif mybridge tap0  
    brctl addif mybridge eth0  
    ifconfig tap0 0.0.0.0  
    ifconfig eth0 0.0.0.0  
    ifconfig mybridge 172.16.2.5 netmask 255.255.255.0 up  
    openvpn --config /sec/openvpn/tap/bridgeserver.conf  
```