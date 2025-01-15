# Active Directory Home Lab : Attack and Monitor

## Project:

This lab is dedicated to maintain Active Directory, simulating cyber
attacks and monitor them in a SIEM. I'll be using virtual machines for
this lab and I'll be using splunk as the SIEM. I'll use sysmon and
atomic red team as well. Below is diagram:

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image11.png" width="557" height="424" />
## VM Installation:

The first step is to install total of four VM.

1.  Installing a Windows 10 as Client

2.  Installing a Windows server as Active directory domain controller

3.  Installing a ubuntu live server(22.04.x version is preferred) as Splunk server

4.  Installing a Kali linux as Attacker
#

After installing all the machines need to update and upgrade the ubuntu
and kali machines:

Command : sudo apt-get update && sudo apt-get upgrade.

Now I'm creating a Nat Network profile for this lab and making sure all
the machines are using this network.
#

## Setup:

Now lets setup splunk on the ubuntu server. Download the splunk
enterprise free from their website for ubuntu (.deb) . Then install
splunk using dpkg.\
Now change the user to splunk and go to "/opt/splunk/bin" directory and
start splunk. Set username and password for login. Then add splunk in
boot-start.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image2.png" width="733" height="58.176" />

Now downloaded Universal splunk forwarder and sysmon on both client and
active directory machine.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image3.png" width="305" height="218.78" />
<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image10.png" width="354.24" height="211" />

Now create a file named 'inputs.conf' in "C:\\Program
Files\\SplunkUniversalForwarder\\etc\\system\\local\\". Now edit that
file:

> \[WinEventLog://Application\]
>
> index = endpoint
>
> disabled = false
>
> \[WinEventLog://Security\]
>
> index = endpoint
>
> disabled = false
>
> \[WinEventLog://System\]
>
> index = endpoint
>
> disabled = false
>
> \[WinEventLog://Microsoft-Windows-Sysmon/Operational\]
>
> index = endpoint
>
> disabled = false
>
> renderXml = true
>
> source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

Splunk will collect log from these sources only. Now from services
restart the SplunkForwarder service to apply this settings. Did this for
both client and active directory machines. Now create a index named
"endpoint" on the splunk server as I have defined this index in the
config file.
#

Lets check on the splunk that the logs are being generated from this two
machines.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image19.png" width="752" height="279" />

Lets configure the Active Directory machine now. Firstly open the server
manager.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image12.png" width="430" height="279" />

Then from the manage option selecting "Add roles and Features" and start
setting up.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image5.png" width="440" height="286" />

Then from the flag option promote this server to domain controller

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image9.png" width="440" height="284" />

Select "add a new forest" from the next window and complete the setup.
The machine will restart and active directory domain controller
installation is done. Now let\'s add some users.

Open the tools section, then open active directory users and computers.
I have created two Organizational Units named HR and IT . Then added one
user in each Unit.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image17.png" width="532" height="305" />

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image7.png" width="331" height="236" /> <img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image14.png" width="342" height="244" />
#

Now for the client machine changed the dns server IP to the Domain
controller machines IP. Added this client machine into the domain from
advance system settings.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image6.png" width="593" height="330" />

After rebooting I logged in using any user credential from that two I
created earlier in Active Directory.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image22.png" width="603" height="312" />

Now for the attacker machine(Kali) logged in using default credential.
Lets install crowbar.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image16.png" width="464" height="248" />

Then installed Atomic Red Team using the following command:

IEX (IWR
\'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1\'
-UseBasicParsing);
Install-AtomicRedTeam -getAtomics

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image1.png" width="493" height="166" />

## Attack and Log Investigation:

On the kali machine edited the rockyou.txt file and added my two users
password that I created earlier.

On the client machine enabled the RDP and added the two users there. Its
time to generate the brute force attack on the client pc using crowbar.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image15.png" width="583" height="253" />

Found the brute force attack. Here I had total 50 password in my
rockyou.txt including one correct password on the last line. So 49
failed login attempt happened.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image20.png" width="608" height="418" />

Found the successful logged one.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image18.png" width="612" height="211" />

Here the Source Address and the Workstation Name shows the attacker
machine IP address and name.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image4.png" width="620" height="212" />

## Telemetry Generation and Log Investigation:

Now generating some telemetry using Atomic Red Team,

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image8.png" width="716" height="384" />

I have generated telemetry using atomic red team that creates a user.
The username is NewLocalUser. Below is the log that was generated.

<img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image21.png" width="660" height="269" /> <img src="https://github.com/muja789/Active-Directory-Project/blob/main/Active%20Directory%20Project/media/image13.png" width="660" height="193" />
#
