While figuring all this out, I needed to check if traffic was actually getting to my Local Server through the VPN.  I used python's http.server for this.  It sets up a simple web server that you can set to any open port.

I was able to run it from the command line: `sudo python3 -m http.server 443`

Looking back on it though, running that command allows anyone that happens to type in https://YOUR_VPS_IP/ with a page that lists all the files in your directory.  So a more safe test would be to run it in it's own folder with a default index.html page like so:
```bash
mkdir temp_dir
cd temp_dir
echo "Working" > index.html
sudo python3 -m http.server PORT_YOU_WANT_TO_TEST
```
With that web server running, you should be able to go to http://VPS_IP:PORT_YOU_WANT_TO_TEST and see a page that says "Working".  A timeout means that there is some issue with traffic flowing from your VPS to your Local server.  If you see the webpage, you know that the traffic is getting from the VPS to your Local server.  That will at least narrow down your troubleshooting search area.

**Note 1: Make sure the PORT you put in is not being used on your local server already.**

**Note 2: You will need to comment out the PostUp in your local server's wg0.config file and run a `sudo systemctl restart wg-quick@wg0` in order to make sure the iptables rules have been cleared.**