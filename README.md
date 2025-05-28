# Azure-honeypot-project
Built a Windows-based honeypot VM on Azure with logging and analysis through Sentinel and PowerShell scripting. Used IPGeo API to map attacker locations. Focused on RDP attack patterns and incident analysis.
azure-honeypot-project/

<h2> Tools Used<h/2>

- Microsoft Azure
- Microsoft Sentinel
- Log Analytics
- Windows Defender Firewall
- Windows Powershell ISE
- IPGeolocation

<h2> Create a Resource Group</h2>

- Got to Azure Portal -> Resource Groups -> *Create a New one*
- This helps organize all the resources related to the honeypot

<h2> Deploy Virtual Machine</h2>

- I did my project using Windows 11
- Standard Size D2s v3 (2 vcpus, 8 GiB memory)
- Location, West US 3 (Zone 1)
- You'll have to set up a username and password in order to access the VM remotely
- Next we go into the Networking tab. Under NIC network security group, we select advanced.
- We delete the default rule and add an inbound rule.
<p align="center"> Inbound Security Rule </p>

!![1699151682330](https://github.com/user-attachments/assets/2bb58e65-9958-4f8a-8a3a-51bf937eaa29)

- <ins>Important</ins>, Allow * destination port ranges.
- You can select review + create. Once the review is approved create the VM.

Once deployed the Virtual Machine overview should look something like this <br/> 
![rSH03IG](https://github.com/user-attachments/assets/7acb3b2f-9a4a-4fd6-ae29-c0c331ec0779)

<h2> Creating our Log Analytics Workspace </h2>

- Search Log Analytics Workspace on the bar and choose *Create log analytics workspace*
- Under resource group we choose the group we created earlier
- Make sure the region is set to the same one, in my case it was West US 3. Name it something simple. I went with law-honeynet.
- Review and create once it passes validation

<h2> Microsoft Defender for Cloud </h2>

- Search for Microsoft Defender for Cloud
- On the left tab we click on environment settings and select the LAW we just created
-  We turn on the servers under defender plan and all events under data collection. Hit save right after.
-  Now we head back to our LAW and connect it to our VM

<h2> Adding Microsoft Sentinel </h2>

- Search for Microsoft Sentinel
- Add Microsoft sentinel to our workspace which should be law-honeynet

<h2> Remote Connect</h2>

- Now in order for me to disable the firewall I had to remote connect to the VM using Remote Desktop Connection from my local computer.
- Simply input the Public IP Address that shows from your VM overview on Azure.
- We simply use our login that we created when choosing the information on our VM.
- *You can fail the login attempt 2-3 times in order to see the attempts on our logs*
- Once you log in we'll have access to the VM.

<h2> Failed Login Attempt</h2>

- Once you're in the VM we can search for Event Viewer from the start menu
- This will have the logs from our VM. We can navigate to Windows Logs> Security.
- Here we'll see all of the Audit Success and Audit Failures that are occurring.
- If you failed your sign in attempt when logging through the RDC then you'll see the failed attempts here

<h2> Disable Windows Defender Firewall</h2>

- Now we can simply search for Windows Defender Firewall with Adanced Security
- This will let us remove our firewall protection from the private and public profile of our VM.
- Once removed we can confirm the VM is receiving traffic by pinging it with our host machine using the command-line prompt
  !![tqJkrV3](https://github.com/user-attachments/assets/6e5c3016-b205-4a4f-ba3d-6927b03b26cd)
