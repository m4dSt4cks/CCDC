# Windows #
**Check off things with a [] next to them as you do them**

---

## Host Password and Remote Workstation Hardening ##

##### Change Host Machine Password [ ] #####
	net user Student [password]
	**WRITE ON STICKY NOTE AND PLACE ON HOST MACHINE**

##### RDP into Workstation [ ] #####
	Windows Flag->Remote Desktop Connection

##### Change User Account Passwords [ ] #####
	# Use on-screen keyboard when you type in actual password
	# Use command prompt with admin privileges
	Windows Flag->cmd->right click->Run as Administrator
	# Type this as one long line into command prompt, this will not work in a batch file
	for /f "tokens=* skip=1" %a in ('wmic UserAccount where "LocalAccount=True" get Name') 
		do for %b in (%a) do (
    		if not "%b"=="" (
       		 	net user %b [password]
    		)
		)

	** IF A USER ACCOUNT IS NOT SUPPOSED TO HAVE ITS PASSWORD CHANGED **
	** THEN FOLLOW THIS PATTERN **
	for /f "tokens=* skip=1" %a in ('wmic UserAccount where "LocalAccount=True" get Name') 
		do for %b in (%a) do (
    		if not "%b"=="user1" if not "%b"=="user2" if not "%b"=="" (
       		 	net user %b [password]
    		)
		)
	User Accounts Password: #___________________________________________________

##### Change Admin Password to Something Else [ ] #####
	net user [admin_user] [password]

	Admin Account- Username: _______________ Password: #________________________

##### Document Information [ ] #####
	IP: _________________
	
	Services/Ports (ex. HTTP/80): ___________, ___________, ___________, ___________, ___________

##### Set UAC to Highest Level [ ] #####
	Windows Flag->Control Panel->User Accounts and set UAC to highest level

##### Update Machine [ ] #####
	Windows Flag->Control Panel->Windows Update

##### Attempt to Install AV [ ] #####	
	Widows Flag->Search for "Windows Defender"
	Turn it on and run a scan
	MalwareBytes and MalwareBytes Anti-Rootkit are online

---

## Firewall [ ] ##

##### Quick Note #####

This part is not a checklist, this is a list of commands that you can use to properly implement a firewall. The rules to be followed:

**RDP uses local port 3389 and any remote port**

0. Allow RDP traffic in/out from our 8 host machine IPs (or just your host if you don't have time)
1. Look at current netstat -a and see what services are running on what port
2. Block all traffic in/out
3. Allow traffic in/out for the appropriate services


##### Commands #####
	# Make sure firewall is on first
	Windows Flag->Control Panel->Windows Firewall->Advanced Settings
	# You can disable rules by highlighting them and clicking "Disable"
	# You can create rules and add specifics by creating a "Custom Rule"
	- OR -
	# You can use the command line
	# Results may vary based on specific Windows OS
	netsh
	advfirewall
	reset
	firewall
	# Unblock specific ports
	add rule dir=in action=allow protocol=TCP localport=3389 name=rdp_in
	add rule dir=out action=allow protocol=TCP localport=3389 name=rdp_out
	add rule dir=in action=allow protocol=TCP localport=80 name=http_in
	add rule dir=out action=allow protocol=TCP localport=80 name=http_out
	# Block all other traffic
	set allprofiles firewallpolicy blockinbound,blockoutbound

	# List local ports that are required to be open here

	______________ ______________ ______________ ______________ ______________

	______________ ______________ ______________ ______________ ______________

	______________ ______________ ______________ ______________ ______________

---

## Malware Search ##

##### Quick Notes #####
	1. Write down all incidents and try to figure out how they happened
	2. Write malicious IPs down on Blacklist Board
	3. Document all malware found at the end of this section

##### Connections [ ] #####
	1. Connection's source port must correspond to service on box
	2. Connections that originate on another port should have the destination IP written down 
	on the Blacklist Board
	3. Write down what services the machine is running
	
	netstat 
		-a (tcp and udp)
		-f [protocol] (only shows specified protocol)
		-b (get executable name associated with connection)
	    -o (get process id associated with connection)

##### Processes [ ] #####
	1. Look for misspelled processes
	2. Look for powershell or cmd.exe (note that you may be running this)
	3. Look for random 10 digit strings
	4. Delete the process

	tasklist
	tasklist | find [pid]
	# Kill process
	taskkill /PID [pid] /F

##### Files [ ] #####
	# List files in temp folder (can also view it with windows explorer)
	dir %temp%
	# Remove files in this folder unless they really seem important
	del /f [filename]
	# Look at some other places around filesystem, spend at most 2 minutes on this

##### Scheduled Tasks [ ] #####
	# Only Windows services should have scheduled tasks, not powershell commands or random executables
	Windows Flag->type "Task Scheduler"
	- OR -
	# Command Line View
	schtasks /query 
	# Command Line Delete
	schtasks /delete /tn [taskname] /f

##### Hosts File [ ] #####
	# This file trumps the DNS server, so it needs to be accurate
	# Check for weird redirects
	type %windir%\system32\drivers\etc\hosts

##### Registry Keys [ ] #####
	**Ignore on first run through, come back after IDS is up**
	# Windows Flag->type "regedit"
	0) HKCU\Software\Microsoft\Command Processor
	1) HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce 
	2) HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices 
	3) HKCU\Software\Microsoft\Windows\CurrentVersion\Run 
	4) HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce 
	5) HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run 
	6) HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\load 
	7) HKCU\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell 
	8) HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\BootExecute 
	9) HKLM\System\CurrentControlSet\Services (start value of 2, auto-start and 3, manual start via SCM) 
	10) HKLM\System\CurrentControlSet\Services  (start value of 0 indicates kernel drivers, which load 
	before kernel initiation) 
	11) HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce 
	12) HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices 
	13) HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce 
	14) HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnceEx 
	15) HKLM\Software\Microsoft\Windows\CurrentVersion\Run 
	16) HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run 
	17) HKLM\Software\Microsoft\Windows NT\CurrentVersion\Windows 
	18) HKLM\Software\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs
	19) HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit 
	20) HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell 
	21) HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad 
	22) HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SharedTaskScheduler (XP, NT, W2k only) 
	23) HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify 
	24) HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
	25) HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
	26) HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
	27) HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders

##### New Users #####
	# IDS should catch this
	# Delete a user
	net user [user] /delete

##### Report [ ] #####

	Malware Report:
	
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________
	___________________________________________________________________________________________

---

## MADCRAPS ##

##### Add Firewall Rule for Port [ ] #####
	Specifically allow the one LOCAL port inbound/outbound AND ONLY FROM/TO THE MADCRAPS SERVER!
	
	See firewall section

##### Run Command [ ] #####
	1. Download nc.exe from the internet
	2. Choose a local port between 55,000-65,000
	# MADCRAPS server and individual ports will be assigned at beginning of competition

	nc.exe [MADCRAPS server] [MADCRAPS port] –e cmd.exe -p [local port]
	
	Local Port: ___________________
	
	MADCRAPS Port: __________________

---

## IDS ##

##### Quick Note #####

If you update the system, you will need to update the pertinent files so that the IDS will continue to work properly

##### Create a Scheduled Task [ ] #####
	schtasks /create /tn check /sc MINUTE /tr C:\check.bat

##### Setup [ ] #####
	cd C:\
	net users > a
	netsh advfirewall show allprofiles > b
	schtasks /query > c
	dir /A /B %temp% > d
	netstat > e

##### check.bat File [ ] #####
	# Create the file
	cd C:\
	echo > check.bat
	edit check.bat
	# Write this to the file
	@echo off
	cd C:\
	net users > a1
	netsh advfirewall show allprofiles > b1
	schtasks /query > c1
	dir /A /B %temp% > d1
	netstat > e1
	FC a a1
	FC b b1
	FC c c1
	FC d d1
	FC e e1
	pause

##### Verify [ ] #####
	Check that cmd prompt appears on-screen every minute
	Validate output- everything should be fine since we just hardened the box
	Scheduled Tasks will probably update to new times as they run and cause the file to change, 
	just make sure nothing sketchy appears

---

## Miscellaneous ##
	# Windows Flag + R then
	1. msconfig [ ]
	-Remove sketchy startup entries
	2. compmgmt.msc [ ]
	-Remove users from admin group (Local Users and Groups->Users)
	-Remove unauthorized shares (Shared Folders->Shares)
	3. secpol.msc [ ]
	-Set lockout threshold to 3 attempts (Account Lockout Policy) 
	4. optionalfeatures [ ]
	-Turn off optional features if you can
	5. appwiz.cpl [ ]
	-Remove unnecessary software
	6. services.msc [ ]
	-Remove unnecessary services

---

## Ongoing Tasks ##

##### Monitor IDS #####
	1. Make sure that scheduled task does not stop, if nothing new is popping up on screen 
	then it is not working
	2. If malware is discovered, change all passwords again
	3. Write down all incidents and try to figure out how they happened
	4. Write malicious IPs down on Blacklist Board

##### Services Hardening #####
	See services packet

---
---

## Helpful Stuff ##

##### Check for RDP logins #####
	Windows Flag->Search for "Computer Management"->Event Viewer->Applications and Services Logs->
	Microsoft->Windows->TerminalServices->LocalSessionManager
	Windows Flag->Search for "Computer Management"->Event Viewer->Applications and Services Logs->
	Microsoft->Windows->TerminalServices->RemoteConnectionManager



