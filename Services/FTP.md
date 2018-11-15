## FTP
### CCDC Prep 2018
Written by Ryan Ramseyer  
USAF Academy Cyber Competition Team  
February 2018

## Windows
**Windows FTP Server on Windows Server 2008 (IIS ~6.0+) and newer**

**Allow incoming/outgoing ports 20 and 21**

### 1. Backup files
We want to know what the clean-state was and tell if anything has been changed. Also check for potentially sensitive information.

### 2. If IIS is not installed:
Install so we can actually manage our box.

- Open *Windows Server Manager*, go to *Roles*, and in the *Roles Summary* panel, select *Add Roles*.
- In *Add Roles*, proceed to *Server Roles* and check *Web Server (IIS)* role.
- In *Add Roles*, proceed to *Role Services* and check *FTP Server > FTP Service* role service. Make sure *Management Service > IIS Management Console* service is checked.
- Follow the rest of the install wizard and click *Install*.

### 3. FTP Logging

FTP Logging -->

- Choose directory
- Schedule for hourly new log files
- Apply

### 4. User Isolation
Configure the starting directory for users -- **if FTP files are not shared among users**.

FTP User Isolation --> 

- Isolate users: Restrict users to the following -directory*select *User name physical directory (enable global virtual directories).
- Apply

### 5. Disable anonymous authentication
**This is good but not absolutely necessary. Check the FTP server packet page on the day of competition - users may not have login information and if not using SFTP passwords will be sent in plaintext. Anonymous is fine (server is read-only) unless there are sensitive files on the system.**

FTP Authentication -->

- On the *FTP Authentication* page, select *Anonymous Authentication*.
- Ensure that *Disabled* is selected for this option in the *Home* panel.

### 6. SFTP, if you can
This will be based on certificate availability and setup.

FTP SSL Settings

## Linux

### Open SSH --> Port 22
Config file located in /etc/ssh/sshd_config
#### Checklist
1. Disable root login

	Comment out "PermitRootLogin without-password" and add "PermitRootLogin no" in /etc/ssh/sshd_config

2. Disable anonymous access

#### Additional (no particular order)
1. Don't enable port forwarding on router
2. Use non-standard SSH port (something other than 22) --> this requires editing firewall rules
3. Allow only local connections (maybe too restrictive)
	
### Vsftpd
"Very Secure File Transfer Protocol Daemon"
Config file location: /etc/vsftp.conf

#### Checklist

1. Enable local user login

	Uncomment "#local_enable=YES"

2. Disable anonymous access
	
	Change line "anonymous\_enable=YES" to "anonymous\_enable=NO"

3. Use encryption

		To use vsftpd with encryption (it's safer), change or add the following options 
		(some options aren't on the original config file, so add them): Code: ssl_enable=YES
		allow_anon_ssl=NO
		force_local_data_ssl=YES
		force_local_logins_ssl=YES
		ssl_tlsv1=YES
		ssl_sslv2=NO
		ssl_sslv3=NO
		# Filezilla uses port 21 if you don't set any port
		# in Servertype "FTPES - FTP over explicit TLS/SSL"
		# Port 990 is the default used for FTPS protocol.
		# Uncomment it if you want/have to use port 990.
		# listen_port=990

#### Interesting/Useful Options:
1. cmds\_allowed
	
	Specifies a comma separated list of FTP commands that the server allows. Any other commands are rejected. There is no default value for this directive.
2. max\_clients
	
	Limit the maximum number of client connections
3. max\_per\_ip
	
	Limit the number of connections by source IP address
4. anon\_max\_rate
	
	Set the maximum rate of data transfer per anonymous login 
5. local\_max\_rate
	
	Set the maximum rate of data transfer per non-anonymous login 

#### Other things to think about:

1. Be careful for VSFTPD version 2.3.4. There is a backdoor where you can login and get shell using only ":)" as the username.