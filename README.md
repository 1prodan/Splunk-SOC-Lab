<h1>SOC Demonstration With Splunk</h1>

<h2>Using Splunk along with a vulnerable Windows machine and Kali Linux to demonstrate basic SOC capabilities.</h2>

For this lab, I setup 3 different virtual machines that will be the core of the project. One Ubuntu vm that will run Splunk, a Windows 10 vm to be the target / simulated corporate user device that will run the Splunk Forwarder and a Kali Linux vm that will be used to attack the target machine.
<br><br>
I began by installing all the correct tools and software required for this lab. I installed Splunk Enterprise on my Ubuntu machine and setup the Forwarder for my Windows 10 Machine. I made sure to create a NAT Network so that all my machines could communicate with each other and tested this by pinging each machine with the other. 
<br><br>
I then created my inputs config file for the Windows machine so that Splunk Forwarder could provide the correct logs to the Splunk instance.
<br><br>
<img src="https://i.imgur.com/FlgyN9w.png" height="60%" width="60%"/>
<br><br>
After the setup and validation that each machine could communicate, I enabled RDP connections on my Windows machine to act as a vulnerable target for my attack.
<br><br>
I now had Splunk setup and ready to go, already having some basic System/Security events from Event Viewer showing.
<br><br>
<img src="https://i.imgur.com/28TrSb8.png" height="80%" width="80%"/>
<br><br>
Now, it was time to conduct the attack with Kali. I started with a basic Nmap scan on the Windows device to reveal some basic information as well as any open ports like a real-life threat actor might do.
<br><br>
<img src="https://i.imgur.com/D3ZARnL.png" height="60%" width="60%"/>
<br><br>
As we can see, the Nmap scan revealed the open 3389 RDP port as well as some other useful information like OS, Target Name, and Mac Address. 
<br><br>
Next, it was time for the actual attack to take place. For this instance, I utilized Hydra, a cracking tool used for brute forcing attacks. The attack itself was unsuccessful, but the logs of everything that occurred had been clearly reflected inside of Splunk.
<br><br>
<img src="https://i.imgur.com/m27FJ5E.png" height="60%" width="60%"/>
<br><br>
<img src="https://i.imgur.com/uMy5cF2.png" height="60%" width="60%"/>
<img src="https://i.imgur.com/7MFe0Uj.png" height="60%" width="60%"/>
<br><br>
This is an example entry inside of Splunk for one of the failed login attempts by the Hydra attack. As we can see, it shows the 4625 Event ID for a failed login attempt. It also shows some other good information such as the login type of 3, which indicates a network login and the source ip of the attacker and workstation name. 
<br>
This is all valuable information we can use to triage this if it were an actual incident or alert in a real world attack. Information such as the ip address or workstation attempting to login, and the machine attempted to be logged into can be used as key information for an investigation. If the machine is typically something that should only be accessed by certain workstations or ips, it could be a huge red flag. The amount of login attempts and the ip address from which it is attempted to be accessed could also be huge red flags for an attack.
<br><br>
An Event ID of 4740 which indicates the account was locked is also used in conjunction with 4625 IDs to give more context to the incident. A lockout of the account could be further proof that a brute force or credential stuffing attack is happening.
<br><br>
<img src="https://i.imgur.com/thFN2jw.png" height="60%" width="60%"/>
<br><br>
I used Claude to assist me with generating an SPL query to be used to create a dashboard on Splunk. 
<br><br>
<img src="https://i.imgur.com/WKx3dVD.png" height="60%" width="60%"/>
<br><br>
This is the query I ended up using after and the dashboard I was able to create with it.
<br><br>
<img src="https://i.imgur.com/UXqE3Vq.png" height="60%" width="60%"/>
<img src="https://i.imgur.com/51NiE4H.png" height="60%" width="60%"/>
<br><br>
The query shows all failed login attempts in a 30 minute time frame window and sorts it by source ip, number of failed attempts and the user that was attempted to be logged into. This allows for a quick glance at the information to make a faster decision if necessary for an incident. 
<br><br>
Claude was also able to point out that the status and sub_status fields in an event log can also provide some valuable information. A 0xC000006A status leads to a bad password, potentially indicating credential stuffing, while a 0xC0000064 implies that the user does not exist, which could point to guessing or brute forcing. From our earlier logs, we can see the former status, which aligned with the correct username but invalid password credentials from the attack.
<br><br>
All of this information can be used to make a decision on each log and alert that comes in. Once an understanding of the baseline environment is established, anomalies can become more easily detected and separated from false positive alerts. 
