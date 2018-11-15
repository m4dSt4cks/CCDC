## Apache ##

### Ports required ###
80 in, 443 in for HTTPS

Your website is probably in /var/www/html

Make a backup of this:

mkdir /bak
<br>
cp -r /var/www/html/ /bak


### Ensure Apache not running as root ###
/etc/apache2/envvars

Check APACHE\_RUN\_USER and APACHE\_RUN\_GROUP

### To migrate Apache to non-privileged user ###
groupadd -r apache
<br>
useradd apache -r -g apache -d /var/www -s /sbin/nologin

### Set user and group in /etc/apache2/envvars ###

### Directory permissions ###

/var/www/html should be 744

### Check what directories are being served ###
/etc/apache2/sites-enabled

### Change security settings ###
vi /etc/apache2/sites-available/default < or whatever your site is
<br>
Under Directory /, change:
<br>
		`Options FollowSymlinks`
<br>
		to
<br>
		`Options -FollowSymlinks -ExecCGI -Includes -Indexes`
<br>
	
Under Directory /var/www, change:
<br>
	`Options Indexes FollowSymlinks MultiViews`
<br>
	to
<br>
	`Options -Indexes -FollowSymlinks`
<br>
	`Options -ExecCGI`
<br>
	
vi /etc/apache2/conf.d/security
<br>
Ensure ServerTokens set to Prod
<br>
Ensure ServerSignature set to Off
<br>
Ensure TraceEnable set to Off