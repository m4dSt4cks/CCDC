# Unix #
**Check off things with a [] next to them as you do them**

---

## Host Password and Remote Workstation Hardening ##

##### Change Host Machine Password [ ] #####
	net user Student [password]
	**WRITE ON STICKY NOTE AND PLACE ON HOST MACHINE**

##### SSH into Workstation [ ] #####
	# PuTTY can be used to establish SSH connections from Windows machines
	- OR -
	# Linux
	ssh [user]@[remote IP]
	# If you do not login as root, you may need to use sudo privileges to do things
	$ sudo su -
	# Then enter current account password
	# If you cannot login as root, try to guess the password, if that doesn't
	work then you can disable the root account
	$ sudo passwd -l root
	# To reenable, remove 'x' from root line in /etc/passwd

##### Change User Account Passwords [ ] #####
	# Generate a list of users
	$ awk -F':' '$2 != "*" {print $1}' /etc/shadow > users.txt
	** IF A USER ACCOUNT IS NOT SUPPOSED TO HAVE ITS PASSWORD CHANGED **
	** THEN ERASE IT FROM THIS FILE **
	# Change every user’s password
	$ for user in `cat users.txt`; do echo $user':[password]' | chpasswd; done

	User Accounts Password: #___________________________________________________

##### Change Admin Password to Something Else [ ] #####
	# while on admin account
	$ passwd 

	Admin Account- Username: _______________ Password: #________________________

##### Sudoers File and Directory [ ] #####
	# Check the sudoers file
	$ cat /etc/sudoers
	# root ALL=(ALL) ALL # This should be only thing uncommented in file
	# Check the sudoers directory
	$ cd /etc/sudoers.d
	# If there is a group in either of these, it may be necessary to leave it to run services
	# Some services need to be sudo, but regular users should be removed
	# See the users in a given group and make sure no regular users are in the group
	$ cat /etc/group | grep [group]
	# Remove user from group
	$ gpasswd -d [user] [group]
	- OR -
	$ deluser [user] [group]

##### Document Information [ ] #####
	IP: _________________
	
	Services/Ports (ex. HTTP/80): ___________, ___________, ___________, ___________, ___________

##### Update Machine [ ] #####
	# Depends on version of linux
	$ apt-get update &
	- OR -
	$ yum update &

##### System Checks [ ] #####
	# Protect all the passwords
    $ chown root.shadow /etc/shadow
    $ chown root.root /etc/passwd
    $ chmod 400 /etc/shadow
    $ chmod 644 /etc/passwd
	# No users with UID of 0
	$ awk -F: '($3 == "0") {print}' /etc/passwd
	# No empty passwords
	$ awk -F: '($2 == "") {print}' /etc/shadow
	# No world writeable files
	$ find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) –print
	# No Files without owner
	$ find / -xdev \( -nouser -o -nogroup \) –print
	# Every folder should be owned by root as a rule, apply exceptions as needed
	$ chmod –R . 774 [dir] (for appropriate folders)
	$ chown root –R . [dir] (for appropriate folders)

---

## Firewall [ ] ##

##### Quick Note #####

This part is not a checklist, this is a list of commands that you can use to properly implement a firewall. The rules to be followed:

