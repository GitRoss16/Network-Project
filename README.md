# Network-Project
 ###Ths is a project where I created a network for a small business###
We began with a pre-configured WAN-Cloud and WAN-Switch in GNS3. We added a FortiGate firewall, two additional switches, and a Windows 10 workstation to the network.
![Phase11](https://github.com/GitRoss16/Network-Project/assets/144251501/a7831216-744f-42a8-be64-eb6275308899)

Next we set up a Virtual LAN interface using PuTTY.
To do this we used the following commands in the CLI.
1. conf sys int 
2. edit port 2 
3. Configure access for ping, http, https and ssh.
4. Set the IP address to 10.123.0.1/24
5. Exit
6. Verified the configuration of port2, "show sys int port2" was used.
<put pic here>

After this we configured a DHCP server for the LAN interface.
1. By using "conf sys dhcp server", we were able to access the DHCP Server settings and set the deafult gateway to 10.128.0.1. and the subnet mask to 255.255.255.0.
2. We assigned the interface to port2 and defined the IP range.
3. Specified the start and end IP addresses.
4. Exited the configuration.
5. Verified the DCHP server settings by using "show sys dchp sever 1" in the terminal.
   <Picture here>

   The next step was to configure the Windows 10 workstation and confirm the DCHP lease.
1. Exectued the "ipconfig /all" command in the terminal.
2. Verified that we had a valid IP range of 10.128.0.[100-199]/24, a gateway address of 10.128.0.1, and a DHCP address of 
   10.128.0.1.

   We then began to run some test's to check our network connectivity. To do this, we ran a simple "ping" command to the 
   different servers.
 1. ping 10.128.0.1 (ping to the LAN that was successful).
 2. ping 8.8.8.8 (ping to the WAN that failed).
 3. ping google.com (ping to the DNS that failed).

Using pre-configured instructions, we accessed the GUI via the Windows 10 browser to make system changes to the dashboard. 
1. Updated hostname.
2. Timezone.
3. Local NTP server was enabled.
4. Interfaces were listed.
5. Idle timeout was set.
6. Auto file system check was enabled.

   Once these changes were made, we performed a backup of the configuration and rebooted the firewall.

   
