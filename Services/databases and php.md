# PostgreSQL

### Interacting with the database
	psql # enter psql command line
	sudo -u [username] psql # password less login, username may be postgres
	\password [username] # change password
	\l # list the databases
	\c [dbname] # connect to a database
	\dt # list the tables
	\deu+ # list user mappings
	\du # list roles
	\q # quit
	REASSIGN OWNED BY [old_username] TO [new_username]; # change parent user
	DROP USER [username]; # delete user
	DROP OWNED BY [username]; # drop privileges for user
	psql -c "[command]" # do command in one line


### Configuration Files
Configuration files are most likely at /var/lib/pgsql/data/

Might also be /etc/postgresql/[version]/main/

If not, try:
	psql -U [username]-d [dbname]-c 'SHOW config_file;'

Port is in postgresql.conf, default is 5432 or 5433


### pg_hba.confg
Connection whitelist based on username, database, and source IP

	host [database] [username] [address] [auth-method]
	# can use all keyword for database and username
	# auth-method should be md5, use reject to reject
	# change host to hostssl if ssl possible
	# then restart:
	/etc/init.d/postresql restart
	service postgresql restart # or this


### Backup and restore
	pg_dump [database] > [filename].sql # backup
	pg_restore [filename].sql # restore, add -C if database doesn't exist


# MySQL
### Reset root password on Windows
	mysql -u root
	mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('[password]');
	mysql> SET PASSWORD FOR 'root'@'127.0.0.1' = PASSWORD('[password]');
	mysql> SET PASSWORD FOR 'root'@'%' = PASSWORD('[password]') 


### Reset root password on Unix
	mysql -u root
	mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('[password]');
	mysql> SET PASSWORD FOR 'root'@'127.0.0.1' = PASSWORD('[password]');
	mysql> SET PASSWORD FOR 'root'@'[hostname]' = PASSWORD('[password]');
	# or:
	mysqladmin -u root password “[password]”
	mysqladmin -u root -h localhost password “[password]”


### Rename root username
	mysql> RENAME USER root TO [username];
	
	# if earlier than 5.0.2:
	mysql> use mysql;
	mysql> update user set user="[username]" where user="root";
	mysql> flush privileges;


### Drop test database
	mysql> drop database test;
	mysqladmin -u [username] -p[password] drop test # or this


### Remove anonymous and other accounts
	mysql> select * from mysql.user where user=""; # will print something if anonymous account exists
	mysql> DROP USER ""; # removes anonymous account

# if earlier than 5.0:
	mysql> use mysql;
	mysql> DELETE FROM user WHERE user="";
	mysql> flush privileges;
	
	mysql> use mysql;
	mysql> select * from users; # lists users
	mysql> DROP USER "[username]"; # delete user


### File permissions on Unix
	ls -l /var/lib/mysql

Above directory should be owned by user mysql and group mysql
	ls -l /usr/bin/my*

Above files should be owned by user root or mysql

Only root and mysql should have access to [datadirectory]/[hostname].err

Only root and mysql should have access to [datadirectory]/*logfileXY


### Delete history
	cat /dev/null > ~/.mysql_history


### Configuration File

On Windows, look for my.ini in  C:\Program Files\MySQL\[version]\


	mysql --help | grep "Default options" -A 1 # get location in Unix


Add `skip-networking` to [mysqld] section to restrict MySQL from opening a network socket, don't add if remote access needed

Add `skip-show-database` to [mysqld] section to disallow any user from getting database names

Add `[mysqld] log =/var/log/[filename]` to [mysqld] section to log all queries, users root and mysql should have write access to this file

Add `set-variable=local-infile=0` to [mysqld] section to not allow local files to be accessed

If local infile needed:

	mysql> SELECT load_file("[filepath]" # eg /etc/passwd would be a good idea)



### Port
	SHOW VARIABLES WHERE Variable_name = 'port'; # default port is 3306


### Grants

To allow a host to connect remotely, use:

	mysql> GRANT SELECT, INSERT ON [database].* TO '[username]'@'[host]';
	mysql> show grants for ‘[username]’@’localhost’; # lists grants for user


### Backup and Restore
	mysqldump --opt -u [username] -p[password] [database] > [filename].sql # backup
	mysql -u [username] -p[password] [database] < [filename].sql # restore


# PHP

### Configuration file
	php --ini # prints configuration file path

Add `expose_php = Off` to disable showing PHP version in HTTP header

Add this to disable insecure functions `disable_functions = phpinfo, mail, exec, passthru, shell_exec, system, proc_open, popen, curl_exec, curl_multi_exec, parse_ini_file, show_source`

Add `file_uploads = Off` to disable HTTP file upload

Add `display_errors = Off` to disable error messages

Add `safe_mode_allowed_env_vars = PHP_` to limit external access to PHP environment

Add `register_globals = Off` to turn off globals registration for input data

Add `allow_url_fopen = Off` and `allow_url_include = Off` to restrict opening of remote files

Add `cgi.force_redirect = 0` to ensure appropriateness of PHP redirecting

Add `logging log_errors = On` to log all errors

Add `safe_mode = On` to enable safe mode

Add `sql.safe_mode = On` to enable SQL safe mode

Add `open_basedir="/var/www/html/"` to only allows PHP to access files ere


### Permissions
	chown -R apache:apache /var/www/html/
	chmod -R 0444 /var/www/html/
	find /var/www/html/ -type d -print0 | xargs -0 -I {} chmod 0445 {}


### Backdoors
	grep -iR 'c99' /var/www/html/
	grep -iR 'r57' /var/www/html/
	find /var/www/html/ -name \*.php -type f -print0 | xargs -0 grep c99
	grep -RPn "(passthru|shell_exec|system|base64_decode|fopen|fclose|eval)" /var/www/html/


### Disable loadable PHP modules

Find them in /etc/php.d/ and disable unnecessary ones:
	php -m # list modules
	cd /etc/php.d/
	mv [module].{ini,disable}
	/sbin/service httpd restart


