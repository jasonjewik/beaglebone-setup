# BeagleBone Setup

I had a BeagleBone Green Wireless left over from a class I took at UCLA and had this really sick idea to serve my own website from the thing and also maybe have it be solar-powered. I ultimately gave up because I don't have a solar panel capable of powering my BeagleBone and also self-hosting a website seems hard. Anyway, this document is to remind me how I setup the BeagleBone in case I ever want to go through that again.

## Steps

1. Download [this image](https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz) for the Beaglebone Green Wireless (BBGW). Other images can be found on the [BeagleBoard website](https://beagleboard.org/latest-images).

2. Use [Balena Etcher](https://www.balena.io/etcher/) to flash the image onto your microSD card. 

3. Plug the microSD into the BBGW, then plug the BBGW into your computer with a USB-to-microUSB cable. Before plugging into the computer, hold down the "User" button. Wait until the BBGW shows up on your computer.

4. Open up the BBGW in your file explorer and check out the `START.htm` file. It'd behoove you to read this.

5. Log into the BBGW with `ssh debian@192.168.7.2`. A warning will be issued. Enter the command it tells you to enter and then try again. The login password is shown at the password prompt (should be `temppwd`). The password for `su` is `root`. Same for logging in with the root user.

6. Check if your BBGW has internet connectivity with `ping 8.8.8.8`. It probably will say "network unreachable".

7. Run `sudo wpa_cli`. Type `scan` then `scan_results` to see all WiFi networks in range. Note the SSID of the network you want to connect to. 

8. Select interface `wlan0` by typing `interface wlan0`, then type `add_network`. 

9. Type `set_network 0 ssid "SSID"` then `set_network 0 psk "PASSPHRASE"`. Then type `enable_network 0` followed by `quit`. At any point before quitting, you can type `list_networks` or `status` to get (different kinds of) status information.
10. You can check the status of your connection with `iwconfig` or `ifconfig`. The latter will show your ip address (as will `ip addr show wlan0`). As before, you can check your connectivity by pinging `8.8.8.8`. If it does not work, try `dhclient -r` then `dhclient wlan0`. Use `sudo` whenever an operation says "not permitted" or "not enough permission" or something like those. 

## Optional

### Auto Reconnect to Wifi

To enable auto reconnect to WiFi on device boot, do the following. 

> Note: I'd recommend using static IP so that you can just connect the BBGW to any power source within range of the WiFi access point and then SSH into it via static IP (i.e., the BBGW does not need to be connected via USB to a computer for you to work with it). If you want a static IP address, check your router to find what addresses are NOT in its DHCP range and pick one of those. You should check if your router needs any additional config to support static IP (mine did not). Your router info can typically be found at `192.168.1.1`, but in my case it was `192.168.254.254`. So, just Google it to find out the address for your router. You'll need to see your router config to fill out most of the details for auto reconnect anyway. See part 6 of this [Ask Ubuntu answer](https://askubuntu.com/questions/16584/how-to-connect-and-disconnect-to-a-network-manually-in-terminal) and also this [SuperUser post](https://superuser.com/questions/1453120/connect-to-wifi-on-boot-beagle-bone-black-ioctlsiocsiwencodeext-invalid-argum) for details.

1. Configure `/etc/network/interaces` to look like this:
  ```
  # This file describes the network interfaces available on your system
  # and how to activate them. For more information, see interfaces(5).

  # The loopback network interface
  auto lo
  iface lo inet loopback

  auto wlan0
  iface wlan0 inet static
  address 192.168.254.8
  netmask 255.255.255.0
  gateway 192.168.254.254
  wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

  # The primary network interface
  #auto eth0
  #iface eth0 inet dhcp
  # Example to keep MAC address between reboots
  #hwaddress ether DE:AD:BE:EF:CA:FE

  ##connman: ethX static config
  #connmanctl services
  #Using the appropriate ethernet service, tell connman to setup a static IP address for that service:
  #sudo connmanctl config <service> --ipv4 manual <ip_addr> <netmask> <gateway> --nameservers <dns_server>

  ##connman: WiFi
  #
  #connmanctl
  #connmanctl> tether wifi off
  #connmanctl> enable wifi
  #connmanctl> scan wifi
  #connmanctl> services
  #connmanctl> agent on
  #connmanctl> connect wifi_*_managed_psk
  #connmanctl> quit

  # /etc/wpa_supplicant/wpa_supplicant.conf
  network={
  ssid="SSID"
  scan_ssid=1
  proto=RSN
  key_mgmt=WPA-PSK
  pairwise=CCMP
  group=CCMP
  psk="PASSPHRASE"
  }
  ```

2. Add Google's nameservers by modifying `/etc/resolv.conf` to look like this:
  ```
  nameserver 8.8.8.8
  nameserver 8.8.4.4
  ```
  You can now test by pinging `google.com`. [Click here](https://developers.google.com/speed/public-dns/docs/using) to read details about Google's public DNS. Also maybe look into [1.1.1.1](https://1.1.1.1/).

3. Reboot the device with `shutdown -r now`. You will have to do this with `sudo` or as root.

4. The BBGW should still be plugged into your computer via the USB cable. Wait until you see it pop up again on your computer. Then you can try SSH'ing into `debian@192.168.7.2` again. Check if it has internet access. It might take a minute or two for the BBGW to establish a connection. 

5. I recommend updating the package list and upgrading your packages with apt update followed by apt upgrade (both run with root permissions, of course). You will need to do this to install packages anyway. For example, if you want to synchronize the time, see this [Linux Hint article](https://linuxhint.com/sync-time-ntp-server-linux/). I recommend using chrony.  

### Other Stuff

If you want to update the boot up scripts and Linux kernel, See the "Update the boot-up scripts and Linux kernel" section of this [page on the BeagleBoard site](https://beagleboard.org/upgrade#:~:text=Ethernet%20(ie.%2C%20uplink).-,Update%20the%20boot%2Dup%20scripts%20and%20Linux%20kernel,-debian%40beaglebone%3A/var). Ignore everything else on that page. 

You can change the password of the users with the `passwd` command. 

Note, if you want to plug in a peripheral device to the USB ports of the BBGW, they have to be plugged in before booting the BBGW. For example, I tried plugging an external HDD into the USB port (because I also wanted to try self-hosting cloud storage on the BBGW) after booting and the BBGW wouldn't recognize it. So just plug it in before booting. 

## Link Dump

Various links I looked at while working on this.

- https://askubuntu.com/a/833894
- https://linux.die.net/man/8/wpa_cli
- https://www.cyberciti.biz/faq/how-to-set-up-ssh-keys-on-linux-unix/
- https://github.com/vrk7bp/BeagleBoneWebServer
- https://askubuntu.com/questions/16584/how-to-connect-and-disconnect-to-a-network-manually-in-terminal
- https://unix.stackexchange.com/questions/182847/how-to-automatically-apply-wpa-supplicant-configuration
- https://gist.github.com/penguinpowernz/1d36a38af4fac4553562410e0bd8d6cf
- https://www.liquidweb.com/kb/how-to-install-and-configure-nmcli/
- https://www.tecmint.com/configure-network-connections-using-nmcli-tool-in-linux/
- https://www.makeuseof.com/connect-to-wifi-with-nmcli/
- https://unix.stackexchange.com/questions/158328/network-manager-not-listing-wifi
- https://askubuntu.com/questions/947965/how-to-trigger-network-manager-autoconnect
- https://embeddedinventor.com/a-beginners-introduction-to-linux-package-managers-apt-yum-dpkg-rpm/
- https://www.linuxquestions.org/questions/linux-wireless-networking-41/wpa-4-way-handshake-failed-843394/
- https://www.linuxbabe.com/debian/connect-to-wi-fi-from-terminal-on-debian-wpa-supplicant
- https://unix.stackexchange.com/questions/415816/i-am-trying-to-connect-to-wifi-using-wpa-cli-set-network-command-but-it-always-r
- https://beagleboard.org/p/mirkix/flying-beaglebone-green-448b60
- https://www.cyberciti.biz/faq/add-configure-set-up-static-ip-address-on-debianlinux/
- https://linuxhint.com/run-script-debian-11-boot-up/
- https://wiki.t-firefly.com/en/ROC-RK3308-CC/network_config.html
- https://beagleboard.org/upgrade
- https://ssg-drd-iot.github.io/getting-started-guides/docs/connectivity/ethernet_over_usb/linux/details-forward_usb0.html
- https://ofitselfso.com/BeagleNotes/HowToConnectPocketBeagleToTheInternetViaUSB.php
- https://superuser.com/questions/1453120/connect-to-wifi-on-boot-beagle-bone-black-ioctlsiocsiwencodeext-invalid-argum
- https://askubuntu.com/questions/942074/ubuntu-server-16-04-2-wifi-connection/942424#942424
- https://www.cyberciti.biz/faq/unix-linux-check-if-port-is-in-use-command/

See these links about updating the security source.

- https://www.debian.org/security/
- https://stackoverflow.com/questions/70789307/how-to-fix-the-following-signatures-couldnt-be-verified-because-the-public-key

Nginx stuff (for serving a website from the BBGW).

- https://jgefroh.medium.com/a-guide-to-using-nginx-for-static-websites-d96a9d034940
- http://nginx.org/en/docs/beginners_guide.html

Links for plugging an HDD into the BBGW:

- https://www.simplified.guide/linux/disk-mount
- https://linuxopsys.com/topics/linux-fstab-options
- https://unix.stackexchange.com/questions/53456/what-is-the-difference-between-nobootwait-and-nofail-in-fstab 
- https://geek-university.com/etc-fstab-file/ 
- https://askubuntu.com/questions/46588/how-to-automount-ntfs-partitions 

When I gave up on self-hosting my website, I tried to salvage this project by using compute time for BOINC. Turns out it doesn't work 😔

- https://boinc.berkeley.edu/wiki/Boinccmd_tool
- https://www.charityengine.com/community/wiki/welcome-linux-users
- https://www.worldcommunitygrid.org/
- https://boinc.berkeley.edu/projects.php
