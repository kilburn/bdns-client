# Setup domains on a Secondary DNS server #

bdns-client is a small utility to automate the addition and removal of domains to the a secondary DNS server.

## Requirements ##

The utility is written in php (so you need the cli version of php). In short:

	php >= 5.1

## Installing ##

To install bdns-client it is necessary to perform three basic steps:

### Install the required files ###

First of all, you should copy the required files into appropriate locations in your filesystem. You can change the routes displayed here so long as you update the configuration file accordingly.

	cp etc/ovh-dnsupdater /usr/local/etc/bnds
	chmod 640 /usr/local/etc/bnds
	cp ovh-dnsupdater /usr/local/sbin/ovh-dnsupdater
	chmod 755 /usr/local/sbin/ovh-dnsupdater

### Setup the utility ###

The next step is to modify the configuration file to reflect your originating IP and the details of your server. All these settings are found in the /usr/local/etc/bdns file (or wherever you have put the configuration).

### First run and cronjob setup ###

Next, it is recommended that you perform a dry run at this (the utility logs what would be done, but does nothing) to avoid breaking anything:

	bdns-client --foreground --dry-run

If everything looks fine and there are changes, you can re-run the utility without the dry run option:

	bdns-client --foreground

Finally, you should setup cron jobs to automate the execution of the utility. The recommended way is to setup a frequently-running (every 5mins or so) task, ensureing that this server quickly syncs with the backup dns.

This can be easily specified in /etc/crontab by adding the following lines:

	*/5	*	*	*	*  root /usr/local/sbin/bdns-client

Alternatively, some systems allow you to define cronjob tasks by creating files inside the /etc/cron.d folder. If this is your case, then you can just copy the included sample file:

	cp cron/bdns-cient /etc/cron.d/bdns-client
	chmod 755 /etc/cron.d/bnds-client

## bdns-client usage ##

	Usage: 
	  $programName [options]
	Options:
	  --conf-file=<file>
		Load the configuration from file <file> instead of /usr/local/etc/bdns.
	  --dry-run    
		Just show what would be done, without actually doing anything.
	  --foregound
	  	Show log messages instead of logging to syslog.
	  -h, --help   
		Show this help message.

## How it works ##

bdns-client works by monitoring your bind's zone folder, and updating the secondary server whenever a change is detected. Essentially, each time that the utility runs, it performs the following steps:

 1. Fetch the list of domains configured in the backup server
 2. Fetch the list of locally setup domains
 3. Compute the difference between these lists
 4. Add/Remove domains so that both lists end up synchronized
