---
title: How to generate self-signed certificates
author: hung
date: 2020-06-11 20:00:00 +0200
categories: [Security, Certificates]
tags: [security, certificate]
---

**Setup certificate**

1.  Create private root CA key:  
    `openssl genrsa -des3 -out private-CA-key.pem 4096`
2.  Create self sign Root Certificate:  
    `openssl req -x509 -new -nodes -key private-CA-key.pem -sha256 -days 500 -out rootCA-cert.pem`
3.  Create CSR for a server (or machine), remember to change "machine" to your machine's name or ip address (also common name):  
```shell
    openssl genrsa -des3 -out machine-private-key.pem 4096  
    openssl req -new -key machine-private-key.pem -out machine-csr.csr 
```

    or in one line:  
```
    openssl req -nodes -newkey rsa:2048 -keyout example.key -out example.csr -subj "/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=example.com"
```
4.  Sign the machine CSR with private root CA key and root CA certificate:  
    `openssl x509 -req -in machine-csr.csr -CA rootCA-cert.pem -CAkey private-CA-key.pem -CAcreateserial -days 500 -sha256 -out machine-cert.pem`
5.  Create DH file:  
    `openssl dhparam -out dh2048.pem 2048`

Check signature algorithm

`openssl x509 -noout -text -in machine-cert.pem | grep "Signature Algorithm" | uniq` **For JKS:**

1.  Create keystore:  
    `keytool -genkey -alias server_name -keyalg RSA -keysize 2048 -keystore server_name.jks`
2.  Generate a CSR:  
    `keytool -certreq -alias server_name -file server_name.csr -keystore server_name.jks`
3.  Sign the CSR with:  
    `openssl x509 -req -in machine-csr.csr -CA rootCA-cert.pem -CAkey private-CA-key.pem -CAcreateserial -days 500 -sha256 -out machine-cert.pem`
4.  Import the CA and signed-certificate into keystore: 
```shell
    keytool -import -keystore clientkeystore -file ca-certificate.pem.txt -alias theCARoot  
    keytool -import -trustcacerts -alias alias_name -file certificate_file -keystore keystore_file
```