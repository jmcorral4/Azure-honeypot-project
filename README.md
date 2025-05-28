# Azure-honeypot-project
Built a Windows-based honeypot VM on Azure with logging and analysis through Sentinel and PowerShell scripting. Used IPGeo API to map attacker locations. Focused on RDP attack patterns and incident analysis.

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

![1699151682330](https://github.com/user-attachments/assets/2bb58e65-9958-4f8a-8a3a-51bf937eaa29)

- <ins>Important</ins>, Allow * destination port ranges.
- You can select review + create. Once the review is approved create the VM.

Once deployed the Virtual Machine overview should look something like this <br/> 
![rSH03IG](https://github.com/user-attachments/assets/7acb3b2f-9a4a-4fd6-ae29-c0c331ec0779)

<h2> Creating our Log Analytics Workspace </h2>

- Search Log Analytics Workspace on the bar and choose *Create log analytics workspace*
- Under resource group we choose the group we created earlier
![1699152226839](https://github.com/user-attachments/assets/d4709fb7-6da3-46af-ae6e-d1db146efa4b)

- Make sure the region is set to the same one, in my case it was West US 3. Name it something simple. I went with law-honeynet.
- Review and create once it passes validation

<h2> Microsoft Defender for Cloud </h2>

- Search for Microsoft Defender for Cloud
- On the left tab we click on environment settings and select the LAW we just created
- We turn on the servers under defender plan
  ![image](https://github.com/user-attachments/assets/4596303a-cf02-4f7b-a38f-1d7e523cfdb6)

- Also all events under data collection. Hit save right after.
![image](https://github.com/user-attachments/assets/f5a739bf-2619-4014-8741-f2acf268d523)

- Now we head back to our LAW and connect it to our VM

<h2> Adding Microsoft Sentinel </h2>

- Search for Microsoft Sentinel
- Add Microsoft sentinel to our workspace which should be law-honeynet
  ![1699152700476](https://github.com/user-attachments/assets/b7f31f82-532b-4fe3-8488-112134478230)


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
  ![jLDUOUo](https://github.com/user-attachments/assets/8f3942dd-77cf-4e9f-b224-c2d9269b9769)


<h2> Disable Windows Defender Firewall</h2>

- Now we can simply search for Windows Defender Firewall with Adanced Security
- This will let us remove our firewall protection from the private and public profile of our VM.
  ![image](https://github.com/user-attachments/assets/d1ea0bb0-9200-44f9-8b95-6532c5e4ae78)

- Once removed we can confirm the VM is receiving traffic by pinging it with our host machine using the command-line prompt
  ![image](https://github.com/user-attachments/assets/2b3483c7-f423-41e0-a4ea-ccff77b440e3)

  ![tqJkrV3](https://github.com/user-attachments/assets/6e5c3016-b205-4a4f-ba3d-6927b03b26cd)

<h2> Windows Powershell ISE</h2>

 - I used a custom PowerShell script that enables us to map inbound RDP geo data to a map on Sentinel
 - Grab the script from this page. https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1
 - You can download, copy, or save to desktop.
 - Under our VM we'll search for PowerShell ISE. Once you open the application we can then open the script we just downloaded.
 ![1699153384903](https://github.com/user-attachments/assets/5bf3a197-62c7-4fc0-b1b0-1d9135b45d0c)
  - Now we need to get our API key needed for the geo locator. We can get this key for free by signing up at https://ipgeolocation.io/
  - Once you have your key you can paste it on line 2 within the apostrophes.
  ![image](https://github.com/user-attachments/assets/f7792ae2-10b6-406b-ac8e-e21fc89fa7f6)

  - Now you can press the green triangle at the top to run the script or F5. <ins> I missed this step and was here for a couple of minutes until I researched further to see what I had missed </ins>
  - This script will now grab all the events of people who failed to login via RDP. It grabs their IP address and creates the geographical data. The log will be saved in C:\ProgramData\failed_rdp.log You can also just search on your file explorer under C drive as failed_rdp.log and it will appear.


<h2> Create our Custom Log</h2>

 - Now we can go to our Log analytics worksapce and choose our LAW we created.
 - Once you choose law-honeynet we can choose tables on the left. We now create a new custom log (mma based)
  ![1699153684296](https://github.com/user-attachments/assets/9117eb8d-885a-4cc5-a2d1-8a06bb4e0ac5)
 - The file needed will be the failed_rdp.log from our VM. We simply copy it and put it on our host computer to upload it to our custom log.
 - Default settings for the record limiter screen
 - The Type for the collection path will be Windows. The path if you left the PowerShell script alone will be saved under C:\programdata\failed_rdp.log
 - Create custome log name, FAILED_RDP_WITH_GEO
 - Now we hit next and then create

<h2> Creating our map in Sentinel </h2>

 - On Azure we navigate to Microsoft Sentinel and choose Workbooks. Click on "add Workbook"
 - Next we're going to click edit on the top left and remove the current widgets.
    ![1699154282939](https://github.com/user-attachments/assets/8299338f-cad3-46cf-a1af-be65a75afafb)
 - Click on "add" and choose "add query"
 - Here we're going to add this script. </br>
   ![swaUmt3](https://github.com/user-attachments/assets/491d1da5-b92a-44bd-8fb4-33f831da733b)
 - Once pasted we're going to hit "run query"
 - Under visualization we're going to select map
    ![1699154387293](https://github.com/user-attachments/assets/93e86305-3c63-47f5-83e9-4ae2659c3350)
 - Once the log populates in Azure it should output data.
 - Mine does not show the countries from where it's getting attacked but I did see them in the PowerShell script from the VM. I had 20 attacks from Germany, 8 from Indonesia, and 1 from California.
![BXPMWCI](https://github.com/user-attachments/assets/02f57df0-c5b3-4002-9892-558a2f6e541f)


