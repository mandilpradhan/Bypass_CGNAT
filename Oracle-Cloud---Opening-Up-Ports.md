# Oracle Cloud - Opening Up Ports
Oracle Cloud blocks most traffic into your VM by default.  In order for traffic to be able to access your VM, follow these steps.
1. Log in to your [Oracle Cloud Account](https://cloud.oracle.com/compute/instances) and access your VM's Instance Page.
2. Select the Subnet link in the Primary VNIC section.
[![Instance](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/instance_01_arrow_s.png)](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/instance_01_arrow.png)
3. Select your Default Security list
[![Security List](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_01_arrow_s.png)](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_01_arrow.png)
4. Click on "Add Ingress Rules"
[![Ingress Rules](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_02_arrow_s.png)](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_02_arrow.png)
5. Add the ports you want to open (this example opens port 55108/UDP)
[![Open ports](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_03_s.png)](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_03.png)
6. Do that for every port/protocol you want to allow through your VPN.  If you rather manage access using your own firewall rules (like ufw), you can allow all ports/protocols in by using the following example.  (I recommend this approach, since the Oracle Ubuntu VM already blocks all traffic by default.  We can manage the traffic from the OS, not from Oracle's System)
[![Open ports](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_04_s.png)](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_04.png)
7. You can verify that the rules have been added.
[![Open ports](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_05_circ.png)](https://github.com/mochman/Bypass_CGNAT/raw/oracle/Oracle%20Cloud/images/firewall_05_circ.png)
8. You have opened up the ports from the Oracle Cloud Side.  If you are manually installing Wireguard, please continue here.