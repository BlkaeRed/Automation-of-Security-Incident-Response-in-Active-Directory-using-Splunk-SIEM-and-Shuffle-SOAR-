# Introduction

This project involved building an automated system for real-time incident alerting and response across both Windows (specifically Active Directory Domain Controllers and hosts) and Linux environments. It consists of several EC2 instances hosting a Splunk server, a Domain Controller, and two monitored hosts. The system uses free and open-source tools, with Splunk acting as the SIEM platform for incident analysis and alert generation. Sysmon and Syslog are used for detailed log collection.
Additional tools include Shuffle, which enables automated responses to alerts—such as sending dedicated messages or emails to a SOC analyst via Slack or Gmail. There is also an option to respond to incidents by, for example, disabling a user account. I tested this system against two attack scenarios: unauthorized access from an unknown IP address to one of the endpoints and potential port scanning, simulated using the Invoke-AtomicRedTeam framework.
The final result is a highly effective system for host monitoring, alerting, and responding to potential incidents, suitable for day-to-day operations in a SOC environment.

# Tools used

AWS – Hosting all EC2 instances  
Splunk – SIEM platform and alert generation  
Sysmon – Log collection on Windows  
Syslog – Log collection on Linux  
Shuffle – Automation playbooks and response handling  
Slack – Message delivery for alerts to SOC analysts  
Invoke-AtomicRedTeam – Environment testing and attack simulation  

# Acquired skills

Hosting VMs using AWS  
Creating and managing a simple Active Directory Domain  
Installing and configuring Splunk for alerting  
Writing custom alert rules in Splunk  
Configuring both Windows and Linux hosts to send logs to Splunk  
Automating parts of incident response using Shuffle (e.g., sending messages to Slack and emails to Gmail with response options)  
Integrating multiple security tools for comprehensive monitoring and protection  
Installing and configuring Sysmon based on specific needs  

# Project

