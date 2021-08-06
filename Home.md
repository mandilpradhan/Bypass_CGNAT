# Bypassing a CGNAT with Wireguard
## Overview
Before switching ISPs, I had a public IP that allowed me to use port forwarding on my router to pass traffic to services hosted on my internal network.  My new ISP uses a CGNAT, so I had to find a workaround.  I chose this path, because it keeps pretty much everything the same for my services.  The main things I wanted to do with my setup were:
* Forward only specific traffic from the internet to my services
* Provide my NPM (Nginx Proxy Manager) Server with clients real IPs (for fail2ban blocking purposes)
* Allow for traffic to flow to internal services that NPM doesn't manage

I went through a couple configurations and VPS providers before I created this solution.  Prior to attempting this, I had little to no knowledge about VPS providers, wireguard, ufw, and iptables.  Getting it to work the way I wanted took a few days of research, trial, and error.
This will hopefully be a useful tutorial for people who are in a similar situation.  
This tutorial assumes you have some basic knowledge about how to use Ubuntu from the command line.

Here is a basic diagram of my configuration.  The IPs and ports will need to be changed by you to meet your requirements.

![Topology](https://github.com/mochman/Bypass_CGNAT/raw/main/Basic%20Topology.png)


If this is something you want to try out for yourself, please follow along with this guide.  Right now, here are the providers I have tested out that I know are capable of routing traffic this way.

1. [Digital Ocean](https://www.digitalocean.com/) - The cheapest is ~$5 a month (as of Mar 2021)
   * [Creating the VM](Digital-Ocean-(Creating))
   * [Installing Wireguard - Automated](Digital-Ocean-(Automatic-Installer-Script))
   * [Installing Wireguard - Manual](Digital-Ocean-(Manual-Installation))
2. [Oracle Cloud](https://www.oracle.com/cloud/) - The cheapest is free (as of Mar 2021)
   * [Creating the VM](Oracle-Cloud-(Creating))
   * [Opening up Oracle Cloud Ports](Oracle-Cloud--(Opening-Up-Ports))
   * [Installing Wireguard - Automated](Oracle-Cloud-(Automatic-Installer-Script))
   * [Installing Wireguard - Manual](Oracle-Cloud-(Manual-Installation))
3. [AWS Lightsail](https://aws.amazon.com/lightsail/) - The cheapest is free (as of Aug 2021)
   * [Creating the VM](AWS-Lightsail-(Creating))
   * [Installing Wireguard - Automated](AWS-Lightsail-(Automatic-Installer-Script))
   * [Installing Wireguard - Manual](AWS-Lightsail-(Manual-Installation))

Select "Creating the VM" above for the VPS you would like to use and follow the guide.
