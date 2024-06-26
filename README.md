<p align="center">
<img src="https://i.imgur.com/Mueltz2.png" alt="Live Cyberattack Map"/>
<h1>SIEM (Azure Sentinel) Live Cyberattack Map</h1>
In this tutorial, we expose a virtual machine to the internet and use Azure Sentinel & Powershell to display live cyberattacks on a map. 
<br>

  
</br>

</p>

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Network Security Groups/Firewalls
- Log Analytics Workspace
- Geolocation API
- Azure Sentinel
- Powershell

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)

<h2>High-Level Steps</h2>

- Create a Windows Virtual Machine
- Create a Log Analytics Workspace
- Set up Azure Sentinel
- Utilize Geolocation API on Virtual Machine
- Create a Custom Log
- Create a Live Cyberattack Map with Sentinel

<h2>Actions and Observations</h2>

<h3>1. Create a Windows Virtual Machine</h3>
<p>First things first, log on to Microsoft Azure and create a virtual machine. In this tutorial, the name of the virtual machine is "honeypot" and the resource group is called "Honeypotlab". Choose the Windows 10 Pro image and a size of 2 vcpus so that the virtual machine runs smoothly.</p>
<img src="https://i.imgur.com/UyQwl4M.png" alt="Live Cyberattack Map"/>
<p>In the Networking portion of the virtual machine, select Advanced under NIC network security groups (essentially a firewall), and then select "Create New" under Configure network security group.</p>
<img src="https://i.imgur.com/7txfsfL.png" alt="Live Cyberattack Map"/>
<p>Remove the default inbound rules and select Add an inbound rule to match what is shown below. The purpose of this is to make sure our virtual machine is exposed to as much traffic and threat actors as possible.</p>
<img src="https://i.imgur.com/xtXSdiG.png" alt="Live Cyberattack Map"/>
<h3>2. Create a Log Analytics Workspace</h3>
<p>After you create your virtual machine, create a Log Analytics Workspace, so that we can take logs of failed log in attempts from our virtual machine and transfer a custom log to Azure Sentinel.</p>
<img src="https://i.imgur.com/jNTofq0.png" alt="Live Cyberattack Map"/>
<p>To enable gathering logs from our virtual machine to Log Analytics Workspace, go to "Home > Microsoft Defender for Cloud | Environment Settings" to enable the following Defender plans:</p>
<img src="https://i.imgur.com/D1x0o5a.png" alt="Live Cyberattack Map"/>
<p>Then select "Data collection" on the same page and select "All Events":</p>
<img src="https://i.imgur.com/gK7j2eY.png" alt="Live Cyberattack Map"/>
<p>To connect our Log Analytics Workspace to our honeypot virtual machine, go to "Home > Log Analytics workspaces > <workspace name> | Virtual machines", and finally select Connect.</p>
<img src="https://i.imgur.com/bLD5uef.png" alt="Live Cyberattack Map"/>
<h3>3. Set up Azure Sentinel</h3>
<p>We now need to set up Azure Sentinel, which is a SIEM that will allow us to take our logs from our virtual machine and display the geo-locations of the failed log in attempts on a map. Select Azure Sentinel in the Azure search bar, and add the workspace to the newly created Sentinel.</p>
<img src="https://i.imgur.com/eXd9Nts.png" alt="Live Cyberattack Map"/>
<h3>4. Utilize Geolocation API on Virtual Machine</h3>
<p>We now need to remote desktop into our virtual machine. To do this, grab the public IP address of the honeypot and use this address on the remote desktop to log in (Note: the first time you log in, use the incorrect username & password, then log in correctly. This will show up in the next step in our logs as a failed login attempt). Go to Event Viewer > Windows Logs > Security in the virtual machine, and filter the Event ID for 4625, as these will be the logs that display "Audit Failure", meaning a failed log in attempt to the virtual machine. Double-clicking on one of these events displays details such as the username used to log in, the IP address of where they were trying to log in from, and date/time:</p>
<img src="https://i.imgur.com/yRi5zgb.png" alt="Live Cyberattack Map"/>
<p>In order to get more information out of the event logs, such as displaying location, latitude, longitude, etc., we are going to use a Geolocation API. On your virtual machine, go on Microsoft Edge and go to "ipgeolocation.io". You can take your failed log in attempt event and copy the IP address into the search bar to display the following information:</p>
<img src="https://i.imgur.com/b55RYO6.png" alt="Live Cyberattack Map"/>
<p>We will use this information to create a custom log in Azure that will be sent to Sentinel. Earlier, we allowed all inbound network traffic in the network security groups, and we are going to now allow the virtual machine to respond (outbound) to ICMP requests so that more threat actors can discover it. To do this, search for Windows Defender Firewall with Advanced Security in the Start menu, select Windows Defender Firewall Properties, and turn off the Firewall state for the Domain, Private & Public Profile.</p>
<img src="https://i.imgur.com/CweXz6H.png" alt="Live Cyberattack Map"/>
<img src="https://i.imgur.com/Go5mYkZ.png" alt="Live Cyberattack Map"/>
<p>Next, we are going to copy this Powershell script https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1 into Powershell ISE on the virtual machine. Simply copy the script and launch Powershell ISE, select File, New, and then paste it. Now save it to the virtual machine's desktop, in this tutorial the file name is "Log_Exporter".</p>
<img src="https://i.imgur.com/LOMt6wr.png" alt="Live Cyberattack Map"/>
<p>In the Powershell script, we need to replace the example API key with one for the virtual machine. This can be done by creating an account on ipgeolocation.io, in which we can copy the API key that was created from the account:</p>
<img src="https://i.imgur.com/yChsH4A.png" alt="Live Cyberattack Map"/>
<p>This will allow the logs to get live location data such as latitude and longitude. After pasting the API key into the script, run the script, and go to C:\ProgramData\failed_rdp to see the output of the script (Note: the only real data is the last line, which is our first failed log in attempt. The rest are examples of extracted data):</p>
<img src="https://i.imgur.com/pWoylt4.png" alt="Live Cyberattack Map"/>
<h3>5. Create a Custom Log</h3>
<p>After this, we are going to create a custom log in Azure by going to Home > Log Analytics workspaces > "workspace name" | Tables > Create a custom log. This will allow us to bring the custom geo-location data file from our Powershell script into the Log Analytics workspace. In "Select a sample log", go back to the virtual machine to copy the contents of the failed_rdp file, paste it on our actual computer in Notepad, and save it as "failed_rdp.log". Below is what needs to be selected for the custom log:</p>
<img src="https://i.imgur.com/Y5ATiUA.png" alt="Live Cyberattack Map"/>
<img src="https://i.imgur.com/xifHIpw.png" alt="Live Cyberattack Map"/>
<img src="https://i.imgur.com/X2DvGJf.png" alt="Live Cyberattack Map"/>
<p>Once the custom log is created, select Logs in the Log Analytics Workspace tab, and input this query:</p>
<p>FAILED_RDP_WITH_GEO_CL</p>
<p>| extend username = extract(@"username:([^,]+)", 1, RawData),</p>
         <p> timestamp = extract(@"timestamp:([^,]+)", 1, RawData), </p>
         <p>latitude = extract(@"latitude:([^,]+)", 1, RawData), </p>
         <p>longitude = extract(@"longitude:([^,]+)", 1, RawData), </p>
        <p> sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData), </p>
        <p> state = extract(@"state:([^,]+)", 1, RawData), </p>
         <p>label = extract(@"label:([^,]+)", 1, RawData), </p>
        <p> destination = extract(@"destinationhost:([^,]+)", 1, RawData), </p>
        <p> country = extract(@"country:([^,]+)", 1, RawData) </p>
