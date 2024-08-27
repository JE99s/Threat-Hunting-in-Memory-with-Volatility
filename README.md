# Hunting in Memory with Volatility

<br />
<h2>Utilities Used</h2>

- <b>Kali Linux</b>
- <b>Volatility Memory Analysis Tool</b>

<h2>Environments Used</h2>

- <b>VirtualBox Hypervisor </b>

<h2>Lab Walk-through:</h2>
<p>This demonstration includes the Volatility command-line tool that will allow me to pull usefull information out of the memory image file of an infected machine that I have for my analysis. I will be looking for simple information such as the operating system of the machine where this memory file came from, how much RAM is included in the analysis, what are the running processes, any user accounts, passwords, and files associated with abnormal activity.</p>
<br />
<h3>Volatility</h3>
Volatility is a very useful memory analysis tool for many who practice IR, forensics, and cybersecurity. The 
command-line tool written in Python is a huge cohesive framework made up of powerful plugins that 
enable users to find so much information from memory images such as information about the system, 
network connections, processes running on the images, and even the ability to extract the raw memory 
dumps for further analysis or revere engineering certain malware found.


<br />
<h3>Gaining Some Information</h3>
I just logged in to my Kali Linux virtual machine with the Volatility memory forensics tool already installed. I open a terminal to locate the Volatility Python script by executing <b><i>locate volatility.py</i></b>
<br/>
<img src="https://i.imgur.com/ZynlVaO.png" height="70%" width="60%" alt="Volatility Framework Program Location"/>
<br />
I use the <b>imageinfo</b> plugin to gather information about the memory image
<br/>
<img src="https://i.imgur.com/pLsVc27.png" height="95%" width="85%" alt="Memory img info"/>
<br />
Here we have some information about the image used, including the suggested operating system and 
image type, the number of processors used, and the date and time of the memory image. It’s useful 
information. From “AS Layer1:”, IA32 Page memory indicates that the machine it was a part of was a 32-bit 
system which means that no more than 4GB of RAM is included in this analysis. Now I know that I’m 
working with an image from a 32-bit architecture Windows XP with a service pack 0. I notice the suggested 
profiles WinXPSP2x86 (Windows XP Service Pack 2x86), and WinXPSP3x86 (Windows XP Service Pack 3x86) 
but the image type section tells me that the service pack is 0 so I’m not sure which of the profiles will be 
the appropriate one to choose. I will be sticking with the WinXPSP2x86 profile for the analysis. The version 
operating system of the memory image is Windows NT displayed by the plugin command <b><i>verinfo</i></b>.
<br/>
I've had no luck finding user account names and passwords besides the user Daniel Faraday. Every time I attempt the <b><i>hivelist</i></b> plugin command for Volatility there are no memory addresses or corresponding names to be shown.
<br />
<img src="https://i.imgur.com/w6WWqDQ.png" height="90%" width="90%" alt="No profile or accounts shown with hivelist"/>
<br />

<h3>Process Analysis</h3>
I'm going to use the Volatility plugin command <b>pslist</b> to show a list of all running processes on the memory file.  It gives important information like the Process ID (PID), and the Parent PID (PPID), and even shows 
the time when a process started. I execute the <b>pslist</b> plugin to get the list of processes. Looking at them listed (e.g., <i>services.exe</i> and <i>explorer.exe</i>) they are booted first and then followed by <i>inetinfo.exe</i>, <i>hxdef100.exe</i>, and <i>poisonivy.exe</i>.
<br/>
<img src="https://i.imgur.com/B0HG3Qd.png" height="85%" width="85%" alt="PSLIST Plugin Command"/>
<br />
When looking at this it’s important to know that 
the PID identifies a process and the PPID identifies the parent of the process, the one before. I can see that 
services.exe has a PID of 732. An abnormal number of processes are starting simultaneously and have a 
PPID of <i>services.exe</i>.

<br/>
<img src="https://i.imgur.com/nHp1hNg.png" height="85%" width="85%" alt="732 PPID"/>
<br />

