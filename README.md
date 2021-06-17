# Backups

Step 1: Create, Extract, Compress, and Manage tar Backup Archives
Command to extract the TarDocs.tar archive to the current directory: tar xvvf TarDocs.tar ~/Projects
 
Command to create the Javaless_Doc.tar archive from the TarDocs/ directory, while excluding the TarDocs/Documents/Java directory:
tar cvf Javaless_Docs.tar --exclude= “TarDocs/Documents/Java” TarDocs/


Command to ensure Java/ is not in the new Javaless_Docs.tar archive:
tar tvf Javaless_Docs.tar | grep Java
Bonus
Command to create an incremental archive called logs_backup_tar.gz with only changed files to snapshot.file for the /var/log directory:
sudo tar --listed-incremental=snapshot.file -cvzf logs_backup.tar.gz /var/log
Critical Analysis Question
Why wouldn't you use the options -x and -c at the same with tar?
-x: extracts and writes the files to disk.Because one command creates the file for the archive and then another command executes the tar. -c: creates the name of the directory where the files will be placed. Conflict of interest. 

# Step 2: Create, Manage,  and Automate Cron Jobs
Cron job for backing up the /var/log/auth.log file: 2 * * * * tar -cvzf /auth_backup.tgz /var/log/auth.log

# Step 3: Write Basic Bash Scripts
Brace expansion command to create the four subdirectories:
mkdir ~/backups/{freemem,diskuse,openlist,freedisk}

system.sh script edits below:

 #!/bin/bash
free -h > ~/backups/freemem/free_mem.txt
free -h > ~/backups/diskuse/disk_usage.txt
free -h > ~/backups/openlist/open_list.txt
free -h > ~/backups/freedisk/free_disk.txt
Command to make the system.sh script executable: sudo chmod +x system.sh


Commands to test the script and confirm its execution:
cd ~/backups/diskuse, cd ~/backups/freedisk, cd ~/backups/openlist,                 cd ~/backups/free_mem
cat disk_usage.txt, cat free_disk.txt, cat open_list.txt, cat free_mem.txt
Bonus
Command to copy system to system-wide cron directory: sudo cp  ~/system.sh /etc/cron.weekly

# Step 4. Manage Log File Sizes
Run sudo nano /etc/logrotate.conf to edit the logrotate configuration file.

 Configure a log rotation scheme that backs up authentication messages to the /var/log/auth.log.


Add your config file edits below:


/var/log/auth.log {
        weekly
        rotate 7
        notifempty
        delaycompress
        missingok
}

# Check for Policy and File Violations
Command to verify auditd is active: systemctl status auditd


Command to set number of retained logs and maximum log file size:sudo nano /etc/audit/auditd.conf


Add the edits made to the configuration file below:


max_log_file = 35
num_logs = 7
Command using auditd to set rules for /etc/shadow, /etc/passwd and /var/log/auth.log:sudo nano /etc/audit/rules.d/audit.rules

Add the edits made to the rules file below:

-w /etc/shadow -p rwa -k hashpass_audit
-w /etc/passwd -p rwa -k userpass_audit
-w /var/log/auth.log -p rwa -k authlog_audit

Command to restart auditd: sudo systemctl restart auditd 


Command to list all auditd rules: sudo auditctl -l


Command to produce an audit report: aureport –au


Create a user with sudo useradd attacker and produce an audit report that lists account modifications:sudo aureport -m


Command to use auditd to watch /var/log/cron: sudo auditctl -w /var/log/cron


Command to verify auditd rules:sudo auditctl -l



 Perform Various Log Filtering Techniques
Command to return journalctl messages with priorities from emergency to error: journalctl -b -p emerg..err


Command to check the disk usage of the system journal unit since the most recent boot:sudo journalctl -b -u systemd -journald


Command to remove all archived journal files except the most recent two: sudo journalctl --vacuum-time=2
Command to filter all log messages with priority levels between zero and two, and save output to /home/sysadmin/Priority_High.txt:                                                                        sudo journalctl -p 0..2 > /home/sysadmin/Priority_High.txt


Command to automate the last command in a daily cronjob. Add the edits made to the crontab file below:  sudo cp ~/Priority_High.sh /etc/cron.daily

#!/bin/bash
@daily journalctl -p 0..2 >> ~/home/sysadmin/Priority_High.txt


