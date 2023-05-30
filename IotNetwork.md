layout: page
title: "Additional Router for IoT Network"
permalink: /IotNetwork

# Background
I spent many hours trying to figure out how to create a separate IoT network that still allows devices on the secure network to find and initiate communication with devices on the IoT network, but prevents devices on the IoT network from accessing devices on the secure network. After a lot of searching I was thinking this could only be done with a router or 3rd-party router firmware that offers VLan capabilities. Even my RT-AX86U Pro offers VLan capabilities but there was no configuration that allows one-way communication with another network.

I finally came across a buried forum post with a solution that should have been obvious: just use an additional router.

Credit to the [original post](https://www.snbforums.com/threads/cascade-two-routers-for-iot-devices-and-home-network-questions.48430/).

**Goal:** You want a separate network for your IoT devices that prevents the devices from finding or initiating communication with devices on your secure network, but devices on your secure network can still find and initiate communication with the IoT devices.

**Solution:** Use an additional router for your secure network. Connect the additional router to the main router that has internet access and all the IoT devices. The standard firewall of the additional router will prevent IoT devices from initiating communication with its devices, but will still allow its devices to initiate communication with the IoT devices.

Read on for more details.

# Explanation
All modern routers should come with a built-in firewall enabled by default. The primary purpose of this firewall is to prevent devices fom the internet from discovering or initiating communication with devices on your network. If an unsolicited packet comes in from the internet, the router just ignores it. However if a device on your network is the one to initiate communication with a device on the internet, the firewall will then allow packets from that internet device.

We can use this feature to create a separate network that can still communicate with and discover devices on an IoT network. We leverage the standard firewall capabilities of the router, but instead of the firewall protecting its devices from the internet, it’s protecting its devices from the IoT network. If a device on the IoT network is compromised, there is some security to prevent it from discovering and communicating with a device on the secure network.

# Setup
Setup is fairly easy by networking standards. Modern routers should come with the correct settings by default. The trickiest part is making sure the additional router has a different local IP from the IoT router, and that its DHCP server is assigning a different range of IPs.

1. Set up a router to be the IoT router.
   - Make sure its firewall is enabled.
   - Connect its WAN port to the internet modem.
   - All IoT devices should connect to this router.
2. Set up an additional router to be the secure router.
   - Make sure its firewall is enabled.
   - In its LAN settings, make sure it has a different IP address from the IoT router. My IoT router uses `192.168.1.1` and my secure router uses `192.168.2.1`
   - In its LAN DHCP Server settings, make sure its IP pool address range is different from the IoT router’s address range. My IoT router has the range `192.168.1.2` to `192.168.1.254`, while my secure router has the range `192.168.2.2` to `192.168.2.254`
   - Connect its WAN port to a LAN port on the IoT router
   - All secure devices should connect to this router

That should be all that’s needed. The standard firewall on the IoT router blocks unsolicited packets coming from the internet. The standard firewall on the secure router blocks unsolicited packets coming from the IoT network.

# Caveats
There are a few things to watch out for with this setup

## Security
This is technically not as secure as having IoT devices on a completely separate network. If an IoT device is compromised it can’t initiate communication with a secure device, but if the secure device intentionally or accidentally initiates communication with the IoT device then there could be a security hole. To reduce this risk, we can create a 3rd network that is completely isolated from both the secure network and the IoT network.

Most modern routers come with the ability to create a separate “guest” network, and there should be an option to prevent devices on this network from accessing the local network, and even an option to prevent devices on this network from accessing each other. They are only allowed to communicate with the internet. Any IoT devices that don’t need local network access (they only need internet access) should be put on this isolated guest network. Devices such as a Ring Doorbell or Ecobee Thermostat only need internet access, so they would be good candidates to go on this isolated network.

## Speed
This setup can be referred to as “cascading routers”. Some suggest you can just use an old cheap router for the IoT router. However you do need to be careful that this router doesn’t become a bottleneck for your internet. For example, my secure network was using a router that supports gigabit speeds, but initially my IoT router only supported up to a 100 megabit internet connection, which bottlenecked my computers to 100 megabit internet. So make sure the IoT router is able to support your max internet speed.

## Nat traversal
This setup is also referred to as a “double NAT”. It basically means that if you are hosting a server on your secure network (such as when playing some online games), it will be that much harder for internet devices (other players) to connect to you. I personally see this as a minor issue as most modern games should have decent NAT punch-through or relay servers.

# Conclusion
The ideal solution for an IoT network would be a router with good software for customizing VLans and firewall rules. However it’s hard to beat the simplicity of just using an additional router. If you’re an enthusiast with many IoT devices and are considering a separate IoT network then odds are you probably already have a spare router laying around.
