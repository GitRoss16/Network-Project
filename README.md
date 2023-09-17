# Network-Project
## Ths is a project where I created a network for a small business ##
We began with a pre-configured WAN-Cloud and WAN-Switch in GNS3. We added a FortiGate firewall, two additional switches, and a Windows 10 workstation to the network
![Screenshot 2023-09-16 205815](https://github.com/GitRoss16/Network-Project/assets/144251501/d351e4df-58bc-4bc6-803d-a3c035cef604)

Next we set up a Virtual LAN interface using PuTTY.
To do this we used the following commands in the CLI.
conf sys int 
edit port 2 
Configure access for ping, http, https and ssh.
Set the IP address to 10.123.0.1/24
Exit
To verify the configuration of port2, "show sys int port2" was used. 
<put pic here>
