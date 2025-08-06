# Introduction

This project involved building an automated system for real-time incident alerting and response across both Windows (specifically Active Directory Domain Controllers and hosts) and Linux environments. It consists of several EC2 instances hosting a Splunk server, a Domain Controller, and two monitored hosts. The system makes use of free and open-source tools, with Splunk serving as the SIEM platform for incident analysis and alert generation and Sysmon and Syslog which are used for detailed log collection. Additional tools include Shuffle, which enables automated responses to alerts, such as sending dedicated messages or emails to a SOC analyst via Slack or Gmail, with option to react to incident via disabling the user. I tested this system against two types of attacks: unauthorized access from an unknown IP address to one of the endpoints, and a potential port scanning, simulated using the Invoke-AtomicRedTeam framework. The end result is highly effective system for monitoring hosts, alerting and reacting to potentional incidents and overall day-to-day work in SOC environment.

# Tools used

AWS (for hosting all EC2 istances)
Splunk (as a SIEM platform and alert generation)
Sysmon (log generation)
Syslog (log generation)
Shuffle (for play book creation and automatic response to alerts)
Slack (as a message app and dedicated place for Shuffle to send alerts to SOC analyst)
Invoke-AtomicRedTeam (for testing the environment)

# Acquired skills

AWS usage for vm hosting

Creation and managing simple Active Directory Domain

Setting up  and configuring Splunk for alert generation

Creation of custom alert rules for Splunk

Setting up hosts in both Windows and Linux enviornment to send logs to Splunk

Automating part of incident response with Shuffle (Sending messages to Shuffle and sending emails to Gmail with option of react to incident)

Integration of multiple security tools for better monitoring and protection

Installing and configuring Sysmon for specific environmental need

# Project

