This repo original was used for nagios-crashplan-linux/check_crashplan_pro_linux but I decided to open it up to contain other scripts to deploy amongst all my nagios servers and hosts.

check-apt-upgrade.pl
=====================

Script to check updates on Ubuntu/Debian systems using SNMP see http://www.logix.cz/michal/devel/nagios/

==

System Up-To-Date monitor (APT / YUM)

Keeping operating system up to date with latest patches is essential in most environments. Following two scripts check whether new patches are available for download. The first one works with APT based systems (e.g. Debian, Ubuntu or even OpenSolaris Nexenta, ...) and the second one is for YUM based systems (e.g. RedHat, Fedora or CentOS).

You can make the script itself to run apt or yum to download information about newest updates from the internet, but that usually takes too long time and Nagios usually in the middle timesout. Instead on my systems I run apt or yum every few hours from cron and let the Nagios script only do the quick tasks. More specifically - on APT systems I run apt-get update from cron, because that download data from the net and then run apt-get -q -s upgrade from SNMP script because that only reads local database and parses the output. On YUM systems I run yum check-update from cron and store its output in a file. Then from SNMP script I only read and parse that file. Keep reading for example usage.

First of all tell cron to run apt or yum every 6 hours:

## /etc/crontab
## Every 6 hours download new data from the net
# For APT-based systems (Debian, Ubuntu, ...)
50 */6 * * *   root    /usr/bin/apt-get -qq update
# For YUM-based systems (RedHat, CentOS, Fedora)
50 */6 * * *   root    /usr/bin/yum check-update > /var/run/yum.check-update
Download the nagios scripts (check-apt-upgrade.pl or check-yum-update.pl ) and tell SNMP daemon about them:

## /etc/snmp/snmpd.conf
# For APT-based systems (Debian, Ubuntu, ...)
extend sw-updates /usr/local/bin/check-apt-upgrade.pl --run
# For YUM-based systems (RedHat, CentOS, Fedora)
extend sw-updates /usr/local/bin/check-yum-update.pl --file /var/run/yum.check-update
Last step is to configure Nagios to poll these services (same for both APT and YUM):

define service{
	use			generic-service
	host_name		remote.server
	service_description	Software updates
	check_command		check_snmp_extension!sw-updates
}
That's all. You will get "OK" result if there are no new updates and "WARNING" with a list of packages to update if there are any. On Ubuntu you'll even get "CRITICAL" result if there are any security updates, and "WARNING" when there are only non-security ones.

Have you found this script useful? Please support author by PayPal donation.

check-yum-update.pl
=====================

Script to check updates on RedHat/CentOS systems using SNMP see http://www.logix.cz/michal/devel/nagios/ or see check-apt-upgrade.pl section above


check_crashplan_pro_linux
======================

Nagios Plugin to check CrashPlan Pro Backups on Linux