<p>| where destination != "samplehost" </p>
<p>| where sourcehost != "" </p>
<p>| summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude</p>
<p>Selecting Run will return the following (Note: it may take upwards of 15-30 minutes for the custom log to sync to Log Analytics workspace):</p>
<img src="https://i.imgur.com/rm2hhAh.png" alt="Live Cyberattack Map"/>
<h3>6. Create Live Cyberattack Map with Setinel</h3>
<p>The final step is to create our live map in Sentinel. To do this go to Azure Sentinel and select our Create Log Analytics workspace, Workbooks and select New Workbook. Remove all of the default widgets, and select Add Query in the drop-down menu. Paste the same query as before and select Run Query:</p>
<img src="https://i.imgur.com/SLMgJCd.png" alt="Live Cyberattack Map"/>
<p>Now, select Map under the Visualization drop-down menu:</p>
<img src="https://i.imgur.com/w6y1jhu.png" alt="Live Cyberattack Map"/>
<p>Note that this map will be different in terms of location points depending on where failed log in attempts took place. Save the workbook as the following:</p>
<img src="https://i.imgur.com/4j5p6Du.png" alt="Live Cyberattack Map"/>
<p>The Failed RDP World Map workbook should look like this:</p>
<img src="https://i.imgur.com/NIi43Wa.png" alt="Live Cyberattack Map"/>
<p>Change the auto-refresh to some sort of time, such as every 10 minutes so that the map will refresh when threat actors start to discover the virtual machine. The live map below is what was shown after about 8 to 10 hours of keeping the virtual machine running:</p>
<img src="https://i.imgur.com/gohL1RE.png" alt="Live Cyberattack Map"/>
<p>Congratulations! You have now created a live Azure-Sentinel Map of attacks!</p>
