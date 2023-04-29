<h1>Microsoft Sentinel Live Attack Lab</h1>

<h2>Description</h2>
<b>I documented my experience of utilizing Microsoft Azure to create a virtual machine on the cloud, running Windows 10. To make this VM accessible via the internet, I employed Azure Log Analytics Workspace, Microsoft Defender for Cloud, and Azure Sentinel to collect and aggregate attack data. The attack data was then displayed on a map using Microsoft Sentinel. Throughout this project, I utilized various tools and resources, including PowerShell. Specifically, I employed PowerShell to scan the Event Viewer in the exposed VM, focusing on EventID 4625 (failed logon attempts). The PowerShell script sent this data to a logfile and also transmitted the IP addresses of failed logons to IPgeolocation.io through an API. This information could then be utilized to map the origin of logon attempts in Microsoft Sentinel. This project allowed me to gain experience with cloud resources and concepts, APIs, SIEMs, and Microsoft Azure. I learned a great deal about configuring and provisioning resources in the cloud and analyzing SIEM logs. Overall, this project was an enjoyable and rewarding experience, and I hope that readers can appreciate the work that was involved.<b/>
<br />

## Blog

Here is my blog about SIEMs and Microsoft Sentinel for more info:

https://medium.com/@mojmdevo/siems-and-microsoft-sentinel-keeping-your-organization-safe-6d377dca854e

<h2>Tools Used</h2>

- <b>Microsoft Sentinel (SIEM)</b> 
- <b>Log Analytic Workbooks</b>
- <b>Microsoft Defender for Cloud</b>
- <b>Virtual Machines</b>
- <b>Remote Desktop</b>
- <b>PowerShell</b>
- <b>API's</b>
- <b>Event Viewer</b>
- <b>Firewalls</b>

<h2>Environments Used </h2>

- <b>Microsoft Azure</b>
- <b>Windows 10</b> (21H2)

<h2>Links</h2>

- <b>Microsoft Azure Free Trial:</b> https://azure.microsoft.com/en-us/free/
- <b>IPGeolocation:</b> https://ipgeolocation.io/

<h2 align="center">Program walk-through</h2>

<p align="center">
<b>To begin, I will establish a Microsoft Azure account to serve as my cloud environment for provisioning resources. For this project, I will make use of the $200 credit that I will receive. As the resources I require are not particularly resource-intensive, I can allocate any remaining credit towards future projects. Additionally, I will use a specific website for my IP geolocation data. </b> <br/>
</p>

