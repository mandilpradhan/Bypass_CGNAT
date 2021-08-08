# Updating an already running configuration with the Installer Script
The installer script checks to see if there is an existing wireguard configuration on either the VPS or the Local Client.
In order to re-configure the wireguard port settings, run:
```
wget "https://raw.githubusercontent.com/mochman/Bypass_CGNAT/main/Oracle%20Cloud/Oracle_Installer.sh"
chmod 755 Oracle_Installer.sh
./Oracle_Installer.sh
```
You will be presented with a couple options.
If you run the script on the VPS, your choices are:
1. Change the ports being passed through to the Client
2. Create a new wireguard configuration (Clears old config)
3. Reload wireguard service

If the script is run on the Local Client, your options are:
1. Change Port->IP Mapping
2. Reload wireguard service

### Example:
You have added a new service on your local network.  The service resides on the same Computer/VM that the wireguard client is on.  The service runs on port `12345` and uses `UDP` as the protocol.

1. On the VPS, run:
```
wget "https://raw.githubusercontent.com/mochman/Bypass_CGNAT/main/Oracle%20Cloud/Oracle_Installer.sh"
chmod 755 Oracle_Installer.sh
./Oracle_Installer.sh
```
2. Choose option 1 when prompted.
3. Update the list of ports/protocols to include your other ports and the new `12345/udp`
4. The script will provide you with a command to run on the **Local Client**.  Run that command and follow the directions.
5. Back in the VPS, press any key to continue to the next section.
6. The firewall will update to allow the new port(s) through.
