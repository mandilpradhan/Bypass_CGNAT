If you want to Manually install Wireguard and configure all the iptables rules, please go [here](Oracle-Cloud-(Manual-Installation)).

# Using the automation script
Before you start this process, make sure you have:
1. Created a Oracle Cloud VM as described in [Creating](Oracle-Cloud-(Creating)).
1. Opened up all the necessary ports as described in [Opening Up Ports](Oracle-Cloud--(Opening-Up-Ports)).
2. Have your Local Server up and connected to the internet.

## Steps
1. On your VPS, run
```
sudo -i
wget "https://raw.githubusercontent.com/mochman/Bypass_CGNAT/main/Oracle%20Cloud/Oracle_Installer.sh" && chmod 755 Oracle_Installer.sh
./Oracle_Installer.sh
```
2. The installer will walk you through the installation process.  The first couple things it does is update/upgrades all your software.  This make take a few minutes.
3. At a minimum, you will need to know your VPS Public IP and what services(Ports & Protocols) you want to allow through.

**Example 1:** Lets say your Local Server has Nginx Proxy Manager (NPM) installed on it, running on port 443.  When prompted on the script, type in `443/tcp`

**Example 2:** Lets say you are also want to pass through traffic to Home Assistant (port 6432) on a different server on your network that NPM doesn't reverse proxy for.  Your input would be `443/tcp,6432/tcp` (passing through both services).

4. Once you have entered all your IP, Port, and Protocol information. The script will provide you with a command to execute on your Local Server.  It will wait for you to enter a public key that will be provided to you when you run the script on your Local Server.

5. On your Local Server, run
```
wget "https://raw.githubusercontent.com/mochman/Bypass_CGNAT/main/Oracle%20Cloud/Oracle_Installer.sh" && chmod 755 Oracle_Installer.sh
<COMMAND PROVIDED TO YOU FROM VPS>
```
6. If you added services to pass through, the script will ask you where those services point to.  If those services are hosted on the same Local Server you are running the script on, just press enter when prompted.  Otherwise you will need to provide the IP address of the device that is hosting the service.  

**Example 1:** Lets say your Local Server has Nginx Proxy Manager (NPM) installed on it, running on port 443.  When you are prompted for the IP of 443/tcp, just press enter. 

**Example 2:** Lets say you are running Home Assistant (port 6432) on a different server on your network that NPM doesn't reverse proxy for.  When you are prompted for the IP of 6432/tcp, enter your Home Assistant's IP address.

7. After all of that information is input, the script will give you a public key to copy and paste back into the VPS script.  Do so.

8. After inputting the public key in the VPS, the wireguard service will be started and both servers will try to ping each other over the VPN to see if a connection is established.  You should hopefully be told a connection has been established.

9. After a connection has been established, the wireguard service will be enabled so it will automatically start on boot.

## Continue with [Limiting Access](Oracle-Cloud-(Limiting-Access))