![Create_Azure_Subscription](https://user-images.githubusercontent.com/108043108/225348757-c41744df-2be1-4ffc-87a6-aa258a4102ef.JPG)
![Set_Up_Completed](https://user-images.githubusercontent.com/108043108/225348950-5d9c01fa-a707-4813-a6f3-012c703c41df.JPG)
![IPGEO](https://user-images.githubusercontent.com/108043108/225456045-a2a5b61c-b0ba-49a0-9552-92bf0c8d1cf1.JPG)

<br />
<br />
<p align="center">
<b>The next thing I'll do is start the process of creating my virtual machines. Provisioning a VM can be a lengthy process so while I move on to the next step, my VM can be provisioned in the background.</b> <br/>
</p>

![Create_VM](https://user-images.githubusercontent.com/108043108/225350437-f6fc6e29-6821-48d5-a379-2099477589ae.JPG)

<br />
<br />
<p align="center">
<b>During the VM provisioning process, it is essential to create a new Resource Group that will encompass all my future resources. In Azure, a resource group is a logical collection of configurations, services, tools, and other resources that fall under a common umbrella. By grouping them together, they can be created and deleted together, as they share the same lifespan. If some resources exist outside a particular resource group and I delete that group, the resources outside of it will remain unaffected. Thus, managing resources becomes more convenient when they are in one place. For this project, I have chosen to name my resource group "HoneyPot_Lab" and my virtual machine "HoneyPot-VM".</b>
<br/>
</p>

![New_Resource_Group](https://user-images.githubusercontent.com/108043108/225352474-3b252a72-20a6-4a56-be2a-6405d4f79cc4.JPG)
![Create_VM_2](https://user-images.githubusercontent.com/108043108/225353275-82a1ee46-586a-4ce1-9697-440c57528b3c.JPG)

<br />
<br />
<p align="center">
<b>Next, I navigated to the same page and selected the VM size that I wanted to provision. Initially, I chose Standard_B1s, as indicated by the green circle in the photo. However, I later decided to upgrade to Standard_B2s, which offered two CPU cores and more RAM, as the initial choice was too slow. The VM was lagging, and PowerShell kept crashing. After selecting the appropriate VM size, I created the admin account. I selected a unique admin name and generated a 30-character password comprising special characters, numbers, and a mix of lowercase and uppercase letters. Since I anticipated brute force and dictionary attacks aimed at logging into the exposed VM, a strong password was essential. At this stage, I set the public inbound port rules, essentially determining which ports would be open to connect to the VM. While multiple authentication methods, such as SSH, were available, I chose to allow only RDP, which operates on port 3389.</b> <br/>
</p>

![Create_VM_3](https://user-images.githubusercontent.com/108043108/225365594-cf2fe158-d887-4bf9-aca6-d423019404a6.jpg)

<br />
<br />
<p align="center">
<b>The subsequent step is to generate a new Network Security Group (NSG), which acts as a firewall that can generate and enforce inbound and outbound traffic regulations to Azure resources. However, for this project, we do not require any traffic regulations. We intend to allow everyone and anyone to communicate with the honeypot VM. Although there exists a default inbound rule, we will eliminate it and introduce a new inbound rule that permits EVERYTHING into the VM. In the Destination Port Ranges box, I inserted an asterisk (*) to denote Anything. This step permits all port ranges and protocols. For Priority, I designated it as 100 since the lower the number, the higher the priority. This rule will be prioritized over any rogue rule that may exist. With these settings, all traffic can pass through into our VM. However, it is critical to note that I would never implement this in an actual production environment.</b> <br/>
</p>

![Networking_NSG_1](https://user-images.githubusercontent.com/108043108/225353983-66d6e530-783f-4db8-9720-23e575719b5f.JPG)
![Network_NSG_2](https://user-images.githubusercontent.com/108043108/225367854-f6c685bb-6e96-4981-8f20-2f3a300f1987.JPG)
![Network_NSG_3](https://user-images.githubusercontent.com/108043108/225367866-761bcc17-991d-4af1-8798-db7bca2acff4.JPG)
![Network_NSG_4JPG](https://user-images.githubusercontent.com/108043108/225367879-4aff3f39-e232-4404-aa7d-30bac028fc60.JPG)

<br />
<br />
<p align="center">
<b>I review, create, and deploy the VM as the last step. </b> <br/>
</p>

![Create_VM_4](https://user-images.githubusercontent.com/108043108/225372110-b0dcf0b3-b991-41d5-8fbd-3817cdc56b8e.JPG)
![Deploying_VM](https://user-images.githubusercontent.com/108043108/225372118-70617d94-0808-4fcd-9a27-0413066d471d.JPG)

<br />
<br />
<p align="center">
<b>As my VM is deploying, I can initiate the process of configuring Log Analytics Workspace. These tools are available in the Azure home dashboard, or alternatively, you can search for them in the search bar. When I generate a Log Analytics Workspace, I ensure that it is assigned to my HoneyPot_Lab resource group, so it can be deleted when I delete the resource group. I named the instance LAW-HoneyPot.</b> <br/>
</p>

![Log_Analytics_Workspace](https://user-images.githubusercontent.com/108043108/225373030-1633fcf3-49cf-4a6f-afa7-33337a60c57c.JPG)
![Log_Analytics_Workspace_2](https://user-images.githubusercontent.com/108043108/225373041-89acebf7-1328-4b8a-96ee-6c1a946c7df0.JPG)

<br />
<br />
<p align="center">
<b>Next, I proceed to Microsoft Defender for Cloud to enroll in plans that will enable me to collect and aggregate data for Microsoft Sentinel to use later on. Additionally, I need to connect my HoneyPot-VM to Microsoft Defender so that it can collect data. Since my VM has been created, I can now connect it to these services via data connectors. In the third picture, I only enable plans for Foundational CSPM and Servers, as I am not running any SQL servers.</b> <br/>
</p>

![Microsoft_Defender](https://user-images.githubusercontent.com/108043108/225375649-d2e6bfc2-92d7-4193-af26-3526bf646744.JPG)
![Microsoft_Defender_2](https://user-images.githubusercontent.com/108043108/225375660-7f554560-6568-4265-a6b1-c5d6f7802774.JPG)
![Microsoft_Defender_3](https://user-images.githubusercontent.com/108043108/225375671-a99097cf-e19e-4037-98b6-a4f4c710eb94.jpg)
![Microsoft_Defender_4](https://user-images.githubusercontent.com/108043108/225375684-4825f9d7-a26d-4eb4-a477-ba3aa0b6a9b1.JPG)


<br />
<br />
<p align="center">
<b>Since my VM was created, I can go back into Log Analytics Workspaces and connect my VM to that service as well.</b> <br/>
</p>

![Connect_VM_To_LAW](https://user-images.githubusercontent.com/108043108/225376430-5bb177f2-a2e9-4faa-ab8b-64bd23625407.JPG)
![Connect_VM_to_LAW_2](https://user-images.githubusercontent.com/108043108/225376446-e56ef573-901b-4d52-824d-529dad6a53d5.JPG)

<br />
<br />
<p align="center">
<b>I can now create a Microsoft Sentinel resource and connect it to my VM. </b> <br/>
</p>

![Azure_Sentinel](https://user-images.githubusercontent.com/108043108/225377514-f5462e65-4b40-4823-a613-7088cfeff63a.JPG)
![Azure_Sentinel_2](https://user-images.githubusercontent.com/108043108/225377521-f4d379cf-58d7-4aca-aee0-1a80cb9aceac.JPG)

<br />
<br />
<p align="center">
<b>Once all the necessary configurations are done in the Azure dashboard, the next step is to set up the virtual machine itself. To begin with, the first thing is to obtain the public IP address of the VM, which is required to RDP into it. This can be done by accessing the Virtual Machines tab in Azure and selecting the HoneyPot VM. The Public IP address of the VM is highlighted and can be noted down for future reference.</b> <br/>
</p>

![Getting_VM_Public_IP](https://user-images.githubusercontent.com/108043108/225379998-5bcad4c5-c878-4249-8384-34e1581dc35d.JPG)

<br />
<br />
<p align="center">
<b>From here I can log into my VM via Remote Desktop. I open up Remote Desktop on my PC, enter the public IP address and credentials needed and connect in! I configure the HoneyPot.</b> <br/>
</p>

![Connect_With_RDP](https://user-images.githubusercontent.com/108043108/225381141-3514bcc6-3417-4281-ad67-1b4af20edfd5.JPG)
![Logging_Into_VM_Via_RDP](https://user-images.githubusercontent.com/108043108/225381164-d6591c73-4cf0-4fee-85c9-22d56abfc8ab.JPG)
![Logging_Into_VM_via_RDP_2](https://user-images.githubusercontent.com/108043108/225662102-2a73273a-c0f7-4424-833f-2d981f5859e0.JPG)

<br />
<br />
<p align="center">
<b>Once I successfully log into the VM via RDP, I access the Event Viewer which logs every activity that occurs in a Windows system. Each action is assigned an EventID for ease of navigation. For this project, I'm interested in monitoring EventID 4625, which tracks Failed Logon Attempts. To view these logs, I navigate to the Security tab. In the images below, I initiate another RDP session and attempt to log in with an incorrect password. This results in a failed logon attempt that is recorded and logged in Event Viewer.</b> <br/>
</p>

![Event_Viewer](https://user-images.githubusercontent.com/108043108/225381724-2603d72a-5334-479d-b555-c1d0baaeb875.JPG)
![Event_Viewer_2](https://user-images.githubusercontent.com/108043108/225381737-0bc2d129-b045-4099-a5fe-d34b4d43f673.JPG)


<br />
<br />
<p align="center">
<b>The next step is the ping the VM from my native computer. I do this to see if I can make a direct connection to the VM and see if it is currently discoverable. It is not. That is because the VM has Windows Defender Firewall activated. The firewall is blocking ICMP Echo Requests, making the VM undiscoverable. I know this because I pinged the VM's public IP address which is 74.235.173.155 and I see Request timed out a few times. The firewall is dropping the ICMP request packets.</b> <br/>
</p>

![Ping_VM](https://user-images.githubusercontent.com/108043108/225383503-2a599790-511e-47ec-b858-4055f42ead0c.JPG)

<br />
<br />
<p align="center">
<b>To ensure that my VM is visible to everyone on the internet, I disable the Windows Firewall by accessing it through the Windows search bar and searching for wf.msc. From there, I disable all firewall settings. I then proceed to open CMD on my local machine and ping the VM again, and this time it responds because the firewall is no longer blocking ICMP requests. </b> <br/>
</p>

![Turn_Off_Firewall](https://user-images.githubusercontent.com/108043108/225385096-6cdeaa2a-3411-46fa-bad1-0afcee031860.JPG)
![Turn_Off_Firewall_2](https://user-images.githubusercontent.com/108043108/225385112-d4ada032-3cdc-4650-b60a-0a373cddfd17.JPG)

<br />
<br />
<p align="center">
<b>With the VM now exposed to the internet, I can proceed with setting up the PowerShell script which is at the core of this project. This script has the ability to parse Event Viewer and look for EventID 4625. Upon finding such events, it extracts the IP address associated with the failed logon attempt and uses an API to send it to IPgeolocation.io for geolocation data retrieval. This was necessary because the IP address alone in event viewer does not include any geographical location. Instead of building a solution from scratch, I chose to use a system that is dedicated to retrieving and sending back geolocation data. The PowerShell script then collects all the geographical data and saves it as a string in a logfile called failed_rdp.log. I will later use this logfile in conjunction with Microsoft Sentinel to map attacks live.</b> <br/>
</p>

![Create_PS_Script](https://user-images.githubusercontent.com/108043108/225409638-0f9735a1-eca0-446f-afb7-74ef0d427ebf.JPG)


<br />
<br />
<p align="center">
<b>Once I open PowerShell, I copy and paste the script that was prepared before the start of the project. Then, I save the script on the desktop as "Log_Exporter". At this point, it is recommended to create a profile on IpGeolocation to obtain an API key. The API key should be pasted into the PowerShell script. It is important to note that the free API calls are limited to 1000, so it's advisable to turn on the Windows Firewall settings until the setup of Log Analytics Workbooks Custom Fields is completed. Once the custom fields are configured, the firewall can be turned off.</b> <br/>
</p>

![Create_PS_Script_2](https://user-images.githubusercontent.com/108043108/225410802-01a83b34-e79a-4516-8bdb-70c01baa76d7.JPG)
![Create_PS_Script_3](https://user-images.githubusercontent.com/108043108/225410821-7078f7c1-1928-49c4-8e03-a123ef43cb3f.JPG)

<br />
<br />
<p align="center">
<b>Upon running my Log_Exporter script, the output is formatted in pink and black for easier readability. Note that the API_KEY displayed has been changed for security reasons. In the first picture, the script appears to be working as intended, with the output displaying the first failed login from earlier. The second picture shows how the data is saved into the failed_rdp logfile in string format. To improve the precision of AI in Log Analytics Workbooks and Microsoft Sentinel, I have included some sample data in this file. It is important to note that the more data that is included, the more precise the results will be.</b> <br/>
</p>

![Run_PS_Script](https://user-images.githubusercontent.com/108043108/225412357-ee390c08-c131-4658-b6d0-b2c6f0485850.JPG)
![Training_Sentinel](https://user-images.githubusercontent.com/108043108/225412372-30c65d96-6678-4ac1-a3a9-d0f3d8331342.JPG)

<br />
<br />
<p align="center">
<b>At this point, another failed logon attempt was intentionally made to test the functionality of the PowerShell script. The log showed that the VM was being targeted by someone attempting to brute force it from Tunisia. Despite the potential risks, the data from this attack was valuable for training the AI in Azure. However, the free API from IPgeolocation.io only provided 1000 calls, which were quickly used up by the attacker from Tunisia. To continue with the project, an additional $15 was spent to purchase an extra 150k API calls.</b> <br/>
</p>

![Showing_PS_Script_Someone_Already_Found_VM](https://user-images.githubusercontent.com/108043108/225413356-343d9ae2-240d-4841-b41f-62cff77e51ab.JPG)

https://user-images.githubusercontent.com/108043108/225419861-a5c9bce1-3f6d-42e9-9bd4-dcd7ca816a33.mp4

<br />
<br />
<p align="center">
<b>Once the sample log data is imported into Log Analytics Workbooks, I can create a new query to extract the relevant fields and data from the failed_rdp log. The query will look for EventID 4625 and parse out the IP address, username, and time of the failed logon attempt. This data will be used to create visualizations and alerts in Log Analytics and Microsoft Sentinel. Once the query is created, I can save it as a custom log search so I can easily access it in the future. This custom log search can also be used to create visualizations in Log Analytics Workbooks.</b> <br/>
</p>

![Custom_Logs](https://user-images.githubusercontent.com/108043108/225421446-5a0c5c4f-62a0-4a07-bb6a-b36edd43b7fd.JPG)
![Custom_Log_2](https://user-images.githubusercontent.com/108043108/225421932-a4d520b3-5f9a-4851-badb-e07744735144.JPG)
![Custom_Log_3](https://user-images.githubusercontent.com/108043108/225421947-dcaadf65-e212-41ae-8e83-322a310df6f7.JPG)
![Custom_Log_4](https://user-images.githubusercontent.com/108043108/225421956-8dd57a76-b0b0-44a0-8f10-675ba6b1dfa9.JPG)
![Custom_Log_5](https://user-images.githubusercontent.com/108043108/225421995-664b1d05-5ba7-48a3-9e90-a1c5eedeed36.JPG)

<br />
<br />
<p align="center">
<b>After creating the custom log in Log Analytics, we need to test that it is properly collecting data from the failed_rdp.log file. To do this, we navigate to the Logs section in Log Analytics and run a query to retrieve data from our custom log. We use the following query: 

`FAILED_RDP_WITH_GEO_CL`

This query retrieves all data from our custom log. If it retrieves data, we know that our custom log is properly set up and collecting data. We can then use this data to create visualizations and alerts in Log Analytics Workbooks. </b> <br/>
</p>

![Custom_Log_6](https://user-images.githubusercontent.com/108043108/225422804-46df9880-9734-420c-84a9-e18ec3a68b57.JPG)
![Custom_Log_7](https://user-images.githubusercontent.com/108043108/225423680-580feffc-72e3-4eaf-9fd0-57d0376976b3.JPG)
![Custom_Log_8](https://user-images.githubusercontent.com/108043108/225424273-b392d383-4025-4206-b406-0fd79058d263.JPG)

<br />
<br />
<p align="center">
<b>Once the custom log is created, it may take some time for the data to sync from the VM to Log Analytics. While waiting for the data to sync, I decided to check the Event Viewer logs which should have already been synced. As shown in picture 1, the Event Viewer logs are displaying properly. After some time, I queried the newly created FAILED_RDP_WITH_GEO custom log and confirmed that it was showing data, indicating that the VM and Log Analytics were synced and data was being sent and received. </b><br/>
</p>

![Security_Event_4625](https://user-images.githubusercontent.com/108043108/225425000-75d4b1ae-fa60-48a4-af30-8fc5ab52cb8f.JPG)
![Failed_RDP_LOGS](https://user-images.githubusercontent.com/108043108/225425211-162958fe-13cb-455a-99bf-2b24897a5a29.JPG)

<br />
<br />
<p align="center">
<b>To use the data in Microsoft Sentinel, it's necessary to extract the fields used in the log. To do this, I right-click on a failed RDP login log that has all the raw data from the PowerShell script and highlight the data needed for each field. I then name each field to correspond to the data it will contain. After the extraction is completed, the Log Analytics AI examines all other sample data and actual logs generated to see if it can correctly pull the data for each field. If there are errors, I have to go in and correct them by re-highlighting the correct data point the AI should use. This process requires me to scroll through the list of data points, right-click any incorrect ones, and re-highlight the correct information for each field. The custom fields that need to be created at this stage are: latitude, longitude, destinationhost, username, sourcehost, state, country, label, and timestamp.</b><br/>
</p>

![Extract_Fields](https://user-images.githubusercontent.com/108043108/225436517-0a82f48e-82c8-45cf-b351-634ee8ba41c7.JPG)
![extract_data_2](https://user-images.githubusercontent.com/108043108/225436525-d57e194f-8a55-4e9a-915b-9d8a413a83e4.JPG)
![extract_data_3](https://user-images.githubusercontent.com/108043108/225436533-e68158a6-3cd5-4f84-a6b2-1329a1339016.JPG)
![extract_data_4](https://user-images.githubusercontent.com/108043108/225436540-4ec75b85-8076-493e-9fb7-8c5bc24cba4b.JPG)


<br />
<br />
<p align="center">
<b>It can be frustrating when certain things don't work as expected, despite multiple attempts to resolve the issue. However, deleting the problematic field and moving on without it was a reasonable solution. Sometimes, it's better to work with what you have and continue moving forward rather than getting bogged down trying to fix something that may not be critical.</b>  <br/>
</p>

![Sourcehost_wouldnt_update](https://user-images.githubusercontent.com/108043108/225451005-edabe896-94b1-4d11-8853-42e4a4ce8e83.JPG)
![sourcehost_wouldnt_update_2](https://user-images.githubusercontent.com/108043108/225451011-80b1dbad-2e90-4407-bfa5-5c516b66a991.JPG)

<br />
<br />
<p align="center">
<b>The subsequent step involved setting up a geomap to plot and visualize the location of the login attempts. This was accomplished by accessing Microsoft Sentinel. The initial image displays that the SIEM had appropriately accumulated and categorized the data. While no alerts were set for this project, it was a possibility for a future endeavor. The snapshot shows that there were approximately 10k events and 6.9k security events, with 2.3k failed RDP attempts originating from the Event Viewer in the VM. Despite not completing the project, the individual from Tunisia had already made  several attempts to brute force into the VM.</b>  <br/>
</p>

![Setting_Geomap](https://user-images.githubusercontent.com/108043108/225451297-e0c57af2-5333-4b6f-8fff-79b11b816e77.JPG)

<br />
<br />
<p align="center">
<b>The next step in creating the map is to create a new workbook in Sentinel. To do this, you would need to navigate to the workbooks section and delete any default graphs or widgets that are already present. Once you have cleared the space, you can start creating the new workbook by selecting the appropriate template.</b>  <br/>
</p>

![setting_geomap_2](https://user-images.githubusercontent.com/108043108/225452457-2c805584-bee1-4c4b-adb3-aa73726c0d0f.JPG)
![setting_geomap_3](https://user-images.githubusercontent.com/108043108/225452473-99fd3ef3-04ff-49e6-aecf-94793b7e60f2.JPG)
![setting_geomap_4](https://user-images.githubusercontent.com/108043108/225452571-ef8a7a55-a541-44ba-9269-24a32d56d14c.JPG)

<br />
<br />
<p align="center">
<b>In order to create the map, a query needs to be added to the workbook. The query will specify the dataset to be used by Microsoft Sentinel by pointing out the necessary fields from Log Analytics. The query will also exclude data points that include "Samplehost" since these are not real attacks and should not appear on the map.</b>  <br/>
</p>

![setting_geomap_5](https://user-images.githubusercontent.com/108043108/225453225-24b50358-ac8c-4897-9e44-8d21f4075239.JPG)

<br />
<br />
<p align="center">
<b>Once the data points were queried, the next step was to choose how to visualize them. In this case, a map was selected. The map was configured to plot the attacks by latitude and longitude, rather than by country due to some of the attacks not including the country information. The metric settings were adjusted to make the bubbles larger based on event counts, so the larger the number of events, the larger the bubble. Once the settings were applied, the map was complete!</b>  <br/>
</p>

![setting_geomap_6](https://user-images.githubusercontent.com/108043108/225453707-62496864-0c33-4d3d-988c-3260b07e9c9b.JPG)
![setting_geomap_7](https://user-images.githubusercontent.com/108043108/225454149-b35a2412-574f-474a-acca-3b541148208a.JPG)
![setting_geomap_8](https://user-images.githubusercontent.com/108043108/225454159-593b96e9-e6dc-4cbc-b6fa-bcb955495bfd.JPG)


<br />
<br />
<p align="center">
<b>It's impressive how quickly the attacks piled up, with a total of 10,529 failed login attempts recorded before the project was stopped. Only a small fraction of those attempts were from the USA, with the majority coming from Tunisia and Cambodia. It's clear that the attackers were using automated brute-forcing software to try different password combinations and usernames. This emphasizes the importance of using strong passwords and unique usernames to minimize the risk of successful attacks. The second picture shows the number of API calls made that day, which had to be upgraded due to the volume of data being processed. </b>  <br/>
</p>

![Stopped_Because_of_API_Calls_Limit_At_150k](https://user-images.githubusercontent.com/108043108/225454550-20b2f3d6-5593-44cf-8bbc-290fb941da8c.JPG)
![API_requests](https://user-images.githubusercontent.com/108043108/225455054-0576753d-b3d0-40dd-aa4b-70246b1616e8.JPG)

<br />
<br />
<p align="center">
<b>Deleting the resource group is a crucial step in managing your cloud resources and avoiding unnecessary costs. It's important to be mindful of your spending limits and only keep resources that are actively being used. By organizing everything under one resource group, you can easily delete all the resources associated with a particular project. This helps keep your Azure environment clean and efficient.</b>  <br/>
</p>

![Deleting_Resources](https://user-images.githubusercontent.com/108043108/225455713-94e78229-b9ec-4f2e-ae17-24ed2636f7a1.jpg)
![deleting_resources_2](https://user-images.githubusercontent.com/108043108/225455718-29980ca4-9976-4abf-955f-573f6d55fdd9.JPG)

<br />
<br />
<h2 align="center">Decided To Run The Lab Again.</h2>
</p>

<br />
<br />
<p align="center">
<b>I opted to repeat the lab experiment, with the intention of allowing attackers more time to target the exposed VM, in the hopes of showcasing a greater variety of locations on the world map. My strategy proved successful, as this time a wider range of threat actors targeted the VM, resulting in a more diverse and detailed world map. I was happier with the outcome as more data was gathered.</b>  <br/>
</p>

![worldmap2](https://user-images.githubusercontent.com/108043108/226672121-d80091a0-6f7b-4f1d-b5bd-d9416823c6bf.JPG)

<br />
<br />
<p align="center">
<b>During my second attempt at the lab, I encountered an issue where my PowerShell script would intermittently stop updating. Even though the script was still running, no new entries were being generated for 5 to 15 minutes. I found this to be odd, given that I knew there were many different countries attempting to brute-force their way into the VM, so there should have been a consistent flow of new entries. Moreover, I had set the refresh rate in the PS script to 300 milliseconds, down from 1 second in my earlier attempt. To understand the root cause of this issue, I decided to investigate further. As a first step, I wrote queries in Log Analytics Workbook to compare the number of security events with Event ID = 4625 against the number of Failed_RDP_Custom_Log’s generated.</b>  <br/>
</p>

![Security_Event_ID](https://user-images.githubusercontent.com/108043108/226673169-7b2f64a3-7ab1-4918-98e8-b680a553aabf.jpg)
<p align="center"><i>After writing my query we can see here that there was a total of 9282 EventID 4625 Security Events. This means that threat actors attempted to log into my VM 9282 times.</i></p>  <br/>

![Failed_RDP_Log](https://user-images.githubusercontent.com/108043108/226673194-787f3452-309d-4d0b-a375-c653fb16c4b1.JPG)
<p align="center">
<i>After writing this query we can see it returned a total of 3892 records, which means that my Custom Log only captured 3892 login attempts. Quite a bit away from 9282.</i>
</p>

<br />
<br />
<p align="center">
<b>Upon noticing a discrepancy between the number of security events and the custom log entries, I decided to investigate the issue. After comparing the numbers and finding them to be significantly different, I looked at my API usage for the day and found it to be much closer to the number of custom log entries. Based on this, I concluded that the issue was not with my VM failing to send data to the custom log.</b>  <br/>
</p>

![API](https://user-images.githubusercontent.com/108043108/226676332-d906603b-59c1-4ad0-94c0-fb665bb26a73.JPG)

<br />
<br />
<p align="center">
<b>Lastly, I chose to go into Microsoft Sentinel and see what the dashboard is showing. My PowerShell Script was still running in the background so the numbers aren't an exact match because these screenshots were taken a few minutes apart. Again, it is showing a number (4,100) that is much closer to the FAILED_RDP_CUSTOM_LOG query.</b>  <br/>
</p>

![Sentinel](https://user-images.githubusercontent.com/108043108/226676717-a08eb2c2-6ae1-46f7-8d78-7359b2d610bf.JPG)

<br />
<br />
<h2 align="center">Conclusions</h2>

<br />
<br />
<p align="center">
<b>What I believe is occurring is that the Windows Event Viewer is functioning correctly. It is properly logging ALL login attempts and displaying the actual number of attempts that have occurred. The weak point in this issue is the PowerShell script. The reason why my PowerShell script was stopping for lengthy periods is due to being overwhelmed. Despite setting the refresh rate to 300 Milliseconds, the number of login attempts from several different attackers was significantly more than 1 attempt per 300 Milliseconds. My PowerShell script was the bottleneck. If it could capture each attempt as it occurred, it would show the actual number of attempts and be consistent with the Event Viewer. However, it could only capture 1 attempt per 300 Milliseconds, leading to some login attempts being lost or postponed (although I doubt it was postponed), resulting in the difference we are seeing. In the future, I may consider lowering the refresh rate even more to allow for more login attempts to be recorded.</b>  <br/>
</p>


<h2>There are several steps that can be taken to improve the Sentinel home lab:</h2>

<p> 
Optimize PowerShell script: One way to improve the PowerShell script is to optimize it so that it can handle a larger number of login attempts. This can be achieved by reducing the refresh rate, improving the code efficiency, and using multithreading to handle multiple connections at once.

Expand data collection: In addition to collecting login data, it may be useful to collect other types of data, such as firewall logs, network traffic data, and application logs. This can provide a more complete picture of the activity on the VM and help identify potential threats.

Configure alerts and notifications: Setting up alerts and notifications can help identify potential threats in real-time. This can be done by configuring alerts in Sentinel for specific events or by integrating Sentinel with other security tools.

Implement threat intelligence: Incorporating threat intelligence feeds can help identify known malicious IP addresses and other indicators of compromise. This can help prioritize and streamline the investigation process.

Regularly review and analyze data: It is important to regularly review and analyze the collected data to identify patterns and potential threats. This can be done by using machine learning and other advanced analytics tools.

Continuously update and improve the lab: Cybersecurity threats are constantly evolving, so it is important to continuously update and improve the Sentinel home lab to stay ahead of potential threats.</p>

<br />
<br />

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
