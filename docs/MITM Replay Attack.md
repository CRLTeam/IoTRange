# Man In The Middle Replay Attack

# Access Level

Network

# Type of Attack

MITM & Replay

# Description

## How The Attack Works

The attacker gains access to the network and is on the same subnet as the target device. Because network switches send packets targeted to various network ports using MAC addresses, traditional “on-wire” sniffing attacks will not work. With ARP spoofing, we send Address Relay Protocol frames to the devices we want to be in the middle of. You can read more about ARP in the Techniques section. This changes the mapping table of IP addresses/MAC addresses. 

An easy way to think about ARP Poisoning is: Alice and Bob are chatting. I send a message to each of them, telling Alice that I am now Bob, and Bob that I am now Alice. However, neither Alice nor Bob know that I am masquerading as them. This is because Alice and Bob have trust in the system that tells them that Bob is at address B, etc. They still believe they are talking to each other. Because I receive both their communications, I can do things like read their unsecured communications, block all or certain communications, and modify messages prior to them arriving at their destination.

To summarize the Alice and Bob example: I changed the numbers in their respective contact apps to my own number, and am passing their messages along to them after reading them. Because they didn’t check the contact app to make sure they were sending it to the correct number, they are sending things to me unknowingly.

As the attacker, I’ll use ARP Poisoning to tell the Controller and the Door Lock that I am in fact, the Controller and Door Lock. All their traffic will pass through me. Similarly, with a cloud connected IoT device you could ARP Poison between the IoT Device and the Router. I will then use Wireshark to capture and search through the packets sent between the devices, looking for a REST call or other traffic that may identify a Unlock Door signal. In this case, I will search Wireshark for a POST REST call, construct the original REST call from the information, and replay it to unlock the door. 

## System Architecture

