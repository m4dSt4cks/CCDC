# Router/Firewall #
**Check off things with a [] next to them as you do them**

---

## Host Password and Remote Workstation Hardening ##

##### Change Host Machine Password [ ] #####
	net user Student [password]
	**WRITE ON STICKY NOTE AND PLACE ON HOST MACHINE**

##### Login to Workstation [ ] #####
	$ ssh [user]@[remote IP]
	- OR -
	Use web interface

##### Change Admin Password [ ] #####
	Admin Account- Username: ______________________ Password: #________________________

---

## Block Outside Network Traffic [ ]  ##
	# Look at list of known IPs we have on the network
	1. Allow traffic inbound only if it is going to one of those IPs
	2. Allow traffic outbound only if it is coming from one of those IPs
	3. Determine if user machines are required for scoring, if they aren't then leave them blocked

## Scan Network [ ] ##
	1. Login to a Windows machine on network without firewall rules that block ICMP packets
	2. Use "arp -a" to quickly see what machines have been communicating on the network,
	and see if any do match the IPs we have
	2. Run this command on a Windows machine:
		FOR /L %i in (1,1,255) do @ping -n 1 -w 100 X.X.X.%i | find "bytes=" (replace Xs with subnet)
	3. If known boxes respond, they have not set up their firewall correctly and need to be alerted
	4. If new boxes are found
		1. Login
		2. Change the password
		3. Write it on the network diagram
		4. Allow traffic from outside the network if necessary

## Monitor Traffic / Set Firewall Rules [ ] ##
	# Do this mostly on the DMZ router to look for malicious incoming traffic,
	but also occasionally on internal routers to look for pivoting
	1. Filter to find malicious traffic
		-SQL injections ('or 1=1--)
		-Similar attacks (see Nathan's packet)
		-Known malware file names
		-Generally see what is coming from outside the network
	2. Look for large nmap scans
	3. Look at traffic that is trying to reach boxes on ports that 
	they don’t have a service on (ex. a request to the web server on port 22)
	4. Look for repeated login attempts on each box
	5. Blacklist IPs as found/on Blacklist Board and put a check mark next to them once completed

## Individual Router Rules [ ] ##
	# Wait until instructed to do this
	1. Get rules from a machine
	2. Double check that they are right
	3. Implement on appropriate router
	4. Repeat for each machine
	5. Deny all other traffic

## Continuously ##
	1. Check the Blacklist Board and make sure their traffic in and out are 
	disallowed by the public facing router
	2. Add known malware signatures to firewall rules
