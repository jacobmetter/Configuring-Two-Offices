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


    
