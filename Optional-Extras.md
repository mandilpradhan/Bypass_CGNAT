If you want to maintain a more hands off style of administration on your VPS, you can enable unattended upgrades.  Just as the name sounds, this will automatically install security upgrades on your VPS.

Steps:
1. `sudo apt install unattended-upgrades`
2. `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades`
   1. Uncomment the line that contains `"${distro_id}:${distro_codename}-updates";` ***should be near the top of the file***
   2. If you want to be emailed when a package is upgraded, or if an error occurs uncomment `Unattended-Upgrade::Mail "your@email.com";` and `Unattended-Upgrade::MailReport "only-on-error";`.  Change the `"only-on-error"` as appropriate.
   3. There are other settings you can change if you like.  For example, you can have the system automatically reboot at a specific time if an update requires it.  Look through the `50unattended-upgrades` file to see what you can enable.
3. `sudo nano /etc/apt/apt.conf.d/20auto-upgrades` and replace anything that is already there with:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```
Those numbers specifiy the number of days between each update/autoclean/download attempt.  Feel free to change as appropriate.

Unattended-upgrades is now installed and configured on your system.

If you would like to test to see if it is working, run:
`sudo unattended-upgrades --dry-run --debug`
You may see a bunch of regexp, but look near the bottom.  There should be a line stating that you have packages that can be upgraded or all your packages are up to date.

Example:
```
Packages blacklist due to conffile prompts: []
No packages found that can be upgraded unattended and no pending auto-removals
The list of kept packages can't be calculated in dry-run mode.
```
You can also check your log files (after a couple days) by running `sudo cat /var/log/unattended-upgrades/unattended-upgrades.log`.