# Network-Project
## This is a project where my team and I created a network for a small business
The client requested the network be secure, with an internal Windows domain, internal Microsoft IIS Webserver, internal Windows 10 workstation, a public webserver, public FTP server, a LAN on 10.128.0.0/24, DMZ on 10.128.10.0/24, and GUEST network on 10.128.99.0/24.
We began with a pre-configured WAN-Cloud and WAN-Switch in GNS3. 

![Stage 0](https://github.com/GitRoss16/Network-Project/assets/144251501/32f89193-c0ed-4663-8261-63e1a66f9da6)


# Stage 1: Network Setup

During this stage we added 2 ethernet switches, a FortiNet firewall (FortiGate), and configured a LAN network within it. Connected a firewall graphical user interface from the WIN10 machine, and then completed the setup of the network though that firewall GUI by connecting all of these components together. Connections are as follows.

- WAN port on firewall to WAN-SWITCH
- LAN port on firewall to LAN-SWITCH
- DMZ port on firewall to DMZ-SWITCH
- WIN10 workstation to LAN-SWITCH.
  
We set up the Virtual LAN interface using CLI. Commands as follows.

- conf sys int
- edit port2
- set allowaccess ping http https ssh
- set ip 10.128.0.1/24
- end

We then verified the configuration by using the command, "show sys int port2".

We configured the DHCP server for the LAN interface. This would allow the firewall to perform DHCP services on the LAN network.

Knowing that the DHCP pool scope would be 10.128.0.100-199, we again utilized the CLI to input commands. Commands as follows.

- conf sys dhcp server
- edit 1
- set default-gateway 10.128.0.1
- set netmask 255.255.255.0
- set interface port2
- config ip-range
- edit 1
- set start-ip 10.128.0.100
- set end-ip 10.128.0.199
- next
- end
- next
- end

We then verified the configuration by using the command, "show sys dhcp server 1". 

Our next objectice was adding the WIN10 workstation

First we verified that the WIN10 had leased a DHCP address from the LAN network. Again using CLI. 

- ipconfig /all

This showed us that we had a valid IP range of 10.128.0.[100-199]/24, a gateway of 10.128.0.1, and DHCP server at 10.128.0.1.

To test their connectivity, we used a ping test for each. Commands as follows.

- For the LAN: ping 10.128.0.1
- For the WAN: ping 8.8.8.8
- For the DNS: ping google.com

These ping test's resulted in a succesful LAN connection, but was unsuccesul for the WAN and DNS.

Connecting to the firewall GUI required us to navigate to "http://10.128.0.1/", login with our credentials, and make various changes to the FortiGate set up prompt. Changes as follows.

- hostname = firewall
- timezone = -6:00 Central Time (U.S.& Canada)
- setup device as local NTP server = enabled
- list on interfaces = port2, port4
- idle timeout = 60
- autofile system check = enabled

We then saved a backup of the firewall configuration and saved it to a folder on our WIN10 machine. After saving, we rebooted the system in order to make the changes.

 The last step within stage 1 was the longest but really started connecting all the peices together.
 Per our clients request, we had to configure the following. (All of this was done on the 'http://10/128.0.1/' webserver)

 - 10.128.0.0/24 as the LAN network
 - 10.128.99.0/24 as the GUEST network
 - 10.128.10.0/24 as the DMZ network
 
 Within the Network > Interfaces tab, the following changes were made.

 ### Port1 WAN
  
 - edit port1
 - alias = WAN
 - role = WAN
 - APPLY CHANGES
 
 ### Port2 LAN

 - edit port2
 - alias = LAN
 - role = LAN
 - create address object matching subnet = enabled
 - administrative access = https, http, ping, ssh
 - dns server = same as interface IP
 - expand adanced
 - ntp server = local
 - APPLY CHANGES
 
  ### Port3 GUEST

 - edit port3
 - alias = GUEST
 - role = LAN
 - ip/network mask = 10.128.99.1/24
 - create address object matching subnet = enabled
 - administrative access = ping, ssh
 - dhcp server = enabled
 - dns server = same as interface ip
 - expand advanced
 - ntp server = local
 - APPLY CHANGES
 
 ### Port4 DMZ

 - edit port4
 - alias = DMZ
 - role = DMZ
 - ip/network mask = 10.128.10.1/24
 - create address object matching subnet = enabled
 - administrative access = ping
 - APPLY CHANGES

 To enable the DNS, we navigated to System > Feature Visibility, and made the following changes.

 - dns database = enabled
 - APPLY CHANGES
   
 To configure the firewall system DNS settings, we nevigated to Network > DNS and made the following changes.

 ### DNS 

 - dns servers = specify
 - primary dns server = 8.8.8.8
 - secondary dns server = 1.1.1.1
 - APPLY CHANGES

 Within Network > DNS Servers...

 ### LAN DNS 
 - create new
 - interface = LAN (port2)
 - mode = recursive
 - APPLY CHANGES

 ### GUEST DNS

 - create new
 - interface = GUEST (port3)
 - mode = recursive
 - APPLY CHANGES

 ### DMZ DNS

 - create new
 - interface = DMZ (port4)
 - mode = recursive
 - APPLY CHANGES

 Within the Policy & Objects > Services tab, we configured the following service objects

 ### LAN Services

 - create new > service group
 - name = LAN-services-group
 - members = ALL_ICMP, NTP, RDP, SSH, WEB ACCESS, WINDOWS AD

 ### DMZ Services

 - create new > service group
 - name = DMZ-services-group
 - members = ALL_ICMP, FTP, RDP, SSH, WEB ACCESS

 Finally, we navigated to the Policy & Objects > IPv4 Policy tab to configure the remaining firewall rules.

 ### LAN-to-WAN policy 

 - create new 
 - name = LAN-to-WAN
 - incoming interface = LAN
 - outgoing interface = WAN
 - source = port2 address
 - destination = all
 - service = all
 - NAT = enabled
 - APPLY CHANGES

 ### DMZ-to-WAN policy

 - create new
 - name = DMZ-to-WAN
 - incoming interface = DMZ
 - outgoing interface = WAN
 - source = port4 address
 - destination = all
 - service = all
 - NAT = enabled
 - APPLY CHANGES

 ### LAN-to-DMZ policy 

 - create new 
 - name = LAN-to-DMZ
 - incoming interface = LAN
 - outgoing interface = DMZ
 - source = port2 address
 - destination = port4 address
 - service = DMZ-services-group
 - NAT = disabled
 - APPLY CHANGES

 ### DMZ-to-LAN policy

 - create new
 - name = DMZ-to-LAN
 - incoming interface = DMZ 
 - outgoing interface = LAN
 - source = port4 address 
 - destination = port2 address 
 - service = LAN-services-group
 - NAT = disabled
 - APPLY CHANGES

 ### WAN-to-DMZ policy 

 - create new
 - name = WAN-to-DMZ
 - incoming interface = WAN
 - outgoing interface = DMZ
 - source = all
 - destination = port4 address
 - service = DMZ-services-group
 - NAT = disabled
 - APPLY CHANGES

 To conclude the last steps of Stage 1, we performed another backup of the firewall configuration to ensure the application of these changes were made. After this, our network topology looked a little something like 
 this.

![Stage2 Start](https://github.com/GitRoss16/Network-Project/assets/144251501/09189b2f-63fc-4354-8f1f-07ac9af8ea9e)

# Stage 2: Domain Setup

During this stage we added more devices and linked them up, prepared a WIN2012r2 Server for the "Actice Directory Domain Services" server role and installed it, created new AD user accounts, prepared the WIN10 from stage 1 to join the domain and joined it, and added a few other small details. 

After turning the WIN2012r2 server on, connecting it, manuevering through the license agreement, we set up a a password for the admin account and logged in. Next we had to set the static IP address in Server Manager.
To get here we followed this path. Server Manager > Local Server > Ethernet instance > Right click on eth0 > properties > Internet protocol version 4 (TCP/IPv4) > Properties > input the following..

### Setting the static IP address

- ip address: 10.128.0.10
- subnet mask: 255.255.255.0
- default gateway: 10.128.0.1
- DNS1: 127.0.0.1
- DNS2: 10.128.0.1

Using a ping test, we were able to ensure that there was network connectivity. Commands as follows

- LAN - ping 10.128.0.1
- WAN - ping 8.8.8.8
- DNS - ping google.com

From here he configured the Network Time Protocol (NTP). This would set the timezone and sync with the LAN interface IP on the firewall. The path we used as follows.

### Configuring NTP 

Right click time in bottom right of screen > select "Adjust date/time" > Select "Change date and time..." > Change time zone to UTC-6:00 Central Time (U.S. & Canada) and select OK > Select the "Internet Time" tab and select "Change Settings" > In the server box, type in 10.128.0.1 and select the "Update now" box. Ensure the text below reads successful.

### Change the hostname 

Navigate to server manager > local server > select "computer name" > and type "dc" in the Computer description box, then select the "Apply box and then the "Change..." box >   

### Installing Active Directory Domain Services 

Within server manager of the WIN2012r2, we navigated to... local server > add roles and features and began the configuration process within the wizard. Steps as follows

- For the first 2 pages, "next" was selected to default options were used
- After selecting our server from the server pool, we selected next on the third page
- Checked the box next to "Active Directory Services Domain" on server roles, and then clicked "Add Features on the pop-up screen. Then next
- Next
- Next
- And finally, select the install button

Here, the installation process would begin. Navigating to the server manager notifications tab, we were able to see the progress. This notification is indicated by a yellow caution symbol. Next steps taken were...

- Select the "Promote this server to a domain controller" link
- Select the option to "Add new forest" and then we typed in the domain name of "widgets.localdomain" that was desired by our client.
- Next was selected for the remaining pages within the wizard, thus keeping all configuration options as default.
- The wizard checked to ensure all prerequisites were met, and the the "Install" button was selected.
 
After this, the server rebooted and the objective was completed successfully.

## Creating new active directory user accounts

For this section, our objective was to create multiple user accounts, assign them their permissions, and then add them to the "domain admins" security group. Here's how...

- Within server manager, we navigated to tools > Active directory users and computers > users > right click and select new > user > filled in info > next > right click the new user and select "Add to a group..." > type 
in group name in the box and select check names to underline it > select ok
This process was repeated again for each of the new user accounts. We configured 2 accounts per user. One being a standrard user account, and one being an admin account with escalated privileges.

## Preparing the WIN10 to join the domain

We began by changing the hostname to "WIN10", then changed the primary DNS server to the IP for the domain controller, and finally set the NTP to sync with the widgets.localdomain. We used the same steps as the ones 
performed at the beginning of the stage to setup the domain.

To join the WIN10, we used an admin account to join the domain within the control panel > system and security > system > computer name, domain, and workgroup settings > change settings > under computer name tab, click change > under member of, click domain and type the domain name (ours being widgets.localdomain) and select ok > and then ok again. Then we rebooted.

We then logged in with a standard user account to verify that the changes were made to our account.

## Setting the desktop background with GPO 

Here, we set a background for the domain user accounts. Here's how...

server manager > tools > group policy manager > widgets.localdomain > domains > right click deafault domain policy > edit > user configuration > policies > admin templates > desktop > desktop wallpaper and assign name (ours was c:\windows\web\wallpaper\windows\img.0.jpg) added into the comment and wallpaper name boxes > apply > ok 

We then logged out and logged back in where we were able to verify that our wallpaper had been changed. Since ours was successful, we did not have to use "gpupdate" or "gpresult" to troubleshoot, but they can be used if problems occur.

This was the completion of Stage 2. 
 
![Stage2 End](https://github.com/GitRoss16/Network-Project/assets/144251501/a09c3701-465d-4a1d-823f-613e90f38f2a)

# Stage 3: IIS Setup

In this stage, we built an IIS webserver on the WIN2012r2 server, then joined the server to the domain. 

## Preparing the WIN2012r2 server

This process was fairly simple as it required many of the same steps as the setup for the domain controller. Our requirements were as follows...

- Give static IP of 10.128.0.80
- Set the DC as primary DNS (10.128.0.10)
- Firewall LAN IP as secondary DNS (10.128.0.1)
- Sync NTP with dc.widgets.localdomain
- Change the hostname to iis 
- Join the server to the widgets domain

## Installing the IIS role 

Using server manager, our step progression went...

- Manage
- Add roles and features
- (within the first page of the wizard) click next 
- Select role-based or feature-based installation, then next
- Select appropriate server from the server pool, then next
- On the server roles page, select Web Server (IIS), then next
- We used defaults settings for the remaining pages
- On the verificatin page, we selected "restart the destination server automatically if required"
- After confirming our selections were correct, "install" was selected.

  Once the installtion had finished, we closed out the page, and went on to verify that the webserver worked. To do this...

- Opened our browser (Internet Explorer)
- Added http://localhost/ to our trusted sites list within the browsers settings menu
- Verified that the site was up and working when navigating to the address above

## Creating a test webpage on IIS 

We needed to make a testing webpage on IIS to make sure we could access it through the browser of the WIN10 workstation

Using notepad, we created an HTML file with the following text...

![Test page for IIS](https://github.com/GitRoss16/Network-Project/assets/144251501/75bba104-e4b4-4490-918a-13a945a6aa72)


We then saved this file to our documents folder and named it "test.html", then copied the file to the "c:\inetpub\wwwroot\test.html", and test the local browser in IIS at "http://localhost/test.html".

Then, using the WIN10 WS, navigated to "http://iis.widgets.localdomain/test.html" to verify. IIS setup was a success.

![image](https://github.com/GitRoss16/Network-Project/assets/144251501/e74ebc62-6fd4-4a54-8f6c-59055f2c1feb)

# Stage 4: LAMP setup

In this stage, we built a LAMP webserver on an Ubuntu server on our DMZ network. Our client requested the server name to be "www.widgets.localdomain"

## Preparing the Ubuntu server 

We began by starting the Ubuntu server and logging in. It had no network connection yet since it couldn't lease DHCP address because there is no DHCP server for the DMZ network. So, we set a static IP...

- IP = 10.128.10.80
- Netmask = 255.255.255.0
- Gateway = 10.128.10.1
- DNS = 10.128.0.10, 10.128.10.1

To make these changes, we navigated to the...

- Settings menu
- Wired connecting > Wired settings
- Network > settings gear in the "wired" tab
- IPv4 tab > selected manual > input the correct address information > selected apply

We then navigated to the terminal within Ubuntu and used the following commands to update the host file...

- 127.0.0.1 localhost.localdomain localhost
- 10.128.10.80 www.widgets.localdomain www
  
We then set the hostname with hostnamectl using...

- hostnamectl set-hostname www
- hostnamectl

After this we updated the packages on the server with the following commands...

- apt update -y
- apt upgrade -y (this one took roughly 45 minutes to complete) 
- apt dist-upgrade -y
- apt autoremove -y
- apt autoclean -y
- systemctl reboot

## Installing DokuWiki

This step required us to login as root within the terminal. The following commands were used...

- apt install php php-gd php-xml php-json -y
- systemctl enable --now apache2
- ufw allow Apache
- wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
- mkdir /var/www/html/dokuwiki
- tar xzf dokuwiki-stable.tgz -C /var/www/html/dokuwiki/ --strip-components=1

## Creating the config file for the wiki 

nano was used for this step. The command to access nano was... nano /etc/apache2/sites-available/dokuwiki.conf 

This is what was added into nano (if trying to replicate, ensure the proper tabbed spacing is used) 

![image](https://github.com/GitRoss16/Network-Project/assets/144251501/c76d131a-850d-4260-bcef-20e614a9efcd)

We saved and then exited back to the terminal in order to finish the config file...

- cp /var/www/html/dokuwiki/ .htaccess{.dist,}
- chwon -R www-data:www-data /var/www/html/dokuwiki
- apache2ctl -t
- a2dissite 000-default.conf
- a2ensite dokuwiki.conf
- systemctl reload apache2

## Verifying access to the installer

In this step we logged onto the DC, opened DNS manager, and added a new host (A) record to the widgets.localdomain domain under the "Forward Lookup Zones" tab. We assigned the following credentials to this host.

- name = www
- ip address = 10.128.10.80
- create associated pointer (PTR) record = unchecked
- add host

We then opened the dokuwiki installer URL in our browser on the WIN10 WS at "http://www.widgets.localdomain/install.php" and used the following credentials...

- wiki name = Widgets Network Documentation Wiki
- superuser = wikiadmin
- real name = service account
- e-mail = wikiadmin@widgets.localdomain
- password = Passw0rd!
- initial ACL policy = Public Wiki

We then renamed the file on www within the terminal using...

- mv /var/www/html/dokuwiki/install.php /var/www/html/dokuwiki/install.php.removed

We then logged into the WIN10 and opened the the wiki URL (http://widgets.localdomain/) and logged in. We created a start page with the following information

![image](https://github.com/GitRoss16/Network-Project/assets/144251501/2636a313-3fb3-45ff-8a08-35d940095776)

Looking back through those start page notes (section provided down below), we discovered there were still two things we needed to do, and that was to create 2 additional hosts on the DC. This would be done the same way we created the "www.widgets.localdomain". The new hosts would be...

- FQDN = firewall.widgets.localdomain
- a-record created = not yet
- FQDN = iis.widgets.localdomain
- a-record created = not yet

Once those were created, we went back into the wiki and updated the notes to display the correct information.

## Setup VIP on firewall

On the WIN10 WS, we navigated to the FortiGate webserver previously, navigated to Policy & Objects > Virtual IPs > Create New > Virtual IP and assigned the following credentials to the new VIP

- name = www_tcp_80
- interface = WAN
- mapped IP = 10.128.10.80
- port forwarding = enabled
- external port service = 80
- map to port = 80

Navigated to IPv4 Policy (within the Policy & Objects tab) and editd the " WAN-to-DMZ" firewall policy destination to "www_tcp_80".

We then used the terminal to find the WAN IP of the firewall using the method "show sys int (space) then press and hold shift and the ? key" to look for and find the port1 address of the WANs public address of the firewall.

We then used our home computer to access the wiki from the webbrowser at "http://wan-ip-of-firewall/", and completing the final step of Stage 4.

![image](https://github.com/GitRoss16/Network-Project/assets/144251501/f767f42a-b528-4a9e-8bc7-c786ae5d9268)


# Stage 5: FTP Setup

In this stage, we built the FTP server on the WIN2012r2 server in the DMZ Network. Our client requested the name of this server be "ftp.widgets.localdomain". The beginning of this stage had the same inital steps as our last stage, just different information which was...

- hostname = ftp
- ip address = 10.128.10.21
- subnet mask = 255.255.255.0
- default gateway = 10.128.10.1
- DNS1 = 10.128.0.10
- DNS2 = 10.128.10.1
- sync ntp with: dc.widgets.localdomain
- join the widgets domain

## Installing the FTP Service

To install the FTP Service we navigated to... Server Manager > Manage > Add roles and features > Role-based or feature-based installation > Selected the proper server from server pool > Selected Web Server (IIS) > Next button until we reach the Role Services page > select FTP Server > next > install > and reboot

This completed Stage 5

![image](https://github.com/GitRoss16/Network-Project/assets/144251501/d7ca35a6-50d7-417b-a44e-06650575e27e)

# Stage 6: Hardening the Environment 

Stage 6 was an opportunity for us to explore the various options for hardening the firewall, WIN10 WS, WIN2012r2 Servers, and our Ubuntu Server. Though they were not all implemented into the system, we put our findings  on the wiki page.


# Stage 7 - The Bonus Stage: Scanning the Environment

For this stage, we used Greenbone to scan the WAN interface for the Widgets network environment.

## Creating a credential record 

Using the browser from a home computer, we navigated to the Greenbone URL provided > Configuration tab > Credentials > New Credential and then used the following information...

- name = <team>-fortigate-ssh-creds #ex: alpha-fortigate-ssh-creds
- #unless you've changed it
- username = admin
- password = Passw0rd!
- #save

## Creating a target record 

Configuration > Targets > New Target and used the following info in the appropriate boxes...

- name = <team>-fortigate-wan #ex: alpha-fortigate-wan
- hosts.manual = fortigate-wan-ip
- allow simultaneous... = no
- ssh = select the credential record you create
- #save

## Creating a task record 

Scans > Tasks > New Task and input following information...

- name = <team>-fortigate-task #ex: alpha-fortigate-task
- scan targets = the tatget record you created
- #save

After checking with our Senior Engineer, we were given the go ahead to scan the target. To do this, we navigated to the tasks page, located the task, and then ran it.
   
All stages were now complete and it was time to document the findings of our target scan on our Wiki page.



# Overview 

In this scenario, my team and I were able to build...

- A SMB (small/medium business) network, with a LAN, DMZ, and Guest network.
- A Windows domain enviornment.
- IIS Webserver
- Windows FTP Server
- WIN10 Workstation
- LAMP Webserver ruunning on Ubuntu (also hosted our Wiki)
- Fortigate firewall with a virtual IP for a DMZ Webserver. 




