## System Diagram
This is how the system looks and operates in practice. All telemetry data is sent to Splunk. If an alert is triggered, it is passed to Shuffle, which then sends it to both Slack and the SOC analyst’s email. The analyst then has the option to disable the user account, which also generates a follow-up message in Slack.
![Image](https://github.com/user-attachments/assets/f58d0f93-7f00-4d0e-95cc-2ca93c467f37)

## Installing and Configuring Active Directory

After creating the machines shown in the system diagram, I began setting up Active Directory.
One Windows EC2 instance (the one with more resources) was designated as the Domain Controller. Using Server Manager, I selected Add roles and features:
<img width="945" height="495" alt="Image" src="https://github.com/user-attachments/assets/d6580d96-2baa-48ea-b171-9a1d9ffc5062" />

Then selected the installation type:
<img width="945" height="668" alt="Image" src="https://github.com/user-attachments/assets/5234c761-459d-4429-83c5-aadc65236b36" />

And chose Active Directory Domain Services in the "Select server roles" panel:
<img width="945" height="667" alt="obraz" src="https://github.com/user-attachments/assets/0b708ff4-f696-4eaf-b27e-f284147bb534" />

After all of that was done installing, I promoted the machine to Domain Host by clicking the alert in right corner and after that:
Creating new forest named "HKdomain.local":

<img width="945" height="693" alt="obraz" src="https://github.com/user-attachments/assets/e36d8836-82c0-481d-94e2-feb65a744508" />

I then set a password. (Note: For this project, the password was kept short for simplicity, which is not recommended in a production environment):
<img width="945" height="689" alt="obraz" src="https://github.com/user-attachments/assets/218951a4-0997-4f0c-a332-24c8c12054a6" />

Next, thing to do was to pass "Prerequisites check" and install all necessities:
<img width="945" height="700" alt="obraz" src="https://github.com/user-attachments/assets/2aa2ee03-4443-4571-9990-d9ee00a4675c" />

And after all of that, and machine restart, the Domain Controller setup was complete.

### Joining Another Machine to the Domain
Next, I created a new user in the local domain and configured the second Windows machine to join the domain:
<img width="945" height="663" alt="obraz" src="https://github.com/user-attachments/assets/89eb2306-2da9-4438-a7d8-900c50d5fce1" />

Again, a simple password and "Password never expires" were chosen for simplicity (not recommended in real-world deployments):

<img width="845" height="738" alt="obraz" src="https://github.com/user-attachments/assets/6fb9c3a8-3f03-4472-aa46-fbb777ad0a52" />

After that I checked if user was created:
<img width="703" height="77" alt="obraz" src="https://github.com/user-attachments/assets/5193d1fd-3067-486e-9787-1feb24555a4e" />

So, only thing left to do was to configure second machine:
First, I changed the DNS server to Domain Controller's IP address:
<img width="945" height="1309" alt="obraz" src="https://github.com/user-attachments/assets/25bbb714-4786-42e5-afc5-658579292346" />

And in the "System Properties" I changed the "Member of" to my local domain:
<img width="828" height="892" alt="obraz" src="https://github.com/user-attachments/assets/12f90903-0369-4ed0-b41c-0292a87453c7" />
<img width="570" height="320" alt="obraz" src="https://github.com/user-attachments/assets/98d1000d-bc2a-4322-907b-4b29eb1c151d" />

So, after the restart, the machine is now officaly part of HKdomain.local
But before I can connect to this machine using newely created user, I needed to change which user are allowed remote access, in "System Properties":


<img width="945" height="836" alt="obraz" src="https://github.com/user-attachments/assets/cbb1e99f-6a6f-4e58-bad6-aa2c669e7757" />

And after all of that:

<img width="791" height="270" alt="obraz" src="https://github.com/user-attachments/assets/38a6f2ec-404d-4ce1-89bf-40e0c6a3eef3" />
<img width="945" height="612" alt="obraz" src="https://github.com/user-attachments/assets/d24a4d00-302e-44ae-843c-09030d4538ad" />

Active Directory is done


## Installing and Configuring Splunk

The next step after configuring Active Directory was setting up Splunk SIEM and making sure all hosts forwarded their logs to it.
I downloaded the installation package on the Linux server and followed the official tutorial from Splunk documentation.

 After installation, I executed splunk_start and opened the instance in the browser:
 <img width="1919" height="907" alt="Zrzut ekranu 2025-06-02 140217" src="https://github.com/user-attachments/assets/521dd201-4faa-42a7-a793-c5419d0f98c9" />

Then, I downloaded the Splunk Add-on for Windows (for better log parsing):
<img width="1895" height="846" alt="Zrzut ekranu 2025-06-02 140440" src="https://github.com/user-attachments/assets/46f07b6f-ec24-4657-acb7-17af7ce8fec8" />

Lastly, I enabled the listening port for log transfers and created the hk-domain index:
<img width="1919" height="908" alt="Zrzut ekranu 2025-06-02 140622" src="https://github.com/user-attachments/assets/ff6d30ab-91b1-4e88-8637-5e1a12f45fea" />
<img width="1894" height="154" alt="Zrzut ekranu 2025-06-02 151443" src="https://github.com/user-attachments/assets/23d184ac-38ad-40c8-8eeb-b2253b650b99" />


Now that everything was set, I configured both Windows and Linux machines to forward logs to Splunk.
On both machines:

Downloaded Universal Forwarder:
<img width="945" height="57" alt="obraz" src="https://github.com/user-attachments/assets/d0a61df7-5d66-46db-81ef-9401a6c5ed2c" />

Setup credentials and receiving index:
<img width="945" height="715" alt="obraz" src="https://github.com/user-attachments/assets/bc445d29-c29a-4ac5-80c7-2ec631fb7fb1" />
<img width="945" height="745" alt="obraz" src="https://github.com/user-attachments/assets/383656af-75ac-4b5a-8a3b-6ea65df30325" />

Then, I modified input.conf in C:\Program Files\SplunkUniversalForwarder\etc\system\local\ to forward Security logs into the hkdomain-ad index and changed SplunkForwarder service properties to use the Local System account:

<img width="433" height="127" alt="obraz" src="https://github.com/user-attachments/assets/dc7a71cb-d4ce-4baf-b763-8277c34eae47" />
<img width="789" height="908" alt="obraz" src="https://github.com/user-attachments/assets/97a3eb8c-e524-4a4d-aa56-225d5943e872" />

To verify everything worked, I searched for logs in the hkdomain-ad index:
<img width="1915" height="799" alt="Zrzut ekranu 2025-06-02 151550" src="https://github.com/user-attachments/assets/c3826724-cb25-490e-8df8-c3880fb89e92" />
<img width="1105" height="338" alt="Zrzut ekranu 2025-06-02 152931" src="https://github.com/user-attachments/assets/3beaa0a7-4374-422f-aacc-93bc0a654b77" />

Now to create alert unknown/unauthorized access I used this query and simulated unauthorized access using VPN:
<img width="1918" height="382" alt="obraz" src="https://github.com/user-attachments/assets/6a309e55-0f20-4cb7-84c1-58687a276014" />

Explaination of alert:
Event code 4624 represents succesful login->what I wanted to detect 

Logon_Type 7 or 10 represent either logging back into already logged session (7) or connecting remotly to host (10)-> both can be seen during remote connections

Source_Network_Address!=....-> I only want to alert when somebody outside of network (blocked addresses are my personal public IP) connects, "-" is there to only check connections with source address present

stats count by .... -> used to extract only relevant information, in other words, who connects, when they did and to what host and user

Next, to make it into alert, I presses "Save as->Alert" option, I set it up with cronjob option to trigger alert every one minute if event happend in span of 1 hour. All of this was purely for testing purposes and should not be done in professional settings:
<img width="993" height="600" alt="obraz" src="https://github.com/user-attachments/assets/1afd7c92-af48-44ba-8b94-2f478ee25801" />

At the end I also added webhook, which will be used later with Shuffler for automatic incident response:
<img width="945" height="333" alt="obraz" src="https://github.com/user-attachments/assets/8cb0145e-c646-4cbe-a7a4-f35f03ebbfef" />

Last thing to do was to check if the alerts trigger when needed and fortunatly they did:
<img width="1918" height="361" alt="Zrzut ekranu 2025-06-02 161447" src="https://github.com/user-attachments/assets/eceefe67-cc65-482a-bf29-3928b8db73d1" />

Next, I configured SplunkForwarder for my Linux server:
I mostly used this blog for help with configuration [Setting up the splunk universal forwarder on kali linux](https://iritt.medium.com/setting-up-the-splunk-universal-forwarder-on-kali-linux-for-your-cybersecurity-home-lab-c153d19215dc)

In the same way as hkdomain-ad index, I created hk-linux index used for all logs from my Linux server
Next I installed the SplunkForwarder, started it and added my Splunk server using as a forward server
After that, I added inputs.conf into /opt/splunkforwarder/etc/system/local/ directory with following contents:

<img width="378" height="114" alt="Zrzut ekranu 2025-06-09 150519" src="https://github.com/user-attachments/assets/c1eda50b-2cf0-4f5c-bd36-33a3357bdeae" />

And this is content of /opt/splunkforwarder/etc/system/local/outputs.conf after adding the forward-server:

<img width="489" height="180" alt="Zrzut ekranu 2025-06-09 150538" src="https://github.com/user-attachments/assets/670a8e47-57a9-4595-91cf-0c608d01bd7c" />

Last thing do to to check if the setup works, so I used query for index hk-linux:
<img width="1888" height="863" alt="Zrzut ekranu 2025-06-09 151011" src="https://github.com/user-attachments/assets/ac4d0ff7-f968-485f-b7a4-e09661a0541c" />

Initial Splunk setup is done.

## Slack and Shuffle automation

To begin, I created free accounts on both Slack and Shuffle. Shuffle serves as the automation platform for this integration, while Slack is used as the notification channel for SOC analysts.
I created a playbook in Shuffle with the following structure:

<img width="1469" height="685" alt="Zrzut ekranu 2025-06-05 190036" src="https://github.com/user-attachments/assets/7ee41f0e-742d-4e57-b7e9-03fd8e23f9ef" />

The first element in the playbook is a Webhook trigger named Splunk Alert. Its role is to receive alerts from Splunk, acting as the entry point for automated response

<img width="336" height="741" alt="Zrzut ekranu 2025-06-05 190054" src="https://github.com/user-attachments/assets/a0699e38-e0a8-43fa-adb8-d45cf45a529b" />

The URL generated here was used during Splunk alert configuration (as described in the previous section of the project).

The next step, "Alert Notification", posts a message to a designated Slack channel. The channel ID was taken directly from the Slack URL.

<img width="326" height="741" alt="obraz" src="https://github.com/user-attachments/assets/3d547399-265d-4e1f-b757-3d6efda9ad83" />

The values in $exec.result.* were discovered after manually triggering an alert while the webhook was active. Shuffle captured and parsed the payload automatically:

<img width="320" height="200" alt="Zrzut ekranu 2025-06-05 190123" src="https://github.com/user-attachments/assets/04e97ba5-8809-4627-a41b-c72c4ff1e1a4" />

Resulting alert in Slack:
<img width="848" height="231" alt="obraz" src="https://github.com/user-attachments/assets/9b6594a2-297e-422a-a936-089ff3733c40" />

This step sends an email to the SOC analyst. It includes details of the alert and allows the analyst to approve or deny a response action.

<img width="335" height="559" alt="obraz" src="https://github.com/user-attachments/assets/d0ccb9c6-fdf7-4497-a2c2-7a16da6281f3" />

Resulting email:
<img width="1514" height="444" alt="obraz" src="https://github.com/user-attachments/assets/d4caf8a5-5b05-4cdf-adef-27ef2bc6d2d0" />


If the SOC analyst chooses to disable the user, Shuffle executes the "Disable User" block, which requires administrator access to a domain controller via Auth5:

<img width="327" height="442" alt="Zrzut ekranu 2025-06-05 190244" src="https://github.com/user-attachments/assets/c7860c05-af58-4457-a7ae-e794a3a5a849" />

After that, all remaining elements are there to give additional message, depending on if SOC analyst decides to disable user or not.

User Attributes block retrieves the current attributes of the targeted user:  

<img width="324" height="439" alt="Zrzut ekranu 2025-06-05 190255" src="https://github.com/user-attachments/assets/200278d9-ce58-4f3f-b82a-920bd462294b" />

A Condition block checks whether the userAccountControl field indicates a disabled account:  

<img width="319" height="425" alt="Zrzut ekranu 2025-06-05 190306" src="https://github.com/user-attachments/assets/91c01949-9557-4c03-bee0-4790bf227478" />
<img width="797" height="314" alt="Zrzut ekranu 2025-06-05 190332" src="https://github.com/user-attachments/assets/3cd90537-9961-419b-a60a-76b481dff477" />

If the user was disabled another message is posted to Slack with following text:  
<img width="322" height="111" alt="Zrzut ekranu 2025-06-05 190349" src="https://github.com/user-attachments/assets/9789340a-5bf3-4099-b506-99d6d07bf319" />

And this is how it looks in practise:  

<img width="310" height="50" alt="Zrzut ekranu 2025-06-05 191142" src="https://github.com/user-attachments/assets/49f61264-c70d-4268-8b1c-da005645debe" />

## Adding Sysmon and integrating it into Shuffle playbook

To go beyond the tutorial from MyDFIR's YouTube series, I implemented Sysmon-based detection to monitor suspicious network behavior, particularly potential C2 (Command and Control) communication.

So first I installed Sysmon on both Windows Machines.
Next I updated configuration file to one found here [SwiftOnSecurity sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)

To make sure Sysmon logs were transferred to Splunk, I added this to the inputs.conf file:
<img width="534" height="86" alt="obraz" src="https://github.com/user-attachments/assets/795a3344-9c33-4a5b-854a-d857491ad8b4" />


After that, I restarted the services and checked Splunk:
<img width="802" height="398" alt="Zrzut ekranu 2025-06-08 211624" src="https://github.com/user-attachments/assets/393d4b93-8486-4614-8828-d97fecc11f53" />

However, there was an issue: there was too much noise. Sysmon generated a significant amount of useless logs related to SplunkForwarder activity:

<img width="839" height="795" alt="Zrzut ekranu 2025-06-08 205949" src="https://github.com/user-attachments/assets/ed09d669-4f16-44e2-8efe-9de5bf003400" />

To stop that, I added the following lines to the Sysmon configuration file (in the <ProcessCreate onmatch="exclude"> section):

<img width="793" height="73" alt="Zrzut ekranu 2025-06-08 210419" src="https://github.com/user-attachments/assets/8bc7c126-04bb-4079-8b25-44fe6b6dd20d" />
<img width="1032" height="52" alt="Zrzut ekranu 2025-06-08 210502" src="https://github.com/user-attachments/assets/86507d07-eff5-476f-af5f-89e1576778b5" />

After that was taken care of, I checked how normal traffic looks and specifically what destination ports are used by traffic coming from my Windows machines:

<img width="1904" height="596" alt="Zrzut ekranu 2025-06-09 213803" src="https://github.com/user-attachments/assets/bb0d6675-9198-4730-aec1-656e44c7e901" />

A few of these ports were probably one-time occurrences related to other activities, but now I had a better idea of what destination ports "normal" traffic uses.
So, to detect potential C2 communication, I created this query:

<img width="1660" height="136" alt="Zrzut ekranu 2025-06-09 222541" src="https://github.com/user-attachments/assets/cf63b7fc-06b8-44fa-8325-a4d4af7a8a6c" />

Next, I set up a simple three-part workflow in Shuffle to alert a SOC analyst about this potential issue:
<img width="762" height="237" alt="obraz" src="https://github.com/user-attachments/assets/f7f60466-0f91-44df-b7c2-1e365a971fb1" />

As before, I created an alert from this query called "HKdomain-potential-C2-communication":
<img width="866" height="604" alt="Zrzut ekranu 2025-06-09 222450" src="https://github.com/user-attachments/assets/18b6e568-0ce1-4093-899b-685e5ab7177a" />

After all of that was set up, I simulated this malicious activity by activating a simple HTTP server on a Linux machine using python -m http.server on port 8080.
Then I used Invoke-AtomicRedTeam to start the communication with T1041-1 tets, although, to be fair, I could have just used a browser. But I wanted to experiment with a different tool.
Here’s the result:

<img width="618" height="247" alt="Zrzut ekranu 2025-06-09 223523" src="https://github.com/user-attachments/assets/d104d338-75d8-410d-a6e5-e221abefbc90" />

The alert worked.

## Summary
This project involved building an automated incident detection and response system across Windows (Active Directory) and Linux environments using Splunk as the SIEM, Shuffle for automation, and a variety of log sources (Sysmon, Syslog) hosted on AWS EC2 instances; the system detects attacks (e.g., port scans, unauthorized access), alerts SOC analysts via Slack and Gmail, and offers automated response options like disabling user accounts, resulting in an integrated, cloud-based SOC solution tested with Atomic Red Team simulations.

## Sources
[Splunk Documentation](https://docs.splunk.com/Documentation/Splunk)   
[Splunk Universal Forwarder Documentation ](https://docs.splunk.com/Documentation/Forwarder)  
[Shuffle Documentation](https://shuffler.io/docs/about)  
[MyDfir](https://www.youtube.com/@MyDFIR)  
[Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)  
[SwiftOnSecurity sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)  
[Setting up the splunk universal forwarder on kali linux](https://iritt.medium.com/setting-up-the-splunk-universal-forwarder-on-kali-linux-for-your-cybersecurity-home-lab-c153d19215dc)  
[Invoke-AtomicRed](https://github.com/redcanaryco/invoke-atomicredteam/wiki/Installing-Invoke-AtomicRedTeam)