## System Diagram
This is how the system work looks and works like in practise. All telemetries are send to Splunk. After that if alert is triggered, it is send to Shuffle and to both Slack and SOC analyst email. Next, SOC analyst has a choise to either disable the user, which would also generate another message in Slack or do nothing.
![Image](https://github.com/user-attachments/assets/f58d0f93-7f00-4d0e-95cc-2ca93c467f37)

## Installing and Configuring Active Directory

After creating all of the machines seen in system diagram, I started working on Active Directory.
First I assigned one of the Windows EC2 instances (the one with more recources), as a Domain Controller.
To do that I used server manager and clicked "Add roles and features":
<img width="945" height="495" alt="Image" src="https://github.com/user-attachments/assets/d6580d96-2baa-48ea-b171-9a1d9ffc5062" />

Next selected installation type:
<img width="945" height="668" alt="Image" src="https://github.com/user-attachments/assets/5234c761-459d-4429-83c5-aadc65236b36" />

And after that, I choose "Active Directory Domain Services" in "Select server roles" panel:
<img width="945" height="667" alt="obraz" src="https://github.com/user-attachments/assets/0b708ff4-f696-4eaf-b27e-f284147bb534" />

After all of that was done installing, I promoted the machine to Domain Host by clicking the alert in right corner and after that:
Creating new forest named "HKdomain.local":

<img width="945" height="693" alt="obraz" src="https://github.com/user-attachments/assets/e36d8836-82c0-481d-94e2-feb65a744508" />

Setting up password (becuase of the nature of this project password choosen wasn't that long, should be longer in professional setting):
<img width="945" height="689" alt="obraz" src="https://github.com/user-attachments/assets/218951a4-0997-4f0c-a332-24c8c12054a6" />

Next, thing to do was to pass "Prerequisites check" and install all necessities:
<img width="945" height="700" alt="obraz" src="https://github.com/user-attachments/assets/2aa2ee03-4443-4571-9990-d9ee00a4675c" />

And after all of that, and after machine restart, the Domain Controller settup was complete.

Next step, was to create a new user in the local domain and configure second Windwows machine to become part of:
<img width="945" height="663" alt="obraz" src="https://github.com/user-attachments/assets/89eb2306-2da9-4438-a7d8-900c50d5fce1" />

And similarly to password in Domain Controller for the sake of "ease of use", I choose simple password and "Password never expires" option, in real word scenarios this would be highly discouraged, because of security implications

<img width="845" height="738" alt="obraz" src="https://github.com/user-attachments/assets/6fb9c3a8-3f03-4472-aa46-fbb777ad0a52" />

After that I checked if user was created, and he was:
<img width="703" height="77" alt="obraz" src="https://github.com/user-attachments/assets/5193d1fd-3067-486e-9787-1feb24555a4e" />

So, only thing left to do was to configure second machine:
First, I changed the DNS server to Domain Controller:
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

Next step after configuring Active Directory, was to setup Splunk SIEM and to make all the hosts forward their logs into it.
First, I downloaded installation package into Linux server and after that I followed official installation tutorial found in Splunk documentation.

After that all that was left to do was executin splunk_start binary and go to the activated instance:
<img width="1919" height="907" alt="Zrzut ekranu 2025-06-02 140217" src="https://github.com/user-attachments/assets/521dd201-4faa-42a7-a793-c5419d0f98c9" />

Next, I downloaded Splunk Add-on for Windows (for better log parsing):
<img width="1895" height="846" alt="Zrzut ekranu 2025-06-02 140440" src="https://github.com/user-attachments/assets/46f07b6f-ec24-4657-acb7-17af7ce8fec8" />

Last, step was enabling listening port for log transfer purposes and created hk-domain index:
<img width="1919" height="908" alt="Zrzut ekranu 2025-06-02 140622" src="https://github.com/user-attachments/assets/ff6d30ab-91b1-4e88-8637-5e1a12f45fea" />
<img width="1894" height="154" alt="Zrzut ekranu 2025-06-02 151443" src="https://github.com/user-attachments/assets/23d184ac-38ad-40c8-8eeb-b2253b650b99" />


Now that that's out of the way, I configured both Windows and Linux machines, to forward their logs to Splunk.
To do that on both machines I:
Downloaded Universal Forwarder:
<img width="945" height="57" alt="obraz" src="https://github.com/user-attachments/assets/d0a61df7-5d66-46db-81ef-9401a6c5ed2c" />

Setup credentials and receiving index:
<img width="945" height="715" alt="obraz" src="https://github.com/user-attachments/assets/bc445d29-c29a-4ac5-80c7-2ec631fb7fb1" />
<img width="945" height="745" alt="obraz" src="https://github.com/user-attachments/assets/383656af-75ac-4b5a-8a3b-6ea65df30325" />

Next, I changed the C:\Program Files\SplunkUniversalForwarder\etc\system\local\input.conf to forward Security logs into hkdomain-ad index and SplunkForward properties to Local System account:
<img width="433" height="127" alt="obraz" src="https://github.com/user-attachments/assets/dc7a71cb-d4ce-4baf-b763-8277c34eae47" />
<img width="789" height="908" alt="obraz" src="https://github.com/user-attachments/assets/97a3eb8c-e524-4a4d-aa56-225d5943e872" />

And to check if everything works, I looked at hkdomain-ad index in Splunk:
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
I mostly used this blog for help with configuration https://iritt.medium.com/setting-up-the-splunk-universal-forwarder-on-kali-linux-for-your-cybersecurity-home-lab-c153d19215dc

In the same way as hkdomain-ad index, I created hk-linux index used for all logs from my Linux server
Next I installed the SplunkForwarder, started it and added my Splunk server using splunk add forward-server command
After that, I added inputs.conf into /opt/splunkforwarder/etc/system/local/ directory with following content:
<img width="378" height="114" alt="Zrzut ekranu 2025-06-09 150519" src="https://github.com/user-attachments/assets/c1eda50b-2cf0-4f5c-bd36-33a3357bdeae" />

And this is content of /opt/splunkforwarder/etc/system/local/outputs.conf after adding the forward-server:
<img width="489" height="180" alt="Zrzut ekranu 2025-06-09 150538" src="https://github.com/user-attachments/assets/670a8e47-57a9-4595-91cf-0c608d01bd7c" />

Last thing do to was checking if the setup works, so I used query for index hk-linux:
<img width="1888" height="863" alt="Zrzut ekranu 2025-06-09 151011" src="https://github.com/user-attachments/assets/ac4d0ff7-f968-485f-b7a4-e09661a0541c" />

And it worked so, initial Splunk setup is done.

## Slack and Shuffle automation

First step of this part creating free account for both Slack and Shuffle. After doing that I created this playbook in Shuffle:
<img width="1469" height="685" alt="Zrzut ekranu 2025-06-05 190036" src="https://github.com/user-attachments/assets/7ee41f0e-742d-4e57-b7e9-03fd8e23f9ef" />

First is webhook called "Splunk Alert" and as the name suguests, it's role is to catch alert generated by Splunk:
<img width="525" height="1158" alt="obraz" src="https://github.com/user-attachments/assets/8fe2c06d-293f-4ae2-b908-d0056fdab16d" />

Url found in this was used during configuration of Splunk alert described in previous section.

Next is "Alert Notification" used to post the alert notification into Slack, the channel ID was found in Slack url, channel currently inactive:
<img width="326" height="741" alt="obraz" src="https://github.com/user-attachments/assets/3d547399-265d-4e1f-b757-3d6efda9ad83" />

And this is the message schema, the $exec.result.* values where extracted by first starting the webhook and activating an alert, after that the parameters where aviable for me to use:
<img width="320" height="200" alt="Zrzut ekranu 2025-06-05 190123" src="https://github.com/user-attachments/assets/04e97ba5-8809-4627-a41b-c72c4ff1e1a4" />

And this is the end result, after the alert has been prokt:
<img width="848" height="231" alt="obraz" src="https://github.com/user-attachments/assets/9b6594a2-297e-422a-a936-089ff3733c40" />

Next is "Email Notification", that similarly to prevous element, is used to send message after alert, but this time through email and with additional option of action for SOC analyst:
<img width="335" height="559" alt="obraz" src="https://github.com/user-attachments/assets/d0ccb9c6-fdf7-4497-a2c2-7a16da6281f3" />






