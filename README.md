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
Now, we'll go check the sample victim emails we launched the campaign toward and it looks like the phishing emails were received!
<br />
<img src="https://i.imgur.com/AGCzKU3.png" height="60%" width="60%" alt="email inbox"/>
<br />
<img src="https://i.imgur.com/d6szVNU.png" height="70%" width="70%" alt="Contents of email"/>
<br />
Above, we can see the email template we created earlier with Gophish.
**A quick note**
In the real world, it's vital to examine every detail in suspicious emails. Big indicators of phishing emails are the grammatic errors. Despite the shady sense of authority from the message, the misspelling and improper email format is usually a dead giveaway, telling most that this email is a phishing attempt. 
<br />
<img src="https://i.imgur.com/FJpR1cQ.png" height="75%" width="75%" alt="Phishing indicators"/>
<br />
We’ll proceed by clicking the hyperlink in the email message, and we’re immediately redirected to a google page. But the URL looks different. It has the IP address of my Kali Linux VM inside the URL tab, which would not normally look right, but this is a good indication that the victim has been successfully redirected to credential harvester URL. I go back to my Kali Linux VM, and the feed shows the activity of the Windows 10 VM connecting to the malicious URL.
<br />
<img src="https://i.imgur.com/HtoxulG.png" height="75%" width="75%" alt="Credential Harvester feed"/>
<br />
So, on the Windows 10 VM, let’s try logging in as the victim who fell for the phishing email. To check our Google account, the victim will input some credentials to log in.
<br />
<img src="https://i.imgur.com/fUQEbez.png" height="80%" width="80%" alt="Sign in to Google..."/>
<br />
Once we click <b>Sign in</b>, we'll navigate back to our Kali machine, we should see what the SET credential harvester tool has captured in real-time, and it turns out that we have captured something!
<br />
<img src="https://i.imgur.com/3K3FwM6.png" height="85%" width="85%" alt="Captured Credentials"/>
<br />
Success! The feed shows the exact same credentials that we just put in as the victim on the Windows 10 VM. We have successfully captured a victim’s credentials through first creating a phishing email and then utilizing the Social Engineering Toolkit as the payload to capture the credentials that the victim would provide. One cool feature of the Credential Harvesting tool is that once a victims’ credentials are captured, the malicious site refreshes and is immediately replaced with the actual site it cloned. Hence, the victim would be directed to the actual Google website after submitting their credentials.
<br />
<img src="https://i.imgur.com/8hG24Zb.png" height="80%" width="80%" alt="Back to regular Google"/>
<br />
