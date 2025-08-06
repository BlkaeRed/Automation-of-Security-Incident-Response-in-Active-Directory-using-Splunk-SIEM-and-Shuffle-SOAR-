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