I can see that it's a similar situation for _explorer.exe_ which is the PPID of the following amount of processes. There are so many instances where _services.exe_ and _explorer.exe_ are PPIDs.
<br />
Now **pstree** is another plugin that can identify and list processes. The plugin outputs the same list of processes as the **pslist** command would but identifies and formats the processes in a manner which displays one child process related to a parent process in ordinal fashion. Here, we have _explorer.exe_ and the processes listed under are indented, implying that _explorer.exe_ is the parent process of those child processes. This is a first step toward getting an idea of what’s going on with this memory image.
<br/>
<img src="https://i.imgur.com/9Kyn7IW.png" height="85%" width="85%" alt="Pstree Output for Explorer.exe and Child Processes"/>
<br />

I can also use the **psscan** Volatility plugin to check for inactive and even hidden processes. I execute the **psscan** plugin and the following outputs:
<br/>
<img src="https://i.imgur.com/uJttcbv.png" height="95%" width="95%" alt="psscan attempt"/>
<br />

Compared to the results from **pslist**, one can assume some abnormalities, but I don't want to conclude anything yet with the little information that I have. Another way to find and list hidden processes is by utilizing the **psxview** Volatility plugin command. Here, I see that **psxview** essentially performs both the **pslist** and **psscan** and even determines whether they are hidden from those respective scans. This explains why there was no output from my **psscan** earlier. All processes are shown as True for **pslist** but False for **psscan** which makes me suspicious.
<br/>
<img src="https://i.imgur.com/jnr8cZg.png" height="95%" width="95%" alt="PSXVIEW PLUGIN CMD ON VOLATILITY"/>
<br />

<h3>Possible Network Connections</h3>
The Volatility tool has the ability to analyze active, terminated, and hidden connections with their corresponding processes from three important plugin commands: <b>connections</b>, <b>connscan</b>, and <b>sockets</b>.
<br />
The <b>connections</b> plugin command lists active connections at the time of the memory image. It also displays local and remote IP addresses with their respective ports and PIDs. The <b>connections</b> command can be used for Windows XP. I execute the <b>connections</b> plugin command and the following displays:
<br/>
<img src="https://i.imgur.com/zuhTCkg.png" height="95%" width="95%" alt="CONNECTIONS PLUGIN CMD ATTEMPT"/>
<br />
I see that there are no active listed connections when using the <b>connections</b> command. I will try the <b>sockets</b> plugin command next. I execute the sockets plugin command and have no luck either in finding connections.
<br/>
<img src="https://i.imgur.com/9dX1Hhl.png" height="95%" width="95%" alt="SOCKETS PLUGIN CMD ATTEMPT"/>
<br />

Now, to see a list of connections that have been ended or terminated, I can use the **connscan** plugin command. This command is also used for Windows XP systems. I execute the **connscan** command. Here, I see the ports of the systems loopback address are cross-connected using ports 1031 and 6667. The PID of 1728 belongs to _iroffer.exe_, and the PID of 1480 belongs to _bircd.exe_. However, what catches my attention is the last recorded connection. 
<br/>
<img src="https://i.imgur.com/XkH2TQs.png" height="85%" width="85%" alt="CONNSCAN ON MEMORY IMG"/>
<br />
The remote address is a clearly different subnet, with a TCP port of 3460. The 480 PID corresponds to <i>poisonivy.exe</i>. This is an abnormal finding. I want to investigate this process, but I will continue to gain more information first.
<br />

<h3>Analyzing DLLs</h3>
Dynamic-Link Libraries (.dll or DLLs) are modules that contain functions (code) and data that can be used 
by either another DLL or a program and have the ability to be used simultaneously by different data 
structures. Inspecting the DLLs of processes can greatly assist in connecting processes and their correlation 
with other programs. I will be using the <b>dlllist</b> plugin command for Volatility to print a list of all running 
DLLs for each process. I will use this plugin command specifically on poisonivy.exe. To print DLLs that are 
respective to a specific process, I use the -p flag and provide the PID of that process after the <b>dlllist</b> command.
Here, <i>poisonivy.exe</i> has quite a few DLLs, one of them being <b>kernel32.dll</b>. This kind of process being in system files is alarming to me.
<br/>
<img src="https://i.imgur.com/jVbr5Fp.png" height="85%" width="85%" alt="DLLLIST FOR POISONIVY.EXE PROCESS"/>
<br />


