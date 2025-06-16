# Configuring Two Offices - Cisco 
This is a project where I configure two office subnets consisting of core switches, distribution switches, access layer switches, routers, end devices, and wireless devices in Cisco Packet Tracer.

![image](https://github.com/user-attachments/assets/b266e798-fe29-409c-8d29-bcb847300c13)


# Objectives
1. Initial Setup - Configure hostnames, usernames, and passwords
2. Configure VLANs and L2 EtherChannel
3. Configure Layer 3 EtherChannel, ip address, and HSRP
4. Configure Spanning Tree Protocol (STP)
5. Static and dynamic routing
6. Network Services: DHCP, DNS, NTP, SNMP, Syslog, FTP, SSH, and NAT
7. Security: ACLs and L2 Security Features
8. Configure IPv6
9. Configure Wireless

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
    int l0
    ip ospf 1 area 0
    int range [LAN facing interfaces]
    ip ospf 1 area 0
    ip ospf network point-to-point

![image](https://github.com/user-attachments/assets/7ed3a4e8-b0b8-4965-8f54-e6b21ec2d89a)

Verification:

    show ip ospf neighbor

![image](https://github.com/user-attachments/assets/3e499f53-41df-4634-bdb3-939df06c3b01)


    
