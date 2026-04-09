# Pfsense Firewall

1. Introduction
2. Objective
3. System Requirements
4. Network Diagram
5. Creating Adapters
6. Installation of pfSense
7. Configuration
8. Testing & Results
9. Challenges Faced
10. Security Improvements 

## Introduction

- What is pfsense : A open-source network firewall designed to run  on computers or VMs
- Using pfsene firewall to secure networks functions as stateful firewall and network router, Manages through web-based interface

## Objective

- Configuring firewall rules
- Securing network traffic
- Simulating Attacks and monitor logs

### System Requirements

- [pfsense](https://www.pfsense.org/download/) ISO
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [VMware Workstation](https://support.broadcom.com/group/ecx/downloads)
- Windows Machine, (wireshark)

**Network Adapter**

- NAT - WAN
- Internal Network (GREEN)
- Internal Network (dmz)

## Network Diagram

![image.png](Pfsense%20Firewall/image.png)

```jsx
WAN : DHCP
LAN : 192.168.1.1/24
pfsense : 192.168.1.1
Windows : 192.168.1.10
KaliLinux : 192.168.1.14
```

## Creating Adapters

Go to VM Settings > Network Tab > Activate 3 adapters 

> Make note of MAC Address of each adapters
> 

Adapter 1: NAT : faces the internet, firewall gets its public IP here

Adapter 2: Internal Netwrok > GREEN : faces internal machines, pfsense acts as gateway for everything on this side

Adapter 3: Internal Network > dmz : for placing vulnerable target VMs, Simulating web attacks

![image.png](Pfsense%20Firewall/image%201.png)

![image.png](Pfsense%20Firewall/image%202.png)

![image.png](Pfsense%20Firewall/image%203.png)

## Installation of  pfSense

1. Download pfSense ISO
2. Settings > Network tab > Internal Network 
3. Boot VM
4. Install pfSense
5. Assign Interfaces: 
- WAN
- LAN

Select install pfsense

![image.png](Pfsense%20Firewall/image%204.png)

Use the default configuration

![image.png](Pfsense%20Firewall/image%205.png)

![image.png](Pfsense%20Firewall/image%206.png)

![image.png](Pfsense%20Firewall/image%207.png)

- Reboot

![image.png](Pfsense%20Firewall/image%208.png)

## Assigning Interfaces

- select option 1

> Take a look at the MAC Address which we made note earlier
> 
- Select em0 for WAN
- Select em1 for LAN
- Select em2 for DMZ
- Select Y to procced and the configuration is done

### Configuration

- Select the option 2 Set interface(s) IP address
- Choose the LAN interface (em1) depends on you network adapter
- Enter the desired IP address : `192.168.1.1/24`
- If want to enable DHCP enter the range of IP addresses
- Save the changes
- I prefer assigning IP of WAN interface from DHCP, or later we can change it in the Admin panel

## Configuring Windows IP

![image.png](Pfsense%20Firewall/image%209.png)

- IP address: `192.168.1.10`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.1.1` (pfsense firewall)
- DNS : `8.8.8.8`

## Configuring Kali linux IP

- IP address: `192.168.1.14`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.1.1` (pfsense firewall)
- DNS : `8.8.8.8`

## pfSense Admin panel

Access the Admin panel `http://192.168.1.1` 

Creds: admin/pfsense

![image.png](Pfsense%20Firewall/image%2010.png)

The setup wizard : Configure the basic settings for pfsense installation by following the wizard

## Firewall Configuration :

![image.png](Pfsense%20Firewall/image%2011.png)

**Firewall > Rules > LAN : Here we can add, modify and delete rules**  

First we’ll allow the IP `192.168.1.14` of attacker machine to pass through pfsense firewall to perform attack on windows machine

## Monitoring Traffic Flow : Wireshark

**Ping scan :** 

![image.png](Pfsense%20Firewall/image%2012.png)

## Simulating Attack

Attacking from Kali Linux (Attacker machine)

Performing nmap aggressive scan on the victims machine 

> `nmap -A 192.168.1.10`
> 

![image.png](Pfsense%20Firewall/image%2013.png)

## Monitoring traffic flow from pfsense firewall

When we attack Machine 2 (target) from Machine 1 (Kali), all traffic flows through pfSense since it's the gateway

> **pfsense firewall logs**
> 

Navigate to Status > System Logs > Firewall tab. Every connection attempt hits a firewall gets logged here 

![image.png](Pfsense%20Firewall/image%2014.png)

- Status > Traffic Graph > Select interface : shows live bandwidth per interface

![image.png](Pfsense%20Firewall/image%2015.png)

Navigate to Firewall > Rules > LAN : We can see the increase of bandwidth of ip `192.168.1.14` under the rules tab as we’re performing port scanning with nmap 

![image.png](Pfsense%20Firewall/image%2016.png)

## Setting up Firewall Rules

Blocking the IP by editing the rule 

Action  > Block > Save

![image.png](Pfsense%20Firewall/image%2017.png)

- Active  : `Block`
- Interface : `LAN`
- Address Family : `IPv4`
- Protocol : `ANY`

## Verifying the Rules

![image.png](Pfsense%20Firewall/image%2018.png)

![image.png](Pfsense%20Firewall/image%2019.png)

![image.png](Pfsense%20Firewall/image%2020.png)

## Challenges Faced

- Interface confusion (WAN vs LAN)
- No internet due to wrong gateway
- Logs not forwarding

## Security Improvements

```jsx
Security Improvements Based on Nmap Scanning

The Nmap -A scan and ping scan were performed to analyze the exposure of the network. The scan results revealed open ports, active hosts, and service information, which could be exploited by attackers.

To enhance network security, the following measures were implemented:

1. ICMP (ping) requests were blocked on the WAN interface to prevent host discovery.
2. Unnecessary ports were closed, and only required services were allowed through firewall rules.
3. Unused services were disabled to reduce vulnerability exposure.
4. Network Address Translation (NAT) was used to hide internal IP addresses.
5. Logging and monitoring were enabled to detect suspicious activities.
6. A default deny rule was applied to block all unauthorized traffic.

These improvements significantly reduced the attack surface and improved the overall security of the network.
```