<h3>Malware Analysis</h3>
<h4>Poisonivy.exe</h4>
To confirm that this process is a malicious program I will be getting the hash of the <i>poisonivy.exe</i> file. Before that, I will need to dump the memory addresses of the executable to the disk of my Kali Linux box. The Volatility Framework tool has a plugin called <b>procdump</b> that allows me to dump the memory of the process to essentially recreate it on my disk for further analysis. I execute the <b>procdump</b> plugin command and it produces a <i>.exe</i> file named <i>executable.480.exe</i> in the current directory I'm in. Next, I get the MD5 hash by executing <b><i>md5sum executable.480.exe</i></b> to get the MD5 hash from the file. I have VirusTotal opened, I enter the hash and commence the scan.
<br />
Here, I see that 66 out of 72 security vendors have flagged this file as malicious from the scan. This most likely some type of malware.
<br/>
<img src="https://i.imgur.com/wDzHOv2.png" height="85%" width="85%" alt="VIRUSTOTAL SCAN OF POISONIVY.EXE"/>
<br />

I look to the <a href="https://attack.mitre.org/" target="_blank">MITRE ATT&CK</a> framework and search for the threat. This is indeed called <i>Poisonivy</i>, a remote administration tool (RAT) that acts as a backdoor to a compromised system. We're able to see this malware because it's memory resident. It's a known threat to Windows 2000, Windows XP, and Windows Server 2003 platforms. This malicious toolkit has been around for a while. There are variants of this malware that can be configured to any or all of the following:

- Capture screen, audio, and webcam 
- List active ports 
- Log keystrokes 
- Manage open windows 
- Manage passwords 
- Manage registry, processes, services, devices, and installed applications 
- Perform multiple simultaneous transfers 
- Perform remote shell 
- Relay server 
- Search files 
- Share servers 
- Update, restart, terminate itself

Most versions of Poisonivy can copy itself into other files, like system files by forking itself into alternate data streams, avoiding detection. I believe the VirusTotal scan seals the deal in identifying that there is or 
was malicious activity happening on this memory image. It says on VirusTotal that one of its imports is 
kernel32.dll to use as an exit process. We know that kernel32.dll is in fact a running DLL for <i>poisonivy.exe</i> from the DLL analysis earlier. I'm going to utilize the <b>malfind</b> Volatility command to find any hidden and injected code associated with <i>poisonivy.exe</i>. Here, there is inject code shown through the memory addresses in the output, "Hacker.Defender" and ".kernel32.dll. I think these files or codes have been injected by <i>poisonivy.exe</i>
<br />
<img src="https://i.imgur.com/KKJzePt.png" height="75%" width="75%" alt="MALFIND VOLATILITY COMMAND ON POISONIVY.EXE PROCESS"/>
<br />

It is common for malware to hide in plain sight. A virus won't be located on the <b>Desktop</b> folder, but malware commonly replaces certain system files located in the <b>system32</b> folder or outside of it. Malware likes to pose as legitimate Windows processes, which enable them to keep hidden and perform actions like keylogging, transferring more malicious files, spreading viruses, and more. I want to check some necessary processes that could possibly be holding malware, enabling the malware, or even the malware itself. The malicious poisonivy.exe has copied itself in to the machine's WINDOWS\system32 folder.

<br />
<img src="https://i.imgur.com/dsVvchI.png" height="90%" width="90%" alt="VOLATILITY FILESCAN & GREP 4 POISONIVY.EXE COMMAND"/>
<br />
I did a similar method in getting the hash of the kernel32.dll file. I used the <b>dlldump</b> plugin command for Volatility to recreate the DLL from the memory image.
<br />
<img src="https://i.imgur.com/6r03Qsv.png" height="95%" width="95%" alt="DLLDUMP CMD"/>
<br />
Once it was generated after executing the <b>dlldump</b> plugin command for Volatility, I got the MD5 hash and copied it to VirusTotal.
<br />
<img src="https://i.imgur.com/4Q4ZmJ4.png" height="85%" width="85%" alt="CMDLINE MD5 HASH OF DLL HASH"/>
<br />
<br />
<img src="https://i.imgur.com/1F3ik4D.png" height="95%" width="95%" alt="VIRUSTOTAL FEED"/>
<br />
<h3>More Findings & Conclusion</h3>
I was curious about <i>services.exe</i> and found that it had an injection as well through the <b>malfind</b> Volatility plugin command.
<br />
<img src="https://i.imgur.com/ymUuGhW.png" height="80%" width="80%" alt="VOLATILITY MALFIND CMD"/>
<br />
I performed a <b>procdump</b> on the process and grabbed its hash. I put the hash in the VirusTotal and the scan showed that there were four flags on it.
<br />
<img src="https://i.imgur.com/wOyRU1H.png" height="85%" width="85%" alt="VIRUSTOTAL SCAN OF SERVICES.EXE HASH"/>
<br />
I looked more into <i>hxdef100.exe</i> which is short for "Hacker Defender". This process is started by <i>services.exe</i> and <i>hxdef100.exe</i> is the PPID of other processes as well.
<br />
<img src="https://i.imgur.com/bE0zvB9.png" height="80%" width="80%" alt="PSTREE CMD TOWARD SERVICES.EXE AND HXDEF100.EXE"/>
<br />
I execute the <b>filescan</b> Volatility command to find any files associated with <i>hxdef100.exe</i>. The executable is in its own created directory labeled as a rootkit.
<br />
<img src="https://i.imgur.com/8ZAy85W.png" height="90%" width="90%" alt="HXDEFROOTKIT"/>
<br />
To confirm that <i>hxdef100.exe</i> is malicious or not, I do the same process of performing a <b>procdump</b> of the process to create an executable from memory, and then get the MD5 hash from the executable file so I can put that in VirusTotal.
<br />
<img src="https://i.imgur.com/sRc6qWH.png" height="90%" width="90%" alt="PROCDUMP ON HXDEF100.EXE"/>
<br />
Here, the VirusTotal scan shows me that 60 security vendors have flagged this file as malicious. This threat is labeled as a backdoor trojan.
<br />
<img src="https://i.imgur.com/zJ13Vpe.png" height="90%" width="90%" alt="VIRUSTOTAL SCAN OF HXDEF100.EXE HASH"/>
<br />
I've come to reveal some major findings throughout the lab. Here is my hypothesis as to how this machine was compromised:
<br />
The Poison Ivy malware (poisonivy.exe) was emitted into the machine via the Windows Explorer browser 
on through an FTP port, maybe port 3460 (from the <b>connscan</b> earlier) most likely through a TCP-enabled transfer file service. Somehow a malicious file containing <i>poisonivy.exe</i> was transferred into the victim's Windows XP instance and then the payload was delivered. Poisonivy could have created a run key Registry pointing to a malicious executable such as <i>hxdef100.exe</i> once it was dropped to disk. Then it executed inside the affected machine, it copies itself as critical files like <i>svchost.exe</i> to stay hidden.
<br />
This enables the download and installation of <i>hxdef100.exe</i> to create a backdoor in the background between different processes in the Windows XP machine or ports (via open ports found in an Nmap scan).
<br />
Hacker Defender or <i>hxdef100.exe</i> is a rootkit that can configure itself to connect to hidden ports on a system via netcat. It can set itself to run when the victim system boots and the file mapping name can be set when it's injected into system files. Utilizing the strings function on Kali, I found the settings section for the <i>hxdef100.exe</i> build and there are some major similarities between that and the <b>malfind</b> Volatility plugin command for PID 480.
<br />
<img src="https://i.imgur.com/dYEHjRv.png" height="90%" width="90%" alt="STRINGS CMD ON KALI FOR HXDEF100"/>
<br />
Here, I found some more configurations possibly done by the hacker showing that <i>hxdef100.exe</i> is 
set as a backdoor shell.
<br />
<img src="https://i.imgur.com/hjzE0KI.png" height="90%" width="90%" alt="STRINGS CMD ON KALI FOR HXDEF100"/>
<br />
<br />
It spread itself to replace or inject itself into essential Windows processes. Then <i>hxdef100.exe</i> can inject 
itself into batch files and other processes via Alternate Data Stream. Once a backdoor is created, the 
hacker was able to connect back to the Windows XP machine via port forwarding through a listed open 
port utilizing netcat or <i>nc.exe</i> to create a bind shell giving the hacker a remote command prompt on the 
Windows XP system.
<br />
<img src="https://i.imgur.com/A7tJZ9D.png" height="95%" width="95%" alt="NETCAT FOUND IN PSTREE COMD"/>
<br />
This provides more malicious activities such as the opportunity to create new passwords for accounts, lock 
users out of their own system, and leave critical damage to a machine.
<br />
<br />
Throughout this demonstration, I utilized a commonly used memory analysis tool to scan, connect, 
identify, and confer on the running processes from this memory image. The compromised machine 
did indeed have malicious activity going on. I found a backdoor with root-like (superuser) privileges that 
took advantage of the machine to gain access to Windows XP.

