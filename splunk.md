# Install Splunk 8.2.2 Enterprise Instance on Ubuntu 20.4 & How to forward data to Splunk Enterprise
### [Click here](https://www.splunk.com/en_us/download/splunk-enterprise.html) and select Linux --> .deb file to install on Ubuntu. Click on download and select command line(wget)
#### - Login to Ubuntu Machine and Install Splunk 8.2.2
```
wget -O splunk-8.2.2-87344edfcdb4-linux-2.6-amd64.deb 'https://d7wz6hmoaavd0.cloudfront.net/products/splunk/releases/8.2.2/linux/splunk-8.2.2-87344edfcdb4-linux-2.6-amd64.deb'
sudo dpkg -i splunk-8.2.2-87344edfcdb4-linux-2.6-amd64.deb
```
#### - Start Splunk at boot, enter administrator username and password (accept license)
```
sudo /opt/splunk/bin/splunk enable boot-start
```
> provide the **username**: admin, **password**:<our choice, Kadambattu@123!> 

#### - Start the Splunk service
```
sudo service splunk start
```
#### - Login to the web interface, type localhost:8000, in our case it will be 137.135.25.217:8000 with above admin and its password.

# - How to forward data to Splunk Enterprise
The most common way to use the universal forwarder is to send data to a Splunk Enterprise indexer or indexer cluster
#### - Install Splunk Universal Forwarder  8.2.2 on the machine from which we need to send the logs, data, etc. Here my machine is RaviTestVM/137.117.11.242

#### - Install Splunk Universal Forwarder
```
wget -O splunkforwarder-8.2.2-87344edfcdb4-linux-2.6-amd64.deb 'https://d7wz6hmoaavd0.cloudfront.net/products/universalforwarder/releases/8.2.2/linux/splunkforwarder-8.2.2-87344edfcdb4-linux-2.6-amd64.deb'
sudo dpkg -i splunkforwarder-8.2.2-87344edfcdb4-linux-2.6-amd64.deb
root@RaviTestVM: cd /opt/splunkforwarder/bin
root@RaviTestVM:/opt/splunkforwarder/bin# ./splunk start --accept-license
create the username: admin and password: Kadambattu@123!
```
### - Enable boot-start/init script
```
/opt/splunkforwarder/bin/splunk enable boot-start
systemctl start splunk
```
### - Enable Receiving input on the Index Server
```
Configure the Splunk Index Server to receive data, either in the manager: Manager -> sending and receiving -> configure receiving -> new or via the CLI:
/opt/splunk/bin/splunk enable listen 9997
Where 9997 (default) is the receiving port for Splunk Forwarder connections
```
### - Configure Forwarder connection to Index Server:
```
/opt/splunkforwarder/bin/splunk add forward-server hostname.domain:9997

(where hostname.domain is the fully qualified address or IP of the index server (like indexer.splunk.com), and 9997 is the receiving port you create on the Indexer:
Manager -> sending and receiving -> configure receiving -> new). In our case hostname is **Splunkvm/137.135.25.217**
```
### - Test Forwarder connection:
```
/opt/splunkforwarder/bin/splunk list forward-server
```
### - Add Data:
```
/opt/splunkforwarder/bin/splunk add monitor /path/to/app/logs/ -index main -sourcetype %app%
```
Where /path/to/app/logs/ is the path to application logs on the host that you want to bring into Splunk, and %app% is the name you want to associate with that type of data

This will create a file: inputs.conf in /opt/splunkforwarder/etc/apps/search/local/ -- here is some documentation on inputs.conf:

[Splunk Inputs.conf](http://docs.splunk.com/Documentation/Splunk/latest/admin/Inputsconf)

Note: System logs in /var/log/ are covered in the configuration part of Step 7. If you have application logs in /var/log/*/
In our case,
/opt/splunkforwarder/bin/splunk add monitor /path/to/app/logs/

#### - Install and Configure UNIX app on Indexer and *nix forwarders:
```
On the Splunk Server, go to Apps -> Manage Apps -> Find more Apps Online -> Search for ‘Splunk App for Unix and Linux’ -> Install the "Splunk App for Unix and Linux'

Restart Splunk if prompted, Open UNIX app -> Configure

Once you’ve configured the UNIX app on the server, you'll want to install the related Add-on: "Splunk Add-on for Unix and Linux" on the Universal Forwarder. Go to http://apps.splunk.com/ and find the "Splunk Add-on for Unix and Linux" (Note you want the ADD-ON, not the App - there is a difference!).
Copy the contents of the Add-On zip file to the Universal Forwarder, in: /opt/splunkforwarder/etc/apps/. If done correctly, you will have the directory "/opt/splunkforwarder/etc/apps/Splunk_TA_nix" and inside it will be a few directories along with a README & license files.

Restart the Splunk forwarder (/opt/splunkforwarder/bin/splunk restart)
Note: The data collected by the unix app is by default placed into a separate index called ‘os’ so it will not be searchable within splunk unless you either go through the UNIX app, or include the following in your search query: “index=os” or “index=os OR index=main” (don’t paste doublequotes)
```
#### - Customize UNIX app configuration on forwarders:
```
Look at inputs.conf in /opt/splunkforwarder/etc/apps/unix/local/ and /opt/splunkforwarder/etc/apps/unix/default/

The ~default/inputs. path shows what the app can do, but everything is disabled. The ~local/inputs.conf shows what has been enabled – if you want to change polling intervals or disable certain scripts, make the changes in ~local/inputs.conf.
```
#### - Configure File System Change Monitoring (for configuration files):
```
http://docs.splunk.com/Documentation/Splunk/4.3.2/Data/Monitorchangestoyourfilesystem

Note that Splunk also has a centralized configuration management server called Deployment Server. This can be used to define server classes and push out specific apps and configurations to those classes. So you may want to have your production servers class have the unix app configured to execute those scripts listed in ~local/inputs at the default values, but maybe your QA servers only need a few of the full stack, and at longer polling intervals. Using Deployment Server, you can configure these classes, configure the app once centrally, and push the appropriate app/configuration to the right systems.
```

> For more details, please check [here](https://community.splunk.com/t5/All-Apps-and-Add-ons/How-do-I-configure-a-Splunk-Forwarder-on-Linux/m-p/72078)
