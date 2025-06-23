# Cisco Office Configuration Project
This is a project where I configure two office subnets consisting of core switches, distribution switches, access layer switches, routers, end devices, and wireless devices in Cisco Packet Tracer.

![image](https://github.com/user-attachments/assets/95809554-9ea2-4d4c-81e2-3269631dfb57)


# Objectives
1. Initial Setup - Configure hostnames, usernames, and passwords
2. Configure VLANs and L2 EtherChannel
3. Configure Layer 3 EtherChannel, ip address, and HSRP
4. Configure Spanning Tree Protocol (STP)
5. Static and dynamic routing
6. Network Services: DHCP, DNS, SSH, and NAT
7. Security: ACLs and L2 Security Features

## Step 1: Initial Setup
The first thing I need to do is set up each device to have its own hostname that follows the naming convention in the picture above. Then we need to create a username, password, and enable password for each device. for simplicity, each device's username will be _admin_ and their password and enable password will both be _cisco_ use the following commands on each device:

    enable
    config t
    hostname "hostname"
    username admin privilage 15 secret cisco
    enable secret cisco
    do write

Note that everything in "" will change from device to device. Adding the "Do write" command at the end will save the configuration, so if the device turns off it will not forget all the configurations I have done. It's important to do this often while working on a lot of devices. Shown below is the commands on R1:

![image](https://github.com/user-attachments/assets/3a84baea-7c6a-42ee-923a-7ab6e54cf86c)

### Step 1: Verification
To verify everything is working use the "reload" command to reboot any device. From there it will ask you for your username and password:

![image](https://github.com/user-attachments/assets/37ef712b-4e71-4301-82cf-1ce860e9870e)

##Step 2: Configure VLANs and L2 EtherChannel

### VLANs
Next I need to set up the VLANs. For this project there are five VLANs: PCs (VLAN10), Phones (VLAN20), Servers (VLAN30), Wi-Fi (VLAN40), and management (VLAN99). By logically separating your network into different VLANs it allows us to improve performance, enhance security, and streamline network management. For this section I will be focusing on the distribution and access layer switches


### PAgP on DSW A1 and A2
Next I will configure PAgP and LACP on the distribution an core switches. On DSW A1 and DSW A2, I will configure PAgP using the commands for both devices:

    enable 
    config t
    int range g1/0/4-5
    channel-group 1 mode desirable

    ### Verificaiton Command
    do show ether summary

![image](https://github.com/user-attachments/assets/29304ef3-5328-4c91-a22f-216179d35398)

![image](https://github.com/user-attachments/assets/c8ee0479-b5b3-4dcf-a01b-6a5fb230c0b4)

As you can see, we have PAgP enabled between DSW A1 and A2.

Next we will configure trunks on the ports connected to other switches. Office A will have VLANs 10,20,40, and 99. So I need to allow those on the trunks.

    int range g1/0/1-3
    switchport moode trunk
    switchport nonegotiate
    switchport trunk native vlan 100
    switchport trunk allowed vlan 10,20,40,99

### LACP on DSW B1 and B2

On DSW B1 and B2, I will instead configure LCAP, which is similar to PAgP, but it isn't Cisco proprietary. To do this use the commands below:

    enable
    config t
    int range g1/0/4-5
    channel-group 1 mode active

![image](https://github.com/user-attachments/assets/c178ce5d-5510-41aa-8b16-d845cd939ebe)

### Step 3: Configure IPv4 Address on each interface

Use the following table to configure each router and switch with the approprate IP address and subnet mask, as well as loopback interface. Then close all unused ports.

    int [INTERFACE]
    ip address [Enter IP address] [subnet mask]
    no shut
    exit
    int loopback 0
    ip address [Loopback address] [255.255.255.255]
    no shut
    exit
    int range [interface ranges not being used]
    shut
    exit

for Verification use

    do show ip int br

![image](https://github.com/user-attachments/assets/70532e28-da52-4f63-80b1-112559024738)


For the access layer switches, we also need to configure a default gateway which will allow us to connect to these devices via SSH, then add their ip address to Vlan 99

    ip default-gateway 10.0.0.1
    int vlan 99
    ip address [ip address] [subnet]
    exit
    
![image](https://github.com/user-attachments/assets/457f072f-d00a-40bf-ac81-fa5454746d12)

IPv4 connections

![image](https://github.com/user-attachments/assets/860d38ec-d48c-45ef-ada1-5b8669af89bf)

Interface connections

### Step 4: Configuring HSRP

HSRP is the Hot Standby Router Protocol. It acts as a failsafe in the event that if one router goes down, there is redundency in the network so one can fail over to the other without a loss in connection. To do this, you will need to configure HSRP on each distribution layer switch (DSW switch) and configure on each of our subnets (Vlan 10,20,30,40 and 99). The command structure is the same, but inputs are slightly different

    int vlan [VLAN]
    ip address [ip address] [subnet]
    standby ver 2
    standby [number] ip [Virtual IP (VIP)]
    standyby [number] priority [105 or 95]
    standby [number] preempt

With HSRP, assigning a priority allows the router to decide which router is the primary, and which one is the backup. Its also important that each vlan is on a different standby number. For example, with vlan 10, the standby number will be 1, but for vlan 20, the standby number will be 2. This allows for deconfliciton between vlans. For Vlans 10 and 99 I made the active router A1/B1 and for vlan 20,30, and 40 router A2/B2 will be the primary. So their priority will be 105, while the backup router will be 95.

![image](https://github.com/user-attachments/assets/6325c2be-e28f-46ac-97ae-5f2232f29bd5)

for Verification

    do show standby brief
    
![image](https://github.com/user-attachments/assets/5229d8ef-a596-4c14-8255-d43c8b062889)

### Step 5: Configure Spanning Tree Protocol

We will be configuring RSTP on all distribution and access layer switches. as well as enabling portfast and BPDU Guard on all end host interfaces. We will need to insure that the root bridge is alligned with the HSRP active router. So our layer 2 and layer 3 connections are going the same direction. So for Vlan 10 and 99, DSW A1/B1 will have the lowest STP priority. and while DSW A2 and B2 will have lowest STP priority for 20,30, and 40

    spanning-tree mode rapid-pvst
    spanning-tree vlan 10,99 priority 0
    spanning-tree vlan 20,30,40 priority 4096

For DSW A2/B2 it will be the opposite

    spanning-tree mode rapid-pvst
    spanning-tree vlan 20,30,40 priority 0
    spanning-tree vlan 10,99 priority 4096

![image](https://github.com/user-attachments/assets/22fde282-1969-4bdc-9abe-940586e5926e)

for verification:

    do show spanning-tree
    
![image](https://github.com/user-attachments/assets/22a6a20c-21a7-4a94-bccb-c97cca06898e)

For configuring portfast and BPDU Guard, go to all ports on end hosts and use the following commands These commands will only be on the access layer switches .

    int f0/1
    spanning-tree portfast
    spanning-tree bpduguard enable
    (optional) spanning-tree portfast trunk

![image](https://github.com/user-attachments/assets/b92fc69b-797b-4c86-bfbf-16f5d668ba09)

### Step 6: Static and Dynamic Routing
First we will be configuring OSPF on R1 and all core and distribution switches. We will use a process ID of 1 with an area of 0. We will also configure each device's router ID (RID) to match the loopback interface ID. All loopback interfaces should be passive interfaces.

    router ospf 1
    router-ID [loopback IP]
    passive-interface l0
    passive-interface vlan 10
    passive-interface vlan 20
    passive-interface vlan 30
    passive-interface vlan 40
    int l0
    ip ospf 1 area 0
    int range [LAN facing interfaces]
    ip ospf 1 area 0
    ip ospf network point-to-point

![image](https://github.com/user-attachments/assets/7ed3a4e8-b0b8-4965-8f54-e6b21ec2d89a)

Verification:

    show ip ospf neighbor

![image](https://github.com/user-attachments/assets/3e499f53-41df-4634-bdb3-939df06c3b01)

## Static routing

Next we will make one static route on R1 internet facing interfaces and one floating route to the internet. 

    ip route 0.0.0.0 0.0.0.0 203.0.113.1
    ip route 0.0.0.0 0.0.0.0 203.0.113.5 2

## Step 7: Configuring DHCP

We will be configuring DHCP on R1 for each vlan in each office the following pools will be as follows with the first 10 addresses as excluded addresses:

a. Pool: A-Mgmt
i. Subnet: 10.0.0.0/28
ii. Default gateway: 10.0.0.1
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)
v. WLC: 10.0.0.7

b. Pool: A-PC
i. Subnet: 10.1.0.0/24
ii. Default gateway: 10.1.0.1
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)

c. Pool: A-Phone
i. Subnet: 10.2.0.0/24
ii. Default gateway: 10.2.0.1
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)

d. Pool: B-Mgmt
i. Subnet: 10.0.0.16/28
ii. Default gateway: 10.0.0.17
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)
v. WLC: 10.0.0.7

e. Pool: B-PC
i. Subnet: 10.3.0.0/24
ii. Default gateway: 10.3.0.1
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)

f. Pool: B-Phone
i. Subnet: 10.4.0.0/24
ii. Default gateway: 10.4.0.1
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)
g. Pool: Wi-Fi

i. Subnet: 10.6.0.0/24
ii. Default gateway: 10.6.0.1
iii. Domain name: anydomain.com
iv. DNS server: 10.5.0.4 (SRV1)

First for the excluded addresses on R1:

    ip dhcp excluded-address 10.0.0.1 10.0.0.10
    ip dhcp excluded-address 10.1.0.1 10.1.0.10
    ip dhcp excluded-address 10.2.0.1 10.2.0.10
    ip dhcp excluded-address 10.0.0.17 10.0.0.26
    ip dhcp excluded-address 10.3.0.1 10.3.0.10
    ip dhcp excluded-address 10.4.0.1 10.4.0.10
    ip dhcp excluded-address 10.6.0.1 10.6.0.10

Next for configuring the DHCP pools for each vlan use these commands for each vlan:

    ip dhcp pool [NAME OF POOL]
    network 10.x.x.x 255.255.255.0
    default-router 10.x.x.x
    dns-server 10.5.0.4
    domain-name: anydomain.com

![image](https://github.com/user-attachments/assets/54ee1088-5edd-4f3b-88ae-b726e9891d28)

next we need to configure the distribution switches as DCHP relay agents in order for the DHCP pool to reach the acess layer and user level computers. On the interface that recieves DHCP request messages needs to have the following helper address. This must be configured on the interface that recieves the broadcast DHCP from clients.

    interface vlan 10
    ip helper-address 10.0.0.76
    interface vlan 20
    ip helper-address 10.0.0.76
    interface vlan 30
    ip helper-address 10.0.0.76
    interface vlan 40
    ip helper-address 10.0.0.76
    interface vlan 99
    ip helper-address 10.0.0.76

now the PC should be getting IP addresses from R1 to confirm, go onto any of the computers and verify their IP address is from R1.

![image](https://github.com/user-attachments/assets/b0fbf3af-c9f6-4e75-b841-f787efd2c026)

As we can see, we are getting the DHCP ip address from the 10.3.0.x subnet from R1.

Step 8: Configuring DNS
First we need to go on our SRV1 and enable DNS and configure a few DNS entries. go into the SRV1 and turn on DNS in the DNS tab, make a new A Record and add youtube.com at 152.250.31.93 and google.com at 152.250.31.93

![image](https://github.com/user-attachments/assets/f14499e1-033d-4b66-856a-6c131ed239ba)

Then on all devices make sure they can get to your domain with the following commands:

    ip domain name anydomain.com
    ip name-server 10.5.0.4

## Step 9: Confiuring SSH
Configuring SSH is very important, as it allows us to remote into any of our devices via SSH, in order to make any changes as necessary. Use the following commands on each device to set up SSH connections.

    ip domain name anydomain.com
    crypto key generate rsa 
    2048
    ip ssh version 2
    line vty 0 15
    transport input ssh
    login local
    logging synchronous

## Step 10: Configure NAT
First we will configure a static NAT to enable hosts on the internet to access SRV1 via the ip address 203.0.113.113. On R1 confugre with the following commands, be sure to configure the inside facing interfaces with IP nat inside, and outside facing interfaces with IP nat outside commands:

    ip nat inside source static 10.5.0.4 203.0.113.133
    int range g0/0/0,g0/1/0
    ip nat outside
    int range g0/0-1
    ip nat inside
    exit

Next we need to configure dynamic NAT, also known as PAT to translate inside IP addresses to access the internet. We will need to make standard ACLs to apprropriately address the inside addresses with outside addreses on R1, our internet facing router. Our NAT pool will be named POOL1 with a range of 203.0.113.200-203.0.113.207/29

i. Office A PCs: 10.1.0.0/24
ii. Office A Phones: 10.2.0.0/24
iii. Office B PCs: 10.3.0.0/24
iv. Office B Phones: 10.4.0.0/24
v. Wi-Fi: 10.6.0.0/24

    access-list 1 permit 10.1.0.0 0.0.0.0.255
    access-list 1 permit 10.2.0.0 0.0.0.0.255
    access-list 1 permit 10.3.0.0 0.0.0.0.255
    access-list 1 permit 10.4.0.0 0.0.0.0.255
    access-list 1 permit 10.6.0.0 0.0.0.0.255
    ip nat pool POOL1 203.0.133.200-203.0.133.207 netmask 255.255.255.248
    ip nat insdie source list 1 pool POOL1 overload

to verify, go on one of the PCs and try to ping one of the domain names we created. For this I pinged google.com

![image](https://github.com/user-attachments/assets/34163de4-ac76-4c86-b307-64cb93cc7945)

and success, out PAT is correctly configured.

## Step 11: Layer 2 Security Features

Next we will configure layer 2 security features on all of our access layer switches to include port security, which allows only minimum number of MAC addresses on each port, DHCP Snooping on all vlans in each LAN which prevents unauthorized devices from acting as DHCP servers in our network, and finally Dynamic ARP Inspection (DAI) which validates ARP packets to prevent ARP spoofing attacks.

### Port Security
Port security is configured directly on the interface facing the end hosts, for all of our cases, int f0/1. This should be configured on ASW-A1, ASW-B1, and ASW-B3.

    int f0/1
    switchport port-security
    switchport port-security violation restrict
    switchport port-security mac-address sticky

interfaces ASW-A2,ASW-A3, and ASW-B2 need an additional command to allow them to have two port connections because they have two devices attached to their int f0/1

    switchport port-security maximum 2

### DHCP Snooping
For this we need to do the following on all access layer switches:

1. Enable it for all active vlans in each LAN
2. Trust the appropriate ports.
3. Disable insertion of DHCP Option 82.
4. Set a DHCP rate limit of 15 pps on active untrusted ports. This prevents packets coming from an untrusted port to a limit of 15 pps.
5. Set a higher limit (100 pps) on ASW-A1’s connection to WLC1.

       ip dhcp snooping
       ip dhcp snooping vlan 10,20,30,40,99
       ip dhcp snooping information option
       int range g0/1-2
       ip dhcp snooping trust
       int f0/1
       ip dhcp snooping limit rate 15
       int f0/2 ip dhcp snooping limit rate 100

### Dynamic ARP Inspection
DAI only needs to be enabled per vlan. It does not need to be enabled globally.

    ip arp inspection 10,20,30,40,99
    ip arp inspection validate dst-mac src-mac ip
    int range g0/1-2
    ip arp inspection trust

## Conclusion
After all of this we have fully configured two offices within different VLANs as well as all IP connections and routes, we configured hostnames, VLANS, OSPF, Static routes, STP, HSRP, NAT, SSH, various layer two security features As well as documenting all of our progress and verifying everything worked along the way. 
