All requests coming into your VPS will be sent to your home server.  

Lets say you only have services running on 443(https for nginx), 1234(Home Assistant), 5001(Synology), and 1194 UDP(OpenVPN).  There is no reason for any other port to be open on your VPS (except 22, and 55107).  So we will use ufw to block all other access.
On your VPS, run the following commands (using your ports):

**Note: If you aren't using 22 as the sshd port, make sure you change the first line to match your port.  If you don't; as soon as you enable ufw, you will be locked out of your VPS**
```bash
sudo ufw allow OpenSSH
sudo ufw allow 55107
sudo ufw allow 443/tcp
sudo ufw allow 1234/tcp
sudo ufw allow 5001/tcp
sudo ufw allow 1194/udp
sudo ufw default allow routed
sudo ufw default deny incoming
sudo ufw enable
```

On your Local Server, if you have ufw enabled, make sure you open up the same ports as you have on your VPS.  Also be sure to run ```sudo default allow routed```.  If ufw is disabled/not installed, then you don't need to worry.

I recommend installing fail2ban on both your Local Server and your VPS.  The VPS fail2ban will handle your ssh and the one on the local server can handle the others.  That's why I set it up so that all the Original IP addresses are sent through the VPN, so I can still block them.

**Original IP Address Limitations**
While all the traffic coming in to your local server has the original IPs intact, the traffice that is forwarded to our other services via the PostUp iptables command (i.e. Home Assistant, Synology, ...) will have their IPs look like they're coming from your local NPM server.  This wasn't a problem for me since I don't run fail2ban on those extra services.