---
title: How to setup self-hosted openvpn server
date: 2022-06-13 16:06 +0700
author: hieu
categories: [Cloud/DevOps, OpenVPN]
tags: [devops, openvpn]
---

## Prerequisite

A VM (ubuntu 20.04) should be available for the following steps. 

### Step 1 - Installing OpenVPN and Easy-RSA

```bash
sudo apt update
sudo apt install openvpn easy-rsa
mkdir ~/easy-rsa # create a new directory on the OpenVPN Server as your non-root user
ln -s /usr/share/easy-rsa/* ~/easy-rsa/ # create a symlink from the easyrsa script that the package installed into the ~/easy-rsa directory
sudo chown azureuser ~/easy-rsa # ensure the directory’s owner is your non-root sudo user
chmod 700 ~/easy-rsa # restrict access to that user
```
{: file='bash'}
### Step 2 - Creating a PKI for OpenVPN

```bash
cd ~/easy-rsa
vi vars # create vars file
```
{: file='bash'}

paste the following 2 lines into vars file

```
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```
{: file='~/easy-rsa/vars'}

Once you have populated the vars file you can proceed with creating the PKI directory

```bash
./easyrsa init-pki
```
{: file='bash'}

### Step 3 — Creating an OpenVPN Server Certificate Request and Private Key

```bash
cd ~/easy-rsa
./easyrsa gen-req server nopass # call the easyrsa with the gen-req option followed by a Common Name (server in this case) for the machine
```
{: file='bash'}

This will create a private key for the server and a certificate request file called server.req. Copy the server key to the /etc/openvpn/server directory:

```bash
sudo cp /home/azureuser/easy-rsa/pki/private/server.key /etc/openvpn/server/
```
{: file='bash'}

### Step 4 — Creating the rootCA and signing the OpenVPN Server’s Certificate Request

```bash
mkdir ~/CA # create folder to store rootCA key and rootCA cert
cd ~/CA 
openssl genrsa -des3 -out ca.key 4096 # create root key
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -out ca.crt # Create and self sign the Root Certificate
cp ~/easy-rsa/pki/reqs/server.req ~/CA # copy server.req to CA directory
openssl x509 -req -in server.req -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha512 # generate server certificate
sudo cp ~/CA/{server.crt,ca.crt} /etc/openvpn/server # copy server.crt and ca.crt to /etc/openvpn/server
```
{: file='bash'}

### Step 5 — Configuring OpenVPN Cryptographic Material

```bash
cd ~/easy-rsa
openvpn --genkey --secret ta.key
sudo cp ta.key /etc/openvpn/server
```
{: file='bash'}

### Step 6 — Generating a Client Certificate and Key Pair

```bash
mkdir -p ~/client-configs/keys
chmod -R 700 ~/client-configs
cd ~/easy-rsa
./easyrsa gen-req client1 nopass # generate client1 key and signing certification request
cp pki/private/client1.key ~/client-configs/keys/ # copy client1.key to ~/client-configs/keys/
cp pki/reqs/client1.req ~/CA # copy client1.req to ~/CA
openssl x509 -req -in client1.req -CA ca.crt -CAkey ca.key -CAcreateserial -out client1.crt -days 365 -sha512 # generate client1 certificate
cp ~/CA/client1.crt ~/client-configs/keys/
cp ~/easy-rsa/ta.key ~/client-configs/keys/
sudo cp /etc/openvpn/server/ca.crt ~/client-configs/keys/
sudo chown azureuser.azureuser ~/client-configs/keys/*
```
{: file='bash'}

### Step 7 — Configuring OpenVPN

