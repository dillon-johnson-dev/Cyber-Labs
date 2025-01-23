# Packet Detective - Cyberdefenders.org

Using **cyberdefenders**, I investigated a network scenario using **Wireshark** to analyse PCAP files. This exercise focuses on examining attacker patterns and uncovering Indicators of Compromise (IOC).

---

## **Scenario:**
In **September 2020**, your SOC detected suspicious activity from a user device, flagged by unusual **SMB protocol usage**. Initial analysis indicates a possible compromise of a privileged account and remote access tool usage by an attacker.

Your task is to examine network traffic in the provided PCAP files to: 
- Identify key indicators of compromise (IOCs) and gain insights into the attacker’s methods.
- Investigate evidence of persistence tactics, and goals.
- Construct a timeline to better understand the progression of the attack by addressing the following questions.

---

## Tools Used:
**Wireshark**

## Investigation:
### 1. Analyzing SMB Protocol Usage to Uncover Attacker Activity, Compromised Credentials, File Access, and Evasion Tactics.
After loading the given PCAP file into Wireshark, I navigated to **Statistics > Protocol Heirachy** to analyse the distribution of traffic. This revealed that SMB-related traffic accounted for a significant portion of the overall network activity. With this, I could verify potential patterns of data transfer or file access.

![1](https://github.com/user-attachments/assets/19bf00ac-f94a-4c1f-a8c5-c3546951e1c1)

Now knowing this, I investigated the compromised credentials the attacker used to authenticate. Since SMB authentication utilises NTLM, I applied the **ntlmssp** filter as this is used to assses different types of authenticated uses. So using **ntlmssp.auth.username** filter, I found that the "Administrator" account was used to perform these attacks. Confirming that a privileged account was compromised.

![2](https://github.com/user-attachments/assets/c894812a-c9b5-4363-86d3-bd52c1233bb9)

Next, I investigated how the attacker utilised this administrative access. Given the high volume of SMB traffic, I checked if there were any files created or exported through the SMB protocol. This was done through **File > Export Objects > SMB**, revealing the **"eventlog"** filename was accessed by this attacker. This file name also helped us see an attempt to clear logs related to this file at exactly **2020-09-23 16:50:16**.

![5](https://github.com/user-attachments/assets/4073d3ab-ca53-4968-845f-05a487be98b7)
![6](https://github.com/user-attachments/assets/8749a36e-95cc-4d2e-ae6e-148ee3233d6e)

---

### 2. Investigating Lateral Movement and Persistence Through Named Pipes and Unauthorized Network Communication.
The attacker had leveraged "named pipes" for communication, likely utilising Remote Procedure Calls (RPC) to facilitate lateral movement accross the network. RPC allows one program to remotely request services from another, allowing the attacker to have unauthorised access and control over the target system. 

Investigating further, I analysed the service that communicated using the named pipe. This was done by filtering the data with **"frame contains 5c:00:50:00:49:00:50:00:45"** (isolating packets related to \PIPE) and allowing the packet to look for PIPE as a string to then reveal the service name **"atsvc"**. Folowing this, I then measured the level of communication by checking conversation duration between the two IP's (172.16.66.1 and 172.16.66.36). This provided insights into how long the attacker maintained communication, revealing the persistance of their unauthorised access.

![5](https://github.com/user-attachments/assets/c187edba-e1bc-45e2-9bdb-aa327845139c)
![6](https://github.com/user-attachments/assets/57f150ca-fc88-4122-905e-ee0787483db3)

### 3. Identifying Covert Access and Remote Execution Methods Used by the Attacker.
To maintain covert access, the attacker had used a non-standard username to set up requests. To identify this username, I applied the **"ntlmssp.auth.username"** filter to the IP **172.16.66.36**. With this I was able to identify **"backdoor"** as the username.

![7](https://github.com/user-attachments/assets/b2937111-9d0d-4165-b682-142f21851cbe)

And because this username was using the SMB2 protocol, I was able to find executable files associated with that username and protocol by applying the filter **"smb2.filename contains '.exe'"**.

![8](https://github.com/user-attachments/assets/3a152ed3-c416-498d-bcdc-ce791710e0d4)

## Skills Obtained:
### 1. Wireshark Statistics
### 2. SMB Protocol Analysis
### 3. Timeline Correlation
### 4. IOC Identification

## Reflection:
This lab reinforced the importance of a methodical approach to analysing network traffic. Each of the questions in the lab provided a huge learning opportunity, leveraging Wireshark's built-in statistics and varied filtering approaches. These skills will be highly beneficial to my ability and experience in analysing website traffic and unauthorised access in networks.