<h3>References</h3>
Balapure, Aditya. “Memory Forensics and Analysis Using Volatility.” Infosec Resources, 13 May 2021, 
resources.infosecinstitute.com/topic/memory-forensics-and-analysis-using-volatility/.
<br />
<br />
Chaturvedi, A. (2010, December 1). Playing around with HXDEF rootkit. Playing Around with HXDEF Rootkit. 
Retrieved April 2, 2023, from http://anadisays.blogspot.com/2010/11/playing-around-with-hxdef
rootkit.html
<br />
<br />

Corporation, M. (n.d.). Microsoft. threat description - Microsoft Security Intelligence. Retrieved April 2, 
2023, from https://www.microsoft.com/en-us/wdsi/threats/malware-encyclopedia
description?Name=Backdoor%3AWin32%2FPoisonivy.I&threatId=-2147363597
<br />
<br />

evild3ad. “Home.” evild3ad.Com, 20 Sept. 2011, evild3ad.com/956/volatility-memory-forensics-basic
usage-for-malware-analysis/.
<br />
<br />

eXPlorer, Hack. “How to Use Volatility - Memory Analysis for Beginners.” YouTube, 24 Jan. 2020, 
youtu.be/eluS7_eSm8M.
<br />
<br />

Hat, Black. “Investigating Malware Using Memory Forensics - A Practical Approach.” YouTube, 14 Jan. 2020, 
youtu.be/BMFCdAGxVN4. 
<br />
<br />

Linux, Kali. “Volatolity -- Digial Forensic Testing of RAM on Kali Linux.” Best Kali Linux Tutorials, 11 Dec. 
2021, www.kalilinux.in/2021/03/volatolity-digial-forensic-testing-of.html. 
<br />
<br />

Poisonivy. PoisonIvy, Software S0012 | MITRE ATT&CK®. (n.d.). Retrieved April 1, 2023, from 
https://attack.mitre.org/software/S0012/ 
<br />
<br />

Poisonivy. POISONIVY - Threat Encyclopedia. (n.d.). Retrieved April 1, 2023, from 
https://www.trendmicro.com/vinfo/us/threat-encyclopedia/malware/poisonivy
<br />
<br />

P4N4Rd1. “First Steps to Volatile Memory Analysis.” Medium, Medium, 13 Jan. 2019, 
medium.com/@zemelusa/first-steps-to-volatile-memory-analysis-dcbd4d2d56a1. 
<br />
<br />

v4L. “Hacker Defender HXDEF Rootkit Tutorial in 10 Steps [Nostalgia].” Ethical Hacking Tutorials, Tips and 
Tricks, 18 Mar. 2014, www.hacking-tutorial.com/hacking-tutorial/hacker-defender-hxdef-rootkit
tutorial-in-10-steps-nostalgia/#sthash.5Bs5vvB2.U7N3Sh54.dpbs.
