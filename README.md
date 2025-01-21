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
- Step 6: Observe RDP Traffic

<h2>Actions and Observations</h2>


<h3> STEP 1.0: Create the Windows 10 VM in Azure with at Least 2 Virtual CPU's (vCPU) and 8 GiB of Memory </h3>

- 