![Alt text](https://github.com/CRLTeam/IoTRange/blob/main/images/MITM%20Replay%20Attack/image18.png?raw=true)

### Normal Behavior of Door Lock System

End user swipes card on the Card Reader. Card Reader communicates with Controller through the network with card details. If it matches the database, Controller sends unlock command to Door Lock. 

### Anomalous Behavior of System

After entering the network, we have identified devices on the network named Controller and Door Lock. We identify the traffic from the Controller to the Door Lock, and can either reconstruct the request and re send it, or actually re send the captured packet itself.  

## Techniques

### ARP Poisoning

With ARP Poisoning, instead of sending traffic to your router, you send the information to the MITM who then passes it onto the destination. In most cases, you will never even know the redirection is taking place.

The ARP Protocol has no security - machines believe if they are receiving information from an ARP Packet, that information is legitimate.

ARP Packets can only work on our local subnet - ARP packets cannot traverse outside of the local network. 

Attacker sends ARP packet to machine on network to update ARP Cache. An ARP Cache is a table of IP addresses connected to MAC addresses, and by the attacker sending the ARP packet, they will update the local machines ARP Cache to show themselves as the router. This way, the local machine will send its packets to the attacker instead of to the router. 

# Tools

## Kali Linux

Kali is a Linux distribution with a great collection of pen-testing and security tools.
You can either use Kali on ARM images if you want a separate Raspberry Pi install, or can install it on a system or VM. 
Offensive Security Kali Linux ARM Images
Kali Linux | Penetration Testing and Ethical Hacking Linux Distribution

## Ettercap

Ettercap is a security tool used for MITM attacks
Ettercap Home Page
Unified Sniffing in Ettercap Text

command to run:

`sudo ettercap -G`

## Wireshark

Wireshark is a network sniffing tool that we can use to capture packets on the network and search through them. Wireshark can be also used to sniff other unsecured traffic, such as image files viewed through an unsecured web browser.

# Steps

## Install Kali Linux

Install Kali on a Raspberry Pi or on a Virtual Machine, or a spare computer!

The Kali computer doing the attack must be on the same subnet as the victim machine.

## SSH Into Raspberry Pi

SSH using the command line using a command similar to:

`ssh kali@ip-address-of-pi`

AKA, put in your own IP address

`ssh kali@192.168.1.20`

user:pass is kali:kali as default

Update your Kali installation:

`sudo apt update`
`sudo apt upgrade -y`
Identify Devices
To identify devices on the network, we will do a port scan using nmap.

Open a terminal (keyboard shortcut CTRL+ALT+T) and enter the following:

`sudo nmap -sn 192.168.1.*`

You may need to change the IP address depending on your network. The * indicates a wildcard, i.e. it will scan the entire 192.168.1.0/24 subnet.



While the nmap scan will not give us ‘friendly hostnames’ for example, if you named your macbook “Toms Macbook” but it will give us manufacturer information.


As we are running our IoT devices as raspberry pi, we can see that the raspberry pi’s are the devices we wish to target.

## Set up Man In The Middle with Ettercap

Note: Ettercap requires a GUI Kali installation, it will not work solely through the command line.

command to run:

`sudo ettercap -G`

Note: Keep the Ettercap terminal window open while running Ettercap.
If you need an additional terminal window you can open a new tab like this:


Below is the initial screen of Ettercap:


Ettercap can use a single network interface (unified mode) or two network interfaces (bridged mode).

Unified mode means Ettercap uses a single network interface for sending/receiving to the client as well as to the server, and sniffs all relevant traffic at the single interface.

Bridge mode means Ettercap is using two bridged network interfaces, one connecting to the client and one connecting to the server, and is sniffing traffic crossing that bridge.

For our uses, and most uses, we will want to use Unified mode.



Click the checkmark highlighted in Red to continue.



First click the **Magnifying Glass** highlighted in Red. This will scan for hosts within the subnet.

You will get a message in the bottom similar to:


Next, click the other object that looks like a **Server Stack**, highlighted in Red. This will open up the list of hosts on your network. 

If you do not see the host you expect, press the **Scan for Hosts Magnifying Glass** again. 



On my second scan it picked up a host it missed on the initial scan. 



After clicking on the **Server Rack icon**, you will see the Hosts List.

### Targets

On a traditional ARP Poisoning MITM attack, we would want to sniff between the Router (typically XXX.XXX.XXX.1) in the network structure and the target device. Because the IoT Controller is on the same network rather than a Cloud service, we will want to sniff between them

**To add a Target to Target 1 or 2, highlight/click it in the list and then click “Add to Target X”**

#### Add to Target 1

We will add the **Door Lock** at IP **192.168.1.70** to target 1

#### Add to Target 2

We will add the **IoT Controller** at IP **192.168.1.72** to target 2





Click the **Globe Icon** to open the **MITM Menu**. You can also **stop** the MITM attack using the **Stop button** to the right of it.



In the MITM Menu, select the first option, ARP Poisoning.



### Optional Parameters Explained:

#### Sniff remote connections:

this will enable Ettercap to sniff connections between the target devices and the gateway
in simple terms, it will sniff internet traffic with this enabled, in addition to local network traffic
Disabling this will only sniff traffic between the two victim machines

#### Only poison one-way:

this will only poison the ARP cache between the two clients, rather than also poisoning the router ARP cache
this is helpful to evade detection in case there is an ARP watcher at the router level

**Press OK to start the MITM Attack. Press the Stop button mentioned previously to stop the attack.**

## Wireshark

Wireshark is a network packet capture tool. We will use this tool to capture the traffic we are directing through our computer using our ARP Poisoning MITM attack. Because we are sitting in the middle, we can see all traffic between the two computers - and sometimes web activity too, depending on our settings.



Click the Kali Application Menu icon at the Top Left of your screen, and search for Wireshark.
You may need to enter your admin password (default password:kali)

Within Wireshark, press the **Blue Shark Fin button** to start capturing your packets.

Press the **Stop button** to the right of your Capture button to stop captures.

### Searching Through Wireshark Capture

To find the traffic that opened the door lock, we will search through the packets.

To use the search tool, on the top menu use Edit > Find Packet.

We can try searching for POST, if we believe that the Controller is using a REST call to activate the door lock. Otherwise, we can simply search for lock and see what comes up in the traffic.

In the search area that has appeared, we will want to search for Packet Details, and search for a String.


The search bar highlights in green when a match is found.



Within the captured packets, we can see the POST call to the Door Lock from the Controller, and below it we can see the ACK (acknowledge) response from the door lock.

Once we look in the section below where the Packet Details are, we can see the full request URL.



Above in the captured packets list, if we click on the captured packet in question, we can also see the Open command and the Length (number of times it will open).



### Reconstructing The Door Lock Command

We can either use Postman or the cURL utility on the command line to send the door unlock command. 

We know the command is JSON structure, we know it is a POST call, and we know the actual command. With that, we can reconstruct the command in this syntax:

`curl --header "Content-Type: application/json" --request POST --data '{ "command": "open", "length": 5}' http://192.168.1.70:8000/lock/command/`

**We can now use this command to open the door lock, and we’re in!**

# Video

add a video link here 