Firstly, copy the sample server.conf file as a starting point for your own configuration file:

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/server/
sudo gunzip /etc/openvpn/server/server.conf.gz
sudo vi /etc/openvpn/server/server.conf # open server.conf for editting
```
{: file='bash'}

make sure the following line are uncommented by removing the ";" or modified (added/removed):

```conf
tls-crypt ta.key # uncomment
cipher AES-256-GCM # uncomment and change value from AES-256-CBC to AES-256-GCM
auth SHA256 # add this line right after cipher AES-256-GCM
;dh dh2048.pem # comment out this line
dh none # add this line
user nobody # uncomment this line
group nogroup # uncomment this line
```
{: file='server.conf'}

If you selected a different name during the ./easyrsa gen-req server command earlier, modify the cert and key lines in the server.conf

```conf
cert server.crt # modified if needed
key server.key # modified if needed
```
{: file='server.conf'}

### Step 8 — Adjusting the OpenVPN Server Networking Configuration

There are some aspects of the server’s networking configuration that need to be tweaked so that OpenVPN can correctly route traffic through the VPN.

```bash
sudo vi /etc/sysctl.conf
```
{: file='bash'}

Then add the following line at the bottom of the file:

```conf
net.ipv4.ip_forward = 1
```
{: file='/etc/sysctl.conf'}

save and apply the change with:

```bash
sudo sysctl -p
```
{: file='bash'}

### Step 9 — Firewall Configuration

modify before.rules file

```bash
sudo vi /etc/ufw/before.rules
```
{: file='bash'}

copy and paste the following line to /etc/ufw/before.rules

```
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
```

result is:
```conf
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#
 
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
 
# Don't delete these required lines, otherwise there will be errors
*filter
. . .
```
{: file='/etc/ufw/before.rules'}

open /etc/default/ufw and change value of DEFAULT_FORWARD_POLICY to ACCEPT:

```conf
DEFAULT_FORWARD_POLICY="ACCEPT"
```
{: file='/etc/default/ufw'}

add udp and ssh port to firewall:

```bash
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
```
{: file='bash'}

After adding those rules, disable and re-enable UFW to restart it and load the changes from all of the files you’ve modified:

```bash
sudo ufw disable
sudo ufw enable
```
{: file='bash'}

### Step 10 — Starting OpenVPN

run the following commands to start openvpn server as boot and directly:

```bash
sudo systemctl -f enable openvpn-server@server.service
sudo systemctl start openvpn-server@server.service
sudo systemctl status openvpn-server@server.service # check status of the service
```
{: file='bash'}

### Step 11 — Creating the Client Configuration Infrastructure

Get started by creating a new directory where you will store client configuration files within the client-configs directory you created earlier:

```bash
mkdir -p ~/client-configs/files
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
```
{: file='bash'}

open the file and edit the following lines

```conf
remote your_server_ip 1194 # replace your_server_ip with your server ip address or domain name
user nobody # uncomment this line
group nogroup # uncomment this line
;ca ca.crt # comment this line
;cert client.crt # comment this line
;key client.key # comment this line
;tls-auth ta.key 1 # comment this line
cipher AES-256-GCM # uncomment this line
auth SHA256 # uncomment this line
key-direction 1 # add this line directly after auth SHA256 line
```
{: file='~/client-configs/base.conf'}

create a script that will compile your base configuration 

```bash
vi ~/client-configs/make_config.sh
```
{: file='bash'}

Inside, add the following content:

```sh
#!/bin/bash
 
# First argument: Client identifier
 
KEY_DIR=~/client-configs/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf
 
cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-crypt>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-crypt>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```
{: file='~/client-configs/make_config.sh'}

make this file executable:

```bash
chmod 700 ~/client-configs/make_config.sh
```
{: file='bash'}

### Step 12 — Generating Client Configurations

```bash
cd ~/client-configs
./make_config.sh client1
ls ~/client-configs/files
```
{: file='bash'}

from local or client machine, copy client1.ovpn file to the local machine

```bash
sftp azureuser@openvpn_server_ip:client-configs/files/client1.ovpn /path_to_save_file/
```
{: file='bash'}

### Step 13 — Installing the Client Configuration

#### Client Linux machine

install openvpn software on ubuntu

```bash
sudo apt update
sudo apt install openvpn
```
{: file='bash'}

open client1.ovpn and comment out line "remote-cert-tls server" with ";"

```conf
;remote-cert-tls server
```
{: file='client1.ovpn'}

#### Connecting

```bash
sudo openvpn --config client1.ovpn
```
{: file='bash'}