0. Allow RDP traffic in/out from our 8 host machine IPs (or just your host if you don't have time)
1. Look at current netstat and see what services are running on what port
2. Block all traffic in/out
3. Allow traffic in/out for the appropriate services

##### Commands #####
	# View firewall rules
	$ iptables -L --line-numbers
	
	# Delete a rule
	$ iptables -D [CHAIN] [NUM]
	ex. iptables -D INPUT 3
		
	# Drop by default
	$ iptables -P INPUT DROP
	$ iptables -P OUTPUT DROP
	$ iptables -P FORWARD DROP

	# Connect interfaces
	$ iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
		
	# Allow established connections
	$ iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
	$ iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
		
	# Allow a specific port in
	$ iptables -A INPUT -p tcp --dport [local port] -j ACCEPT
		
	# Allow a specific port out
	$ iptables -A OUTPUT -p tcp --sport [local port] -j ACCEPT
		
	# Allow a specific port in from specific IP
	$ iptables -A INPUT -p tcp -s [remote IP] --dport [local port] -j ACCEPT
		
	# Allow common services examples
	$ iptables -A INPUT -p tcp --dport 80 -j ACCEPT
	$ iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
	$ iptables -A INPUT -p udp --dport 53 -j ACCEPT
	$ iptables -A OUTPUT -p udp --sport 53 -j ACCEPT

	# Block an IP
	$ iptables -A INPUT -s 15.15.15.51 -j DROP
		
	# Save rules
	$ iptables-save > /rules.txt
		
	# Restore rules after reboot
	$ cat /rules.txt | iptables-restore
	- OR -
	# Restore automatically after reboot
	Put the above in /etc/rc.local

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
	1. Connections source port must correspond to service on box
	2. Connections that originate on another port should have the destination IP written 
	down on the Blacklist Board
	3. Write down what services
	
	$ netstat
		-plant (tcp connections)
		-planu (udp connections) 

##### Processes [ ] #####
	1. Look for misspelled processes
	2. Look for /bin/sh or /bin/bash
	3. Delete the process
	
	$ ps
	$ ps -ef
	$ ps aux
	$ lsof –i
	# Kill a process
	$ kill -9 [pid]

##### Files [ ] #####
	# Remove files in /tmp unless they really seem important
	$ ls /tmp
	$ rm [filename] (use * as a wildcard ex. rm * or rm *.txt)
	# Change owner to root
	$ chown root /tmp
	# Check these directories
	$ ls -a /var/spool/cron/crontabs/
	$ ls -a /etc/cron.d/
	# Check these files
	$ cat /etc/rc.local
	$ cat /etc/bash.bashrc
	$ cat /root/.bashrc
	$ cat ~/.bash_login
	$ cat ~/.bash_logout
	# Look at some other places around filesystem, spend at most 2 minutes on this

##### Crontabs [ ] #####
	# Only Unix services should have cronjobs, not random executables, shell scripts, or code that 
	opens a port
	# View
	crontab -l
	# Edit
	crontab -e
	# Directories to check
	$ ls -a /etc/crontab
	$ ls -a /etc/cron.hourly
	$ ls -a /etc/cron.daily

##### Hosts File [ ] #####
	# This file trumps the DNS server, so it needs to be accurate
	# Check for weird redirects
	$ cat /etc/hosts
	# Check IPs that are allowed to connect, shouldn't include IPs outside the network
	$ cat /etc/hosts.allow
	# Anything that this command lists allows a certain IP to login without a password
	needless to say you want to get rid of these files
	ls /home/\*/.rhosts

##### New Users #####
	# IDS should catch this
	# Delete a user
	$ userdel -r [user]

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

---

## MADCRAPS ##

##### Add Firewall Rule for Port [ ] #####
	Specifically allow the one LOCAL port inbound/outbound AND ONLY FROM/TO THE MADCRAPS SERVER!
	
	See firewall section

##### Run Command [ ] #####
	Choose a local port between 55,000-65,000
	# MADCRAPS server and individual ports will be assigned at beginning of competition

	nc [MADCRAPS server] [MADCRAPS port] –e /bin/bash -p [local port] &
	
	Local Port: ___________________
	
	MADCRAPS Port: __________________

---

## IDS ##

##### Quick Note #####

If you update the system, you will need to update the pertinent files so that the IDS will continue to work properly

##### Create a Crontab [ ] #####
	$ tty
	# Result should look somthing like "/dev/pts/0"
	$ crontab -e
	# put the next line in the crontab
	* * * * * /check.sh > [tty result] (ex. * * * * * /check.sh > /dev/pts/1)

##### Setup [ ] #####
	$ cd /
	$ cat /etc/passwd > a
	$ cat /etc/shadow > b
	$ iptables -L > c
	$ ls -a /tmp > d
	$ netstat -plant > e
	$ crontab -l > f

##### check.sh File [ ] #####
	# Create a file called check.sh in the root directory (which is /)
	cd /
	vim check.sh
	# Write this to the file
	cd /;
	echo "";
	diff a /etc/passwd >/dev/null || echo "new user";
	diff b /etc/shadow >/dev/null || echo "new password";
	sudo iptables -L | diff c - >/dev/null || echo "new firewall rules";
	sudo ls -a /tmp | diff d - >/dev/null || echo "new file in /tmp";
	sudo netstat -plant | diff e - >/dev/null || echo "connections are different";
	sudo crontab -l | diff f - >/dev/null || echo "new crontab";
	echo "Check Complete.";
	# Now give it execute permissions
	chmod +x check.sh

##### Verify [ ] #####
	# Check that message appears on-screen every minute
	# Validate output- everything should be fine since we just hardened the box

---

## Ongoing Tasks ##

##### Monitor IDS #####
	1. Make sure that cronjob does not stop, if nothing new is popping up on screen then it is not working
	2. If malware is discovered, change all passwords again
	3. Write down all incidents and try to figure out how they happened
	4. Write malicious IPs down on Blacklist Board

##### Services Hardening #####
	See services packet

	# Upgrade everything
	$ apt-get upgrade
	# List services
	$ systemctl list-unit-files --type=service | grep enabled
	# Disable service
	$ systemctl disable service_name

---
---

## Helpful Stuff ##

##### Check for SSH Logins #####
	# Debian Clones
	-View all failed login attempts
	$ cat /var/log/auth.log | grep 'sshd.*Failed'
	-View all successful logins
	$ cat /var/log/auth.log | grep 'sshd.*opened'
	# RedHat/CentOS Clones
	-View all failed login attempts
	$ cat /var/log/auth.log | grep 'sshd.*Failed'
	-View all successful logins
	$ cat /var/log/secure | grep 'sshd.*opened'

##### Check for Specific Changed Password #####
	$ cat /etc/passwd | cut -d: -f1 | diff ~/allusers.txt -

##### Show All Terminals #####
	who -a

##### SSH on Non-standard Port #####
	1. Open /etc/ssh/sshd_config in your favorite text editor:
	$ vi /etc/ssh/sshd_config
	2. Browse to the following line:
	# Port 22
	3. Uncomment and edit this line to reflect the new port.
	# Port 2255 (this can be set to any non standard port)
	4. Save and quit the file, and restart ssh.
	$ /etc/init.d/sshd restart
	5. If you are connected to the server via ssh on port 22, 
	your connection will drop and you will need to reconnect using the new port.

	New SSH Port: __________________
