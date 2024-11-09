 # Troubleshooting DShield Sensor

https://github.com/DShield-ISC/dshield/blob/main/STATUSERRORS.md<br>
https://github.com/DShield-ISC/dshield/blob/main/docs/general-guides/Troubleshooting.md<br>

## DShield Sensor Setup
The sensor only support 64-Bit OS on the PI and any combination of VM or hardware sensor<br>

## Review Error Logs
After the installation or upgrade, you can review the errors during the installation<br>
sudo cat /srv/log/isc-agent.err

## Reset & Reinstall
This command is used to reset the DShield sensor and remove everything.<br>

$ sudo ./bin/quickreset.sh<br>
$ sudo ./bin/install.sh<br>

## Testing Connection to ISC Website
Testing connection to ISC website<br>
curl -s 'https://isc.sans.edu/api/portcheck?json'<br>

## Testing iptables Logs
Testing connection to the ISC website to confirm the iptables logs are able to upload to the site. If it gives an error, provide the error in Slack to check what is the issue.<br>
$ sudo /srv/dshield/fwlogparser.py

## Cannot Login to DShield Sensor

$ ps -aef | grep 12222<br>

If TCP/12222 isn't listening to run the following commands:<br>

$ sudo systemctl status ssh<br>

If not listening<br>

$ sudo systemctl start ssh<br>
$ sudo systemctl status ssh<br>
$ sudo systemctl enable ssh<br>

## sudo to cowrie

sudo su cowrie -<br>

## Editing the Cowrie Configuration File
This is the location for the DShield sensor configuration. This file also contains the IP address your are going to be accessing the sensor from<br>
$ sudo vi /etc/dshield.ini<br>

## Troubleshooting DShield isc-agent

Sometimes thew isc-agent has errors such as this picture:<br>

