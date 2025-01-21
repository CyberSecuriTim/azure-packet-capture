<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1> Manipulating Network Security Groups (NSGs) and Analyzing Traffic Between Azure Virtual Machines</h1>

- In this tutorial, we will observe network traffic of varying protol types being transmitted to and from Azure Virtual Machines using Wireshark as well as experimenting with 
Network Security Groups. <br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDP, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 24.04 LTS

<h2>High-Level Steps</h2>

- Step 1: Create your Windows 10 and Ubuntu VM's (place them in the same Vnet and subnet)
- Step 2: Observe ICMP Traffic 
- Step 3: Observe SSH Traffic
- Step 4: Observe DHCP Traffic
- Step 5: Observe DNS Traffic
- Step 6: Observe HTTP vs HTTPS traffic
- Step 7: Observe RDP Traffic

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

    - Set the VM's Size to be at least 2 Virtual CPU's (vcpu's) and 8 GiB (gigabytes) of memory.
    - Assign the credentials to the local administrator account that will be created for this VM.
    - Ensure TCP port 3389 is selected as an allowed inbound port (this will enable us to establish a remote desktop connection to this VM)

![image](https://github.com/user-attachments/assets/801bc238-83a5-4da5-ae89-5135182353d9)

    
- Navigate to the "Networking" tab:
    - Create a new Virtual Network (Vnet) and subnet
    - Ignore the security warning regarding exposing the RDP port to all IP addresses on the internet we will filter IP addresses in a later step. 

  ![image](https://github.com/user-attachments/assets/f5914c09-e494-489f-9eb0-548f2d80e0a8)

- "Review + Create" and once the validation process has been passed create the VM.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.1: Harden the Windows 10 VM by Restricting the Source IP Addresses that can Connect to it via RDP </h3>

- Once the VM is created, navigate to its Network settings select the RDP rule configured in its Network Security Group ruleset.
  - Change the allowed source IP addresses from any to "My IP address" to restrict the RDP connection to only originate from your current IP address
  - Or you could select "IP Addresses" and specify an IP address(s) that you would like to connect from.
 
![image](https://github.com/user-attachments/assets/8364fb65-a882-462e-8d9c-7a6a87073f5c)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.2: Create the Ubuntu Virtual Machine. </h3>

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
       
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.3: Harden the Ubuntu VM by Restricting the Source IP Addresses for SSH Connection </h3>

- Within the VM's network settings:
  - Modify the SSH rule in the network security group by restricting the IP addresses that can establish an SSH connection to the Ubuntu VM.
  - I explicitly listed the private IP address of the Windows 10 VM (10.0.0.4)
 
  ![image](https://github.com/user-attachments/assets/9c0d2be3-b433-4bb1-a793-31cbb26c78a5)


------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.4: Observe the Topology of the created Virtual Network within the Network Wathcer </h3>

- From the home page of the Azure portal:
  - Navigate to "Virtual Network" > (Name of the virtual network)
    - Select Monitoring > Diagram

- Verify that both VM's are connected within the same subnet.

![image](https://github.com/user-attachments/assets/ec570eea-a7bc-47a9-865e-0bc1c1974adc)

------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h3> STEP 1.5: Connect to the Windows 10 VM and Install Wireshark </h3>

- Establish the remote desktop connection to the Windows 10 VM via its public IP address and your preferred RDP client.
   - Use the admin account's credentials that were provisioned during the VM's creation. 

![image](https://github.com/user-attachments/assets/58dac6bf-8124-4846-b52d-fe2adbbe58e7)


- Browse to the [wireshark download link](https://www.wireshark.org/download.html) within the Windows 10 VM.
