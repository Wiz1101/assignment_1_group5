# Group 5

Labs:
Day | Time    
-| -| 
Thursday |  10:00 - 12:00
Friday | 10:00 - 12:00


Team Members
- Khairallah Ahmad Obay
- Napetvaridze Shota
- Palomares Estrella Hugo Edgar
- Rindebrant Wictor
- Shaban Ryustem


Linux server login creds: 
- username: group5
- password: group5 

Windows server login creds: 
- username: Administrator
- password: Group5 

The commands we used to reset to factory default settings before starting:

1. **Firewall (Cisco ASA 5510)**:
    - useful commands: # show running-config


   ```shell
    G5-ASA-5510> enable
    G5-ASA-5510# conf t
    G5-ASA-5510(config)# configure factory-default
    G5-ASA-5510(config)# wr mem
   ```

2. **Switch (Cisco Catalyst 3550 SERIES)**:

   ```shell
    Switch> enable
    Switch# conf t
    Switch(config)# configure factory-default
    Switch(config)# no spanning-tree vlan 1
    Switch(config)# wr mem
   ```

2. **Access Point (Cisco Aeronet 3502 series)**:

   ```shell
    ap> en
    ap# write erase
    ap# reload
   ```
## The configuration steps on devices
**Important Note**: Before proceeding, ensure that you have console access to the Cisco ASA 5510 through a console cable and terminal emulation software like PuTTY or a similar tool.

1. **Connect to the Cisco ASA 5510**:
   - Connect one end of the console cable to the console port on the ASA 5510.

2. **Enter privileged EXEC mode by typing**:

   ```shell
   ciscoasa> enable
   ```

   - Enter global configuration mode:

   ```shell
   ciscoasa# conf t
   ```

3. **Setting Hostname** (Optional):
   - You can set a hostname for the ASA if desired:

   ```shell
   ciscoasa(config)# hostname G5-ASA-5510
   G5-ASA-5510(config)# wr mem

   ```

   
Done
4. **Configuring Interfaces**:
   - Define the outside and inside interfaces.

   ```shell
   G5-ASA-5510(config)# int Ethernet0/0
   G5-ASA-5510(config)# nameif outside
   G5-ASA-5510(config)# ip address 192.168.0.50 255.255.255.0
   G5-ASA-5510(config)# security-level 0
   G5-ASA-5510(config)# no shut
   G5-ASA-5510(config)# int Ethernet0/1
   G5-ASA-5510(config)# nameif inside
   G5-ASA-5510(config)# ip address 10.0.0.1 255.255.255.0
   G5-ASA-5510(config)# security-level 100
   G5-ASA-5510(config)# no shut
   G5-ASA-5510(config)# wr mem
   ```

   - The above configuration says that Ethernet0/0 will be the interface for outside network and Ethernet0/1 is the interface for inside network.

5. **Configuring Routing(from inside to outside)**:

   ```shell
   G5-ASA-5510(config)# route outside 0 0 192.168.0.1
   ```



6. **Configuring DHCP & DNS inside**:
    - Address range of DHCP inside network: 10.0.0.100 - 10.0.0.200 
    - We binded DNS to OpenDNS servers (More Secure)
   ```shell
   G5-ASA-5510(config)# dhcpd address 10.0.0.100-10.0.0.200 inside
   G5-ASA-5510(config)# dhcpd dns 208.67.222.222 208.67.220.220
   G5-ASA-5510(config)# dhcpd lease 7200                          
   G5-ASA-5510(config)# dhcpd enable inside 
   G5-ASA-5510(config)# wr mem
   ```

7. **Configuring NAT**:
   - Configure basic NAT to allow inside devices to access the internet:

   ```shell
   G5-ASA-5510(config)# object network myNatPool
   G5-ASA-5510(config-network-object)# range 192.168.0.51 192.168.0.52
   G5-ASA-5510(config-network-object)# exit
   G5-ASA-5510(config)# object network myInsNet  
   G5-ASA-5510(config-network-object)# subnet 10.0.0.0 255.255.255.0  
   G5-ASA-5510(config-network-object)# nat (inside,outside) dynamic myNatPool
   G5-ASA-5510(config-network-object)# exit 
   
   TODO 
   G5-ASA-5510(config)# object network myLinuxServer
   G5-ASA-5510(config-network-object)# host 192.168.0.50  
   G5-ASA-5510(config-network-object)# nat (outside,inside) static 192.168.2.13
   G5-ASA-5510(config-network-object)# exit
   G5-ASA-5510(config)# same-security-traffic permit inter-interface
   G5-ASA-5510(config)# wr mem 

   ```

8. **Configuring Access Control List (ACL)**:

   ```shell

    G5-ASA-5510(config)# access-list allow_icmp_inside extended permit icmp 10.0.0.0 255.255.255.0 192.168.0.0 255.255.255.0
    G5-ASA-5510(config)# access-group allow_icmp_inside in interface inside
    G5-ASA-5510(config)# wr mem
   ```

9. **Configuring DNS**:

   ```shell

   ```

## The configuration steps on Servers
1. **Configuring static IP for linux server**:
   - We have to edit "/etc/netplan/00-installer-config.yaml" file
   ```shell
   network:
      version: 2
      renderer: networkd
      ethernets:
         eth0:
            addresses:
               - 10.0.0.250/24
            routes:
               - to: default
                 via: 10.10.10.1
            nameservers:
               addresses: [208.67.222.222, 208.67.220.220]

   ```
   ```shell
   $ sudo netplan apply

   ```
1. **Configuring HTTP server on linux**:

   ```shell
   $ sudo apt install apache2
   $ sudo ufw enable 


   ```

2. **Configuring FTP server on linux**:

   ```shell

   ```