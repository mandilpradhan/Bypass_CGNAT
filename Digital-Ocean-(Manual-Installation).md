## Contents
1. [VPS Setup](#1-vps-setup)
   1. [Locking down your server](#1a-locking-down-your-server)
   2. [System config](#1b-system-config)
   3. [Installing Wireguard](#1c-installing-wireguard)
2. [Home Server Setup](#2-home-server-setup)
   1. [System Config](#2a-system-config)
   2. [Installing Wireguard](#2b-installing-wireguard)
3. [Starting Wireguard](#3-starting-wireguard)

Here is a basic diagram of my configuration.  The IPs and ports will need to be changed by you to meet your requirements.

![Topology](https://github.com/mochman/Bypass_CGNAT/raw/main/Basic%20Topology.png)

For reference, here are all the IPs, Ports, and Names that I will be using in this guide for you to reference and change as appropriate.

Name | IP used in tutorial | Port | Description
------------ | ------------- | ------------- | -------------
VPS IP | 1.2.3.4 | N/A | Your VPS's IP Address (Assigned to you)
VPS Wireguard IP | 10.0.0.1 | 55107 | The Wireguard IP:Port we will set up in our VPN connection (Created by you)
Nginx IP | 192.168.2.5 | 443 | The Local IP Address of our NPM Server (Should already exist in your local network)
Nginx Wireguard IP | 10.0.0.2 | N/A | The Wireguard IP Address we will use to talk with the VPS Server (Created by you)
Home Assistant IP | 192.168.2.6 | 1234 | The IP:Port of another service that NPM doesn't provide routing for (Should already exist / Home Assistant is just an example)
Synology NAS | 192.168.2.4 | 5001 | The IP:Port of another service that NPM doesn't provide routing for (Should already exist / Synology NAS is just an example)
Docker Server App | 192.168.2.7 | 1194 | The IP:Port of another service that NPM doesn't provide routing for (Should already exist / OpenVPN is an example)

# 1. VPS Setup
***This tutorial will assume you are running Ubuntu 20.04 on both your VPS and Local Server.***

## 1a. Locking down your server
I recommend following a system hardening guide like [this one](https://www.digitalocean.com/community/tutorials/how-to-harden-openssh-on-ubuntu-18-04), [this one](https://medium.com/@jasonrigden/hardening-ssh-1bcb99cd4cef), or [this one](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server).  After this, I will assume you have kept sshd running on port 22.  If you changed the port, pay attention in the following steps and adjust as appropriate.
## 1b. System config
Enable forwarding by running:
```bash
sudo nano /etc/sysctl.conf
```
Make sure `net.ipv4.ip_forward=1` is not commented, save the file then run:
```bash
sudo sysctl -p
```
## 1c. Installing Wireguard
After a `sudo apt update && sudo apt upgrade` run:
```bash
sudo apt install wireguard
umask 077 && printf "[Interface]\nPrivateKey = " | sudo tee /etc/wireguard/wg0.conf > /dev/null
sudo wg genkey | sudo tee -a /etc/wireguard/wg0.conf | wg pubkey | sudo tee /etc/wireguard/publickey
```
Those commands will install wireguard, create a file in `/etc/wireguard/wg0.conf`, place a generated private key into that file.  Then it prints out a public key **that you need to keep** (if you forget it, the public key is also in the `/etc/wireguard/publickey` file).
Now open the wireguard configuration file.
```bash
sudo nano /etc/wireguard/wg0.conf
```

### Traffic forwarding choices:
You have 2 options on how to forward traffic.  You can forward everything through, or you can forward explicit ports.

## a. Forward everything:
If you use this option, iptables will be set up to forward all traffic (except ports 22 and your wireguard port) through to your local server.  This means that all port restricting/firewalling will need to be done on your local server.


### Use the following config:
**Things you need to change:**
Name | Item | Description
--- | --- | ---
*VPS IP* | 1.2.3.4 | The IP Address of your VPS
*interface* | eth0 | Your internet facing interface.

**Things you can change:**

Name | Item | Description
--- | --- | ---
*Wireguard Port* | 55107 | Any unused port you like
*Wireguard Server IP* | 10.0.0.1/24 | Any RFC1918 IP/CIDR.  Don't you your home network's IPs (192.168.2.0/24 in this tutorial).
*Wireguard Host IP* | 10.0.0.2 | Same as above, make sure it's in the same address range.
*Wireguard Host IP/32* | 10.0.0.2/32 | The above IP Address with /32 after it.
```
[Interface]
PrivateKey = SHOULD_ALREADY_BE_FILLED_OUT
ListenPort = 55107
Address = 10.0.0.1/24

PostUp = iptables -t nat -A PREROUTING -p tcp -i eth0 '!' --dport 22 -j DNAT --to-destination 10.0.0.2; iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 1.2.3.4
PostUp = iptables -t nat -A PREROUTING -p udp -i eth0 '!' --dport 55107 -j DNAT --to-destination 10.0.0.2;

PostDown = iptables -t nat -D PREROUTING -p tcp -i eth0 '!' --dport 22 -j DNAT --to-destination 10.0.0.2; iptables -t nat -D POSTROUTING -o eth0 -j SNAT --to-source 1.2.3.4
PostDown = iptables -t nat -D PREROUTING -p udp -i eth0 '!' --dport 55107 -j DNAT --to-destination 10.0.0.2;

[Peer]
PublicKey = 
AllowedIPs = 10.0.0.2/32
```

## b. Forward specific ports:
If you use this option, iptables will be set up to only forward the ports that you want through the VPN.  This is a more secure setup, since it opens your local server to less.  All port blocking/firewalling will need to be done on your VPS. 


### Use the following config:
**Things you need to change:**
Name | Item | Description
--- | --- | ---
*VPS IP* | 1.2.3.4 | The IP Address of your VPS
*interface* | eth0 | Your internet facing interface.
*TCP Ports* | 443,8443,5001 | a list of TCP ports to pass through the VPN.
*UDP Ports* | 51820 | a list of UDP ports to pass through the VPN.

**Note:** If you aren't going to forward any TCP or UDP ports, the respective *PostUp* and *PostDown* lines can be excluded from the following config.

**Things you can change:**

Name | Item | Description
--- | --- | ---
*Wireguard Port* | 55107 | Any unused port you like
*Wireguard Server IP* | 10.0.0.1/24 | Any RFC1918 IP/CIDR.  Don't you your home network's IPs (192.168.2.0/24 in this tutorial).
*Wireguard Host IP* | 10.0.0.2 | Same as above, make sure it's in the same address range.
*Wireguard Host IP/32* | 10.0.0.2/32 | The above IP Address with /32 after it.
```
[Interface]
PrivateKey = SHOULD_ALREADY_BE_FILLED_OUT
ListenPort = 55107
Address = 10.0.0.1/24

PostUp = iptables -t nat -A PREROUTING -p tcp -i eth0 --match multiport --dports 443,8443,5001 -j DNAT --to-destination 10.0.0.2
PostUp = iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 1.2.3.4
PostUp = iptables -t nat -A PREROUTING -p udp -i eth0 --match multiport --dports 51820 -j DNAT --to-destination 10.0.0.2

PostUp = iptables -t nat -D PREROUTING -p tcp -i eth0 --match multiport --dports 443,8443,5001 -j DNAT --to-destination 10.0.0.2
PostUp = iptables -t nat -D POSTROUTING -o eth0 -j SNAT --to-source 1.2.3.4
PostUp = iptables -t nat -D PREROUTING -p udp -i eth0 --match multiport --dports 51820 -j DNAT --to-destination 10.0.0.2

[Peer]
PublicKey = 
AllowedIPs = 10.0.0.2/32
```


We will fill in the PublicKey section after we install Wireguard on our local server.

For your inforamtion, the PostUp and PostDown commands will run when wireguard makes/loses connection.
The first two PostUp commands will forward the assigned TCP traffic through the wireguard VPN to our server without changing any of the incomming IP addresses.
The third PostUp command will do the same with UDP traffic.
The PostDown commands just remove what was created with the PostUp commands.

# 2. Home Server Setup
## 2a. System config
Enable forwarding by running:
```bash
sudo nano /etc/sysctl.conf
```
Make sure `net.ipv4.ip_forward=1` is uncommented, save the file then run:
```bash
sudo sysctl -p
```
## 2b. Installing Wireguard
We're going to do the same installation steps as we did on the VPS.
```bash
sudo apt install wireguard
sudo (umask 077 && printf "[Interface]\nPrivateKey = " | sudo tee /etc/wireguard/wg0.conf > /dev/null)
sudo wg genkey | sudo tee -a /etc/wireguard/wg0.conf | wg pubkey | sudo tee /etc/wireguard/publickey
```
**Take this public key and place it in the `PublicKey = ` section on the VPS's `/etc/wireguard/wg0.conf` file.**
Now open the wireguard configuration file.
```bash
sudo nano /etc/wireguard/wg0.conf
```
### Use the following config:
**Things you need to change:**
Name | Item | Description
--- | --- | ---
*PublicKey* | THE_PUBLIC_KEY_FROM_YOUR_VPS_WIREGUARD_INSTALL | The public key you copied when installing wireguard on the **VPS**.

**Things you may have to change:**

Name | Item | Description
--- | --- | ---
*Wireguard Port* | 55107 | The port you used in the VPS config
*Wireguard Host IP* | 10.0.0.2/24 | The Host IP you used in the VPS config with a /24 after it
```
[Interface]
PrivateKey = SHOULD_ALREADY_BE_FILLED_OUT
Address = 10.0.0.2/24

PostUp = iptables -t nat -A PREROUTING -p tcp --dport 1234 -j DNAT --to-destination 192.168.2.6:1234; iptables -t nat -A POSTROUTING -p tcp --dport 1234 -j MASQUERADE
PostUp = iptables -t nat -A PREROUTING -p tcp --dport 5001 -j DNAT --to-destination 192.168.2.4:5001; iptables -t nat -A POSTROUTING -p tcp --dport 5001 -j MASQUERADE
PostUp = iptables -t nat -A PREROUTING -p udp --dport 1194 -j DNAT --to-destination 192.168.2.7:1194; iptables -t nat -A POSTROUTING -p udp --dport 1194 -j MASQUERADE

PostDown = iptables -t nat -D PREROUTING -p tcp --dport 1234 -j DNAT --to-destination 192.168.2.6:1234; iptables -t nat -D POSTROUTING -p tcp --dport 1234 -j MASQUERADE
PostDown = iptables -t nat -D PREROUTING -p tcp --dport 5001 -j DNAT --to-destination 192.168.2.4:5001; iptables -t nat -D POSTROUTING -p tcp --dport 5001 -j MASQUERADE
PostDown = iptables -t nat -D PREROUTING -p udp --dport 1194 -j DNAT --to-destination 192.168.2.7:1194; iptables -t nat -D POSTROUTING -p udp --dport 1194 -j MASQUERADE

[Peer]
PublicKey = THE_PUBLIC_KEY_FROM_YOUR_VPS_WIREGUARD_INSTALL
AllowedIPs = 0.0.0.0/0
Endpoint = 1.2.3.4:55107
PersistentKeepalive = 25
```
If all of your traffic just needs to be routed to NPM, then you can delete all of the PostUp and PostDown lines.
Otherwise, you will **need to edit the PostUp and PostDown lines** to suit your needs.  Here is an explanation of the ones I have provided:

Lets say you have Home Assistant running on port 1234 on a different server (IP 192.168.2.6).  You also have a Synology (port 5001) running on a server with the IP 192.168.2.4.  Finally you have an OpenVPN server (port 1194 UDP) running on 192.168.2.7.
 * The first PostUp command will route all (tcp) traffic coming through the VPN on port 1234 to 192.168.2.6
 * The second PostUp command will route all (tcp) traffic from the VPN on port 5001 to 192.168.2.4
 * The third PostUp command will route all (udp) traffic from the VPN on port 1194 to 192.168.2.7

The postDown commands are exactly the same as the PostUp couterparts except that '-A' becomes '-D'.
If you have more services you want to forward traffic to, just add another PostUp command and change the IP address and port as appropriate.  Don't forget to add the similar PostDown command.

# 3. Starting Wireguard
On both the VPS and Local Server, run:
```bash
sudo systemctl start wg-quick@wg0
```
After you have run both of those commands, test your connection from the VPS:
```bash
ping 10.0.0.2
```
You should see the ping replies.  If you don't please make sure you followed all of the steps and have not received any errors during the installation processes.  Once you have a good connection, run:
```bash
sudo systemctl enable wg-quick@wg0
```
on both machines to ensure that wireguard automatically starts.

## Please continue with [Limiting Access](Digital-Ocean-(Limiting-Access))
