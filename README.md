# Introduction

This project involved building an automated system for real-time incident alerting and response across both Windows (specifically Active Directory Domain Controllers and hosts) and Linux environments. It consists of several EC2 instances hosting a Splunk server, a Domain Controller, and two monitored hosts. The system makes use of free and open-source tools, with Splunk serving as the SIEM platform for incident analysis and alert generation and Sysmon and Syslog which are used for detailed log collection. Additional tools include Shuffle, which enables automated responses to alerts, such as sending dedicated messages or emails to a SOC analyst via Slack or Gmail, with option to react to incident via disabling the user. I tested this system against two types of attacks: unauthorized access from an unknown IP address to one of the endpoints, and a potential command-and-control (C2) connection to an attacker’s server, simulated using the Invoke-AtomicRedTeam framework. The end result is highly effective system for monitoring hosts, alerting and reacting to potentional incidents and overall day-to-day work in SOC environment.

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









