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

### Normal Behavior of Door Lock System

End user swipes card on the Card Reader. Card Reader communicates with Controller through the network with card details. If it matches the database, Controller sends unlock command to Door Lock. 

### Anomalous Behavior of System

After entering the network, we have identified devices on the network named Controller and Door Lock. We identify the traffic from the Controller to the Door Lock, and can either reconstruct the request and re send it, or actually re send the captured packet itself.  

## Techniques