![isc_agent_error](https://github.com/user-attachments/assets/b9c3e43b-4de5-444d-960a-7e1c84634362)

If _/var/log/dshield.log_ is missing do the following to create an empty file:<br>
$ sudo touch /var/log/dshield.log<br>
$ sudo chown  syslog:adm /var/log/dshield.log<br>

Run the following command to check the isc-agent status:<br>

$ sudo systemctl status isc-agent<br>

If it isn't running, try to start it after with this:<br>

$ sudo systemctl start isc-agent<br>
$ sudo systemctl restart isc-agent<br>
$ sudo systemctl status isc-agent<br>
$ sudo systemctl enable isc-agent<br>

## Extend Log Storage Past 8 Days
To keep logs longer than 7 days, you need to edit:<br>
$ sudo vi /etc/cron.d/dshield<br>

Change the -ctime from +7 to the lenght you wish:<br>

0 6 * * * root find /srv/db -name 'webhoneypot*json' -ctime +7 -delete<br>
0 10 * * * root find /srv/cowrie/var/log/cowrie -name 'cowrie.*' -ctime +7 -delete<br>

## PI Won't start IPTables & isc-agent

Add root cronjob to delay start if part to the PI won't start the isc-agent:<br>

@reboot sleep 60 && systemctl start isc-agent<br>

## Sensor Status and Listening Ports

$ sudo /srv/dshield/status.sh<br>
$ sudo lsof -i -P -n | grep LISTEN<br>
<pre>
guy@collector:~$ sudo lsof -i -P -n | grep LISTEN
systemd      1            root  233u  IPv6   7564      0t0  TCP *:12222 (LISTEN)
systemd-r  556 systemd-resolve   15u  IPv4   8331      0t0  TCP 127.0.0.53:53 (LISTEN)
systemd-r  556 systemd-resolve   17u  IPv4   8333      0t0  TCP 127.0.0.54:53 (LISTEN)
python3    962            root    9u  IPv4  10880      0t0  TCP *:8000 (LISTEN)
python3    962            root   10u  IPv4  10881      0t0  TCP *:8443 (LISTEN)
master    1355            root   13u  IPv4  11561      0t0  TCP *:25 (LISTEN)
sshd      1386            root    3u  IPv6   7564      0t0  TCP *:12222 (LISTEN)
twistd    3050          cowrie   11u  IPv4  22593      0t0  TCP *:2222 (LISTEN)
twistd    3050          cowrie   12u  IPv4  22594      0t0  TCP *:2223 (LISTEN)
</pre>

## Troubleshooting Filebeat Connection to Logstash
````
sudo su -<br>
filebeat test config<br>
````
Expected output: _Config OK<br>_
````
filebeat test output<br>
````
Expected output:
<pre>
logstash: 192.168.25.231:5044...
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 192.168.25.231
    dial up... OK
  TLS... WARN secure connection disabled
  talk to server... OK
</pre>
If filebeat failed to connect to logstash, check the firewall and that logstash is running on the ELK server<br>
````
sudo iptables -nL
sudo iptables -F   (to flush the firewall)
sudo docker restart logstash (restart logstash service)
````
## Logs Location
**Enable TTYLog** by editing this file. It is important to remember when you  the DShield sensor this will reset to false.<br>
Look for: ttylog = false to ttylog = true<br>
$ sudo vi /srv/cowrie/cowrie.cfg 

/var/log/dshield.log -> firewall<br>
/srv/db -> webhoneypot-*.json<br>
/srv/cowrie/var/lib/cowrie/downloads<br>
/srv/cowrie/var/log/cowrie/ -> Logs<br>
/srv/cowrie/var/lib/cowrie/tty -> tty logs (if you have enabled them)<br>

## Router Port Forwarding or DMZ
The easiest way of getting your DShield sensor expose it to add it to the **DMZ** of your router.<br>
This is an example of custom router port forwarding to the DShield sensor.<br>
https://github.com/bruneaug/DShield-SIEM/blob/main/Troubleshooting/DShield_Sensor_Port_Forwardng_Example.PNG<br>

If you get this error, your webserver might not be exposed and need to check port forwarding<br>

![image (1)](https://github.com/user-attachments/assets/1f8bd54b-7793-4de6-a1fb-b9cfa3ded2d5)

## Disable WIFI
This command should disable WIFI on the PI<br>
$ sudo nmcli radio wifi off<br>

## Config file
This is the DShield sensor configuration file<br>
sudo vi /etc/dshield.ini<br>

## Remote Sensor Login

SSH is refused by the sensor<br>

Login the sensor and watch the syslog with trying to remotely login<br>
Do you have a public key in the sensor from a specific host (must used the same host that key came from)<br>
If you do, try removing the public key and try to login again<br>

$ sudo grep your_account /var/log/syslog<br>

## IPTables Issues
These steps is for troubleshooting Ubuntu OS. The iptables firewall is located:<br>
$ sudo cat /etc/network/iptables

Firewall won't start:<br>
Lookin for any forms of errors in syslog that might indicate why it isn't starting.<br>
$ sudo grep -i error /var/log/syslog | grep iptables

Normal output of firewall when checking the status:<br>
$ sudo systemctl status dshieldfw
<pre>
dshieldfw.service - DShield Firewall Configuration
  Loaded: loaded (/etc/systemd/system/dshieldfw.service; enabled; preset: enabled)
  Active: inactive (dead) since Fri 2024-08-09 16:58:28 UTC; 3h 53min ago
Duration: 1.616s
    Docs: https://isc.sans.edu
Main PID: 642 (code=exited, status=0/SUCCESS)
     CPU: 19ms

Aug 09 16:58:26 collector systemd[1]: Started dshieldfw.service - DShield Firewall Configuration.
Aug 09 16:58:28 collector systemd[1]: dshieldfw.service: Deactivated successfully.
****
</pre>
````
sudo systemctl restart dshieldfw
sudo systemctl status dshieldfw
````
## Custom IPTables Rules
From Dr J.<br>
Added local iptables rules that will be applied in addition to the automatic rules created by the install script. The intent is to allow you to add more flexible rule to allow access to the honeypot.<br>
This has been an issue in particular for cloud based honeypots.<br>
The rules are applied after the default rules are applied. A small dummy file with instructions is created by default.<br>
```` 
sudo vi /etc/network/iptables.local
````
## Update the DShield Sensor
````
cd dshield
sudo git pull
sudo bin/install.sh --update
sudo reboot
````
## Check if DShield Sensor is Receiving any Traffic
If after checking the log files they are not storing log, use tcpdump to check if the sensor is receiving packets via port forwarding or the DMZ exposure.<br>
````
ip address -> for the interface
sudo tcpdump -nni eth0 -c 1000 'host 192.168.25.105 and not dst net 192.168'
````
## OS Update
````
sudo apt-get update
sudo apt-get upgrade
sudo reboot
````
## Sensor Log Backup
You are encourage to setup some kind of backup for your logs since the sensor is setup by defaul to keep the cowrie logs for 8 days and the iptables only daily.<br>
These scripts can be used to backup you logs nightly<br>
https://github.com/bruneaug/DShield-SIEM/blob/main/AddOn/Backup_DShield_Sensor_Logs.md<br>
Jesse's example of data backup<br>
https://isc.sans.edu/diary/DShield+Honeypot+Maintenance+and+Data+Retention/30024

## Checking Cowrie Logs in ISC
Access your logs at this location on the ISC website. The 3 highlighted are the logs populating your data in your account.<br>
![image](https://github.com/user-attachments/assets/e8a10de8-49c9-4f6f-bd5a-ddd2bc34f547)

## DShield Sensor in Cloud
If you are setting up a sensor in one of the cloud and your home IP isn't static, consider setting up a script (example below) to check your home IP and update the firewall as necessary so you don't loose access to your sensor. <br>
https://isc.sans.edu/diary/DShield+Sensor+Setup+in+Azure/29370<br>

### AWS Recovery - Suggestions from Jesse

AWS failed to access via TCP/12222 - Jesse recommend trying the EC2 serial console or the EC2 Instance Connect to reconnect to the sensor. If that fails, you may have to rebuild the sensor.<br>

I'd focus on getting working data and go back to try and recovery that honeypot VM if you're interested. It looks like you may be able to use a similar snapshot recovery process that I've used in AWS Lightsail, based on a reddit recommendation [1]. <br>
If you didn't move any of the log data to a different location on your honeypot, you'll only be losing 7 days of local data. It may not be worth the effort to recover.<br>
 If you cannot ssh to an instance, and assuming you did not configure EC2 Session Manager (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/session-manager.html)<br>
 or EC2 instance connect (https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/), then your only chance is:<br>
<pre>
1 - Create a snapshot of your root disk
2 - launch a new EC2 instance.
3 - Mount the snapshot on the healthy instance, verify the configuration and make any changes if needed.
4 - Unmount the disk
5 - Stop your unhealthy instance, replace the root disk with the new on
[1] https://www.reddit.com/r/aws/comments/hhdxbg/locked_out_of_ec2_instance/
</pre>

## SSH to Remote Sensor
This option has been suggested by Michael Tigges.
This gets stored in ~/.ssh/config and invoked with ssh honeypot or ssh jumpbox respectively.
<pre>
$ touch ~/.ssh/config
$ chmod 600 ~/.ssh/config
If needed, build your private/public keys.
$ ssh-keygen
### Jumpbox
Host jumpbox
  HostName JUMPBOX-IP
  Port 22
  User root
  IdentityFile ~/.ssh/_docean

### Honeypot
Host honeypot
  HostName HONEYPOT-IP
  User root
  ProxyJump jumpbox
  IdentityFile ~/.ssh/dshield
  Port 12222
</pre>
Login the honeypot with this simple command:<br>
$ ssh honeypot<br>
# Troubleshooting DShield SIEM

## Installing DShield SIEM on PI 5
This is an example of installation of using a PI 5 by a former Internship Student<br>
https://github.com/amelete11235/homelab/blob/main/Installing%20DShield%20SIEM%20on%20a%20Raspberry%20Pi%205%20-%208%20GB%20RAM/Installing%20DShield%20SIEM%20on%20a%20Raspberry%20Pi%205%20-%208%20GB%20RAM.md

## DShield SIEM main page is missing some data
If the main interface is only showing some logs and not all of them like the example below, check the tables next to make sure if have been loaded correcly.<br>
This picture shows only the iptables logs. Something isn't configured correctly.

![DShield_Interface_Missing_logs](https://github.com/user-attachments/assets/09832dd1-7fff-45ba-b10e-3c0b7eb2b298)

Check the folowing tables to make sure they are correctly loaded with the date as an extension?<br>
Go to Management -> Data -> Index Management<br>
Type **cowrie** in the search box<br>
Do they all look like this with the date? You can also see there is data in the indices<br>

![image](https://github.com/user-attachments/assets/623df8e9-ceb0-4a44-b635-16dc91aef143)

If there is data in the indice, you need to refresh the cowrie* data table to force a reload<br>
You need to do this step to refresh the index:<br>
In the Kibana console<br>
Go to Management -> Kibana -> Data Views -> select cowrie*<br>
Select **Edit** in the upper Right corner<br>
Select **Save** to refresh the index<br>
Note: Sometimes you might get an error and you have to click **Save** twice to save the changes.

## Resetting the elastic User Password

Login in your VM with the following command to get into the elastic container:
````
sudo docker exec -ti es01 bash
````
Run the following command to reset the elastic user password:
````
bin/elasticsearch-reset-password --url "https://127.0.0.1:9200" --username elastic -i
````

# Docker Troubleshooting Commands
This is a list of command that can be useful to troubleshoot issues with the docker.<br>
https://github.com/bruneaug/DShield-SIEM/blob/main/Troubleshooting/docker_useful_commands..md

## Docker Start with Errors
To troubleshoot why some components are failing to start, start the docker this way without going in deamon mode:<br>
````
sudo docker compose up --build
````

# Linux Commands

SANS Linux Essential Cheet Sheet https://sansorg.egnyte.com/dl/T2c7pGW9p0

Official JQ site: https://jqlang.github.io/jq/download/<br>
CSVJSON: https://csvjson.com/json2csv<br>
Convert JSON to CSV: https://www.convertcsv.com/json-to-csv.htm

JQ paper for parsing various cowrie logs by Kaela Reed<br>
https://isc.sans.edu/diary/The+Art+of+JQ+and+Commandline+Fu+Guest+Diary/31006

Paper for corralation DShield Sensor data by Joshua Jobe<br>
https://isc.sans.edu/diary/Is+that+It+Finding+the+Unknown+Correlations+Between+Honeypot+Logs+PCAPs+Guest+Diary/30962

Parsing cowrie logs with Python by Josh Jobe<br>
https://github.com/jrjobe/DShield-Cowrie-json-Parser

## Various Linux Commands
$ sudo last                             -> shows who was login the beside your account<br>
$ netstat -an | grep 5044               -> Check if the port is active and connected to an IP<br>
$ w                                     -> Shows who else is login<br>
$ uptime                                -> Shows how long the server has been running without a reboot<br>
$ crontab -l                            -> List all the active cronjobs<br>
$ netstat -an | grep '5601\\|9200\\|5044'  -> Shows if Elasticsearch and logstash are listening<br>


