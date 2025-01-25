<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1> Manipulating Network Security Groups (NSGs) and Analyzing Traffic Between Azure Virtual Machines</h1>

- In this tutorial, we will observe network traffic of various protocol types being transmitted to and from Azure Virtual Machines using Wireshark as well as experimenting with Network Security Groups. <br />

- NOTE: Network Security Groups (NSGs) are essentially access control lists that contain statically configured rules that regulate access to and from various computing resources in Azure.

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDP, DNS, DHCP, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)
- Ubuntu Server 24.04 LTS

<h2>High-Level Steps</h2>

- Step 1: Create your Windows 10 and Ubuntu VMs (place them in the same Vnet and subnet)
- Step 2: Observe ICMP Traffic 
- Step 3: Observe SSH Traffic
- Step 4: Observe DHCP Traffic
- Step 5: Observe DNS Traffic
- Step 6: Observe RDP Traffic

<h2>Actions and Observations</h2>


<h3> STEP 1.0: Create the Windows 10 VM in Azure. </h3>

- Access the [Azure portal](https://portal.azure.com) to begin creating your VM.
- Under the "Basics" tab:
  - Ensure to create a Resource Group (RG) and a Subscription (if needed) while creating the VM
    - The VM will be placed into this RG (consider it a folder for your Azure resources).
    - Name the VM
    - Place it in the region closest to you
    - Select Windows 10 Pro 22H2 for the VM's operating system image

  ![image](https://github.com/user-attachments/assets/2a99d7d7-3253-442e-94a9-04bd06e0641a)

    - Set the VM's Size to be at least 2 Virtual CPUs (vCPUs) and 8 GiB (gigabytes) of memory.
    - Assign the credentials to the local administrator account that will be created for this VM.
    - Ensure TCP port 3389 is selected as an allowed inbound port (this will enable us to establish a remote desktop connection to this VM)

![image](https://github.com/user-attachments/assets/801bc238-83a5-4da5-ae89-5135182353d9)

    
- Navigate to the "Networking" tab:
    - Create a new Virtual Network (VNet) and subnet
    - Ignore the security warning regarding exposing the RDP port to all IP addresses on the internet we will restrict the source IP addresses that can 
       access this VM via RDP in a later step. 

  ![image](https://github.com/user-attachments/assets/f5914c09-e494-489f-9eb0-548f2d80e0a8)

- "Review + Create" and once the validation process has been passed create the VM.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.15: Harden the Windows 10 VM by Restricting the Source IP Addresses that can Connect to it via RDP </h3>

- Once the VM is created, navigate to its Network settings and select the RDP rule configured in its Network Security Group ruleset.
  - Change the allowed source IP addresses from any to "My IP address" to restrict the RDP connection to only originate from your current IP address
  - Or you could select "IP Addresses" and specify an IP address(es) that you would like to connect from.
 
![image](https://github.com/user-attachments/assets/8364fb65-a882-462e-8d9c-7a6a87073f5c)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.30: Create the Ubuntu Virtual Machine. </h3>

- Still within the [Azure portal](https://portal.azure.com) create another VM
  - Under the "Basics" tab: 
    - Place it into the previously created Resource Group.
    - Name the virtual machine
    - Place it into the same region as the previously created VM.
    - Assign "Ubuntu Server 24.04 LTS" as the virtual machine's operating system image.

![image](https://github.com/user-attachments/assets/6a498366-dfcb-42af-ae0e-c7bb8b9eacbe)



   - Change the "Authentication type" for the local administrator account from "SSH public key" to "Password"
   - Provision login credentials for the admin account.
   - Select port 22 as an allowed inbound port
     - This will enable us to establish an SSH (Secure Shell) connection to the Ubuntu VM.
    

   - Under the "Networking" tab:
     - Ensure the Ubuntu VM is placed into the same virtual network (Vnet) and subnet as the Windows 10 VM.
    
![image](https://github.com/user-attachments/assets/5a299645-1e19-425f-8224-3eb9d7e15fca)

   - "Review + Create", once the validation has successfully passed then create the VM.
     - NOTE: you may receive this error message, simply return to the Basics tab and verify the configuration then "Review + Create" again.

![image](https://github.com/user-attachments/assets/26ff24d5-2fbe-43d5-87de-b71656bbb3ab)
       
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.45: Harden the Ubuntu VM by Restricting the Source IP Addresses for SSH Connection </h3>

- Within the VM's network settings:
  - Modify the SSH rule in the network security group by restricting the IP addresses that can establish an SSH connection to the Ubuntu VM.
  - I explicitly listed the private IP address of the Windows 10 VM (10.0.0.4) as the allowed source IP address.
 
  ![image](https://github.com/user-attachments/assets/9c0d2be3-b433-4bb1-a793-31cbb26c78a5)


-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.60: Observe the Topology of the created Virtual Network within the Network Watcher </h3>

- From the home page of the Azure portal:
  - Navigate to "Virtual Networks" > (Name of the virtual network)
    - Select Monitoring > Diagram

- Verify that both VMs are connected within the same subnet.

![image](https://github.com/user-attachments/assets/ec570eea-a7bc-47a9-865e-0bc1c1974adc)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.75: Connect to the Windows 10 VM and Install Wireshark </h3>

- Establish the remote desktop connection to the Windows 10 VM via its public IP address and your preferred RDP client.
   - Use the admin account's credentials that were provisioned during the VM's creation. 

![image](https://github.com/user-attachments/assets/58dac6bf-8124-4846-b52d-fe2adbbe58e7)


- Browse to the [wireshark download link](https://www.wireshark.org/download.html) within the Windows 10 VM.
  - Select the "Windows x64 Installer"

![image](https://github.com/user-attachments/assets/32c20bf3-52bb-4742-8792-bdcf68190a2a)

- Open the Wireshark x64 Setup wizard.

![image](https://github.com/user-attachments/assets/8ae35629-68f2-4070-afeb-cb74adc1c9a9)

- The default configurations can be left unchanged and install the default components.
- Install Npcap as well.

![image](https://github.com/user-attachments/assets/baa9e310-aeaa-4b1b-9160-5d2c090454e3)


-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 2.0: Open Wireshark and Filter for ICMP Network Traffic.</h3>

- Open Wireshark and filter for ICMP traffic exclusively.
  - This is the network protocol that the "ping" command line utility uses to perform its network diagnostic/troubleshooting operations  
  - Right-click the "Ethernet" network interface and select "Start capture"
  - Enter "icmp" in the Wireshark search filter.

![image](https://github.com/user-attachments/assets/9f1656d6-c097-4359-a31f-79e0eee09e8b)


-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 2.33: Retrieve the Private IP Address of the Ubuntu VM from the Azure Portal and Attempt to Ping it (Perpetually) From the Windows 10 VM. </h3>

- Open the command prompt or PowerShell to run the command "ping (Ubuntu VM IP address) -t"
  - My Ubuntu VM's private IP address is 10.0.0.5
  - The "-t" parameter will run the ping command perpetually/non-stop until it is manually stopped (Ctrl + C)

![image](https://github.com/user-attachments/assets/4ec8af7d-56ad-4082-9849-422ecb15485a)



- Observe the ICMP packets that were captured and displayed by Wireshark 
- Notice that the source IP address is the same as the Windows 10 VM's IP address (10.0.0.4) and the destination IP address is the Ubuntu VM's IP address for ICMP 
  Echo (ping) requests and vice versa for ICMP Echo (ping) replies
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 2.66: Attempt to Ping a Public IP address/domain such as Google's Domain (www.google.com) and Observe the ICMP Traffic in Wireshark </h3>

- Restart the Wireshark packet capture
- Run the command "ping www.google.com"

  ![image](https://github.com/user-attachments/assets/e87e3cac-b045-4c10-b815-2eda3918d5e8)

  - Observe the source and destination IP addresses for ping requests and replies respectively.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

  <h3> STEP: 2.99: Initiate Another Perpetual Ping from the Windows 10 VM to the Ubuntu VM, then Modify the NSG Associated with the Ubuntu VM to Block Inbound 
   ICMP Traffic 
  </h3>

- Run the command "ping (Ubuntu VM IP address) -t"
- Initiate another ICMP packet capture in Wireshark as well. 

![image](https://github.com/user-attachments/assets/24a2a509-7503-4dca-8d0c-10f89f844194)


- Within the [Azure portal](https://portal.azure.com), access the network settings for the Ubuntu VM and create a new inbound port rule within its network security group (NSG). 

![image](https://github.com/user-attachments/assets/a9075852-3206-48bf-95d0-935c97ebca89)

- Create a new Inbound security rule 
- Assign a higher priority (lower number) to the new rule so it will be evaluated and executed earlier in the NSG ruleset.
  - Select "ICMPv4" from the list of network protocols
    - Notice that the destination port ranges field immediately gets filled with an * once ICMP is selected.
    - This protocol does not use port numbers during its operations
    - It operates directly on top of the IP protocol and does use a transport layer protocol (eg. TCP or UDP) which would require a port number to identify the 
       particular application or service being used on the network. 
  - Set the Action to "Deny"
  - Name the NSG rule appropriately
  - Add the rule

![image](https://github.com/user-attachments/assets/520fc9fb-3bb5-4267-825e-0b6d8fa231ca)


- Observe the ICMP echo (ping) requests being blocked by the NSG in real-time.

![image](https://github.com/user-attachments/assets/7baf98c9-0dd8-457e-8fa1-ead805f6e9e0)


- Finally, Re-enable inbound ICMP traffic to the Ubuntu VM by deleting the newly created NSG rule (or modifying it to allow ICMP traffic) in the Azure portal

![image](https://github.com/user-attachments/assets/65710b93-049d-4e49-9cec-073c987b6ed6)

- Observe that the ICMP request/response process begins working again...eventually😅 

![image](https://github.com/user-attachments/assets/a8d8c253-eb8d-4816-80c0-8337822edb65)

- NOTE: Stop the perpetual ping on your Windows 10 VM by entering (Ctrl + C)

-------------------------------------------------------------------------------------------------------------------------------------------------------------


 <h3> STEP 3.0: Establish an SSH connection from the Windows 10 VM to the Ubuntu VM and Initiate a New Packet Capture in Wireshark, Filtering for SSH traffic. </h3>

- NOTE: The SSH (secure shell) protocol facilitates the establishment of secure, encrypted access to the command line interface (CLI) of a remote host via a network connection.

- Enter "tcp.port == 22" or "ssh" in the Wireshark search filter.
  - SSH uses TCP port 22 for its remote connections

- Open the command prompt and run the command "ssh (ubuntu-vm-admin username)@(ubuntu-vm private IP address)
   - I entered "ssh ubuntu-vm-admin@10.0.0.5" to establish the SSH connection to the Ubuntu VM.
           - Notice SSH traffic is already being captured before even accessing the Ubuntu VM's CLI.


- Type "yes" to continue the connection process
   - Enter the password for the local admin account (NOTE: the password will not be visible while typing)
 

- You now have access to the command line interface of this Ubuntu VM. Run a few Linux commands and observe the SSH packets being captured by wireshark throughout 
 the session. 


![image](https://github.com/user-attachments/assets/bddf8897-def5-4093-990c-6beae97eba46)



- Type "exit" into the CLI when you are ready to terminate the SSH connection

![image](https://github.com/user-attachments/assets/265c50b8-1858-43f7-ab15-7f0fa788b27e)


-------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 4.0: Initiate a New Packet Capture in Wireshark and Filter for DHCP (Dynamic Host Configuration Protocol) traffic. </h3>

- Enter either "dhcp", "udp.port == 67" or "udp.port == 68" or "udp.port == 67 || upd.port == 68" in the wireshark search filter.
   - DHCP servers use UDP port 67 for communication and DHCP clients use UDP port 68, so filtering using either port number (or both) will display the DHCP communication.
 
<h4> OPTIONAL STEP: Enter the command "ipconfig /all" in the command prompt or PowerShell and observe the IP address of the DHCP server. </h4>

![image](https://github.com/user-attachments/assets/86e053ec-724d-42b7-9cf2-8617d4e1a4ba)


- Now, enter the command "ipconfig /renew" and observe the captured DHCP traffic
   - This command simply renews the lease of the IP address that the DHCP client (our Windows 10 VM) received from the DHCP server.
   - NOTE: The same IP address that we observed from the "ipconfig /all" command for the DHCP server appears here in the captured DHCP traffic. 


   ![image](https://github.com/user-attachments/assets/bf9a1b88-1885-44e5-bbfa-86d0dabc30e9)

<h4> OPTIONAL STEP: If you would like to observe the entire DHCP process captured in Wireshark enter the command "ipconfig /release | ipconfig /renew"</h4>

- NOTE: (Don't freak out! 😅)You will temporarily lose connection to the VM via RDP as when this command is executed the VM essentially releases its IP address and other DHCP-assigned 
 network parameters and then immediately requests a new IP address and other DHCP-assigned network configurations from the DHCP server.
     - You should be automatically reconnected afterwards.
     - This will give us the chance to capture all four stages of the DHCP process known as DHCP "DORA":
        1. Discover
        2. Offer
        3. Request
        4. Acknowledge
      
![image](https://github.com/user-attachments/assets/43a3974c-f53b-439d-b05c-03de281b46e2)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

 
<h3> STEP 5.0: Start a New Packet Capture in Wireshark and Filter for DNS Traffic. </h3>

- Enter "dns" or "upd.port == 53" in the Wireshark search filter.
   - The DNS (Domain Name System) service commonly runs on UDP port 53

- From the command prompt, run the command "nslookup (domain name)" to perform DNS lookups for the IP addresses of various domains (such as google.com, facebook.com, disney.com 
   etc.)

- Observe the DNS traffic that is captured in Wireshark. 

![image](https://github.com/user-attachments/assets/1aea15f2-9143-4af9-ad63-711e37e46c99)


<h4> OPTIONAL STEP: Run the command "ipconfig /all" and observe the DNS server's IP address and notice that it is the same IP address involved in the DNS communications captured 
  in Wireshark. </h4>

![image](https://github.com/user-attachments/assets/825ab05a-84c8-42d2-80f3-a769090296c5)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 6.0: Initiate a New Packet Capture in Wireshark and Filter for RDP Traffic. </h3>

- Enter "tcp.port == 3389" into the wireshark display filter.
  - The Remote Desktop Protocol server software (running on our Windows 10 Azure VM) "listens" on TCP port 3389.
  - I have blurred the multiple appearances of the client's IP address connecting to the Windows 10 VM for privacy concerns,
    as that is the public IP address associated with the computer I used to perform this lab.


- Observe that there is a constant flow of network traffic being captured by wireshark while filtering for RDP traffic.
  - As you might have guessed, this is due to us establishing an active RDP connection to this VM from our host computer. 

![image](https://github.com/user-attachments/assets/55e9dbc7-e6e2-4be2-b1c5-381892c420fc)


------------------------------------------------------------------------------------------------------------------------------------------------------------------



<h2> Congratulations! You have made it to the end of this lab and hopefully now have a better understanding of how packet capture and traffic/protocol analysis works at a fundamental level. Happy (packet) sniffing!👃</h2>


<h3> 
 
 PS: Don't forget to delete your computing resources in Azure once you are finished with the lab to avoid incurring unnecessary costs. Just access 
 the [Azure Portal](https://portal.azure.com) and delete the resource group(s) which contain all your Azure resources.

</h3>
