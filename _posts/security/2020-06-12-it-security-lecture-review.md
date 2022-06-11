---
title: IT Security Lecture Review
date: 2020-06-12 00:33 +0200
author: hung
categories: [Security]
tags: [security, review]
---

## I. Introduction

### 1\. Define the term "IT-Security"

Protection of information and information systems against unauthorized access and modification and availability of information system services for legitimate users, including measures to thwart, discover or log threats. Protection against unauthorized access must be ensured during storage, processing or in transit.

### 2\. What are the aims of “IT-Security”? Describe each aim briefly and present an example.

*   Confidentiality: Protection against unauthorized access
*   Integrity: Protection against unauthorized modification
*   Availability: Resources and Services are available for legitimate users
*   Authenticity and authentication: Unambiguous idenfification of the sender of information or a communication peer
*   Nonrepudiation: Ability to prove that a given message was sent and by whom to a third party not involved in communication
*   Authorization: Accces to resources is limited to certain group of (authenticated) users

### 3\. What are passive and active attacks and why is the distinction between them helpful in practice?

Passive Attacks

*   Attacker does not actively participate
*   Example: Snooping and wiretapping
*   Difficult to discover
*   Preventive means of protection against active attacks

Active attacks

*   Attacker actively manipulates systems or data
*   Manipulation of data or systems
*   Attacker often leaves traces
*   Prevention and Detection possible

## II. Cryptographical Principles and Methods

Cryptographic model:  
![](/assets/images/20190808/crypto-model.png)  
Classical Encryption Methods:

*   Substitution
*   Transposition
*   Product cipher
*   Applying multiple substitution and/or transposition

### 1\. Symmetrical Encryption Schemes

![](/assets/images/20190808/semetricscheme.png)

### 2\. Public-Key Methods

![](/assets/images/20190808/publicmethod.png)

### 3\. Hybrid Methods

![](/assets/images/20190808/hybrid.png)

### 4\. Modes of Operation

Electronic Code Book  
Cipher Block Chaining  
Cipher Feedback Mode  
Output Feedback Mode  
Counter Mode

## III. Authentication

Authentication and authenticity: Unambiguous identification of the sender of information or a communication peer

### 1\. Factors for Authentication

*   What you know: password
*   What you have: token
*   What you are: biometric

### 2\. Methods

Size of Passowrd domain:  
![](/assets/images/20190808/pass-size.png)  

Authentication by Symmetric Key  
![](/assets/images/20190808/authsym.png)  

Authentication by Public Key Cryptography  
![](/assets/images/20190808/authpub.png)  

### 3\. Digital Signatures

Root Certificate  
Certificate for products (servers...)  
Certificate Request (come along with csr file)  
Public-private key pair  

The CA have its own signed certificate, and private key.  
An organization create public-private key pair, and certificate signed request.  
The CA must sign the CSR with its private key and send back to organization.  

From the internet:  
You usually start by generating a private key / public key pair, followed by a CSR (Certificate Signing Request). The CSR would contain a copy of the public key and some basic information about the subject. Once you've generated a CSR, you would then submit that CSR to a CA. Once the CA is done signing the cert, the CA would then return the cert to you and you would then import that signed certificate to your server. If we recall our discussion on digital certificates, the signed cert would contain some basic information regarding the subject (your site), the issuer, the validity period, the public key (of your site), and a digital signature of the cert signed using the CA's private key.  
In summary, You generate a private key / public key pair and submit a CSR to a Certificate Authority. The contents of the CSR will form part of the final server certificate. The CA verifies whether the information on the certificate is correct and then signs it using its (the CA's) private key. It then returns the signed server certificate to you. You import the signed server certificate unto your server.

## IV. Operating System Security

![](/assets/images/20190808/ossec.png)

Vulnerabilities in Services  
![](/assets/images/20190808/vulnebi.png)

## V. Introduction to Network Security

**Threats** ![](/assets/images/20190808/intethreat.png)

*   Sniffing
*   Manipulation of a message
*   ARP-Spoofing:  
    ![](/assets/images/20190808/arp-spoofing.png)
*   IP-Spoofing:  
    ![](/resource//img/ip-spoofing.png)
*   TCP sequence attack:  
    ![](/assets/images/20190808/tcp-sequence-attack.png)
*   DNS-Spoofing:  
    ![](/assets/images/20190808/dns-spoofing.png)

**Security:**

*   Application Layer Security
*   Security on Transport Layer
*   Network Layer Security
*   Data Link Layer Security

## VI. Firewall

![](/assets/images/20190808/firewall-solution.png)

Problems and Limits of Packet filter:

*   No user or application-specific filtering possible
*   Use of Transport-layer information to identify application protocols
*   Application with varying port number
*   Tunneling and encryption
