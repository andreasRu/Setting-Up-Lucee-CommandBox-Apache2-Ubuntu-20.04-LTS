# Setting-Up-Lucee-CommandBox-Apache2-Ubuntu-20.04-LTS
A simple step by step guide about installing Lucee with CommandBox behind Apache2 with AJP. The setup is made from a remote computer with Windows 10. The first two steps are equal to the first two videos I've added to [Lucee documentation here](https://docs.lucee.org/guides/installing-lucee/installation-linux/linux-ubuntu-quick-video-guide.html). The purpose of this step by step guide in this repository is for my own documentation, but also for others to experiment and play arround. As soon as this information gets more solid proof, I'll PR it to the Lucee Documentation. Please enjoy!

### Step 1: Adding Required Ubuntu Packages
Please follow the steps I've shown in the following video:
<div>
<a href="http://www.youtube.com/watch?feature=player_embedded&v=Hk9mbHWFGvQ
" target="_blank"><img src="http://img.youtube.com/vi/Hk9mbHWFGvQ/0.jpg" 
alt="Adding Required Ubuntu Packages Video" width="240" height="180" border="10" /></a></div>

### Step 2 - Setting Up SSH X11 Forwarding
Please follow the steps I've shown in the following video:
<div>
<a href="http://www.youtube.com/watch?feature=player_embedded&v=mUsaqdLmWAc
" target="_blank"><img src="http://img.youtube.com/vi/mUsaqdLmWAc/0.jpg" 
alt="Setting Up SSH X11 Forwarding Video" width="240" height="180" border="10" /></a></div>

### Step 3 - Installing Java Development Kit (JDK) 
CommandBox needs Java Development Kit (JDK) to run. In this example we are going to install Ubuntus default JDK __OpenJDK__. First you need to connect through SSH with X11 forwarding enabled to your server and log into your sudo account. After that you can install Ubuntus default JDK with:
```
$ sudo apt update
$ sudo apt install default-jdk
```

### Step 4 - Installing CommandBox 
Install CommandBox as specified at [Ortus official installation documentation](https://commandbox.ortusbooks.com/setup/installation) with the following commands:
```
$ sudo apt install libappindicator-dev
$ curl -fsSl https://downloads.ortussolutions.com/debs/gpg | sudo apt-key add -
$ echo "deb https://downloads.ortussolutions.com/debs/noarch /" | sudo tee -a /etc/apt/sources.list.d/commandbox.list
$ sudo apt update && sudo apt install apt-transport-https commandbox
```

### Step 5 - Create a non-root/non-sudo user to run CommandBox as a service 
To run CommandBox with a different user we need to create a user and usergroup ( e.g. named "cfbox" ) with no login capabilities: 
```
$ sudo useradd -r -m -U -d /opt/CommandBox -s /bin/false cfbox
```
 
*Explanation of the command '**$ sudo useradd -r -m -U -d /opt/CommandBox -s /bin/false cfbox**':*
* '-r' : creates the group also with the same name as user.
* '-m' : creates the user's home directory if it does not exist.
* '-U' : Creates a group with the same name as the user, and adds the user to this group.
* '-d /opt/CommandBox' : specifies the home directory for new user.
* '-s /bin/false' : specifies the user's default shell. The value *'bin/false'* locks the default shell, so there is no log in for the user *'cfbox'* available.
* '-c "user display information"' : use this to optionally to add a text display informations to the users account

### Step 6 - Create a wwwroot to hold your web applications files and add a index.cfm file to it for testing 
Create a folder *'wwwroot'* that will hold your web application cfm files and resources. To do that, open the file explorer *'thunar'* with sudo with: 
```
$ sudo thunar
```
 1. Then navigate to */var/www/* and
 2. Create a folder named **wwwroot** and
 3. Add a *'/var/www/wwwroot/__index.cfm__'* file with the following simple content for testing:
```html
<!-- file located at /var/www/wwwroot/index.cfm -->
<cfexecute name="whoami" variable="cfexecOutput"></cfexecute>
<cfdump var="#[cfexecOutput,now(),cgi]#">
```
### Step 7 - Add a CommandBox server.json to configure the server to serve your web application
CommandBox uses a *'server.json'* file to configure specific server settings to be used for the web application to run with its embedded server 'Undertow'. Some settings need to be disabled, otherwise the service start wil fail. If you want to see a full description of the possible settings of the server.json file, visit the [Ortus Solution - server.json settings documentation](https://commandbox.ortusbooks.com/embedded-server/server.json). For now we are going to create the server.json file at *'/var/www/wwwroot/__server.json__'* with the following content:
```json
{
  
  "name": "myapp",
  "openBrowser": false,
  "trayEnable": false,

  "web": {
      
      "HTTP": {
          "enable": true,
          "port": 8080
      },
      "AJP": {
          "enable": true,
          "port": 8009
      },

  },
   "app": {
        "cfengine": "lucee@5.x",
        "serverHomeDirectory": "/opt/CommandBox/web-contexts/wwwroot-myapp"
  }	

}
```
*Explanation of what the file '/var/www/wwwroot/__server.json__' shown above does:*
 * specifies the web applications name "myapp"
 * prevents CommandBox from launching a Browser
 * prevents CommandBox from launching the IconTray
 * enables HTTP on port 8080 for simple test browsing on http://127.0.0.1:8080
 * enables AJP on port 8009 for connecting Apache2
 * uses Lucee's latest version as the running cfengine of the app
 * deploys Undertow's server context for the app at "/opt/CommandBox/web-contexts/wwwroot-myapp".
 There is much more that can be configured with the server.json file. Please find further information about configuring your CommandBox inbuilt server 'Undertow' at [Ortus - server.json](https://commandbox.ortusbooks.com/embedded-server/server.json)

### Step 8 - Give CommandBox read/write permissions to the server.json file
Because CommandBox opens and rewrites the server.json file, the user 'cfbox' needs read/write priviliges for it. To adjust it navigate to the file with the file explorer 'thunar' -&gt; right click the server.json file -&gt; 'permissisions' &gt; group &gt; __'cfbox'__ with __read/write__ permissions.

### Step 9 - Run CommandBox for the first time as the user 'cfbox' for initialization
At first start CommandBox installs/initializes itself and deploys all necessary packages and dependencies it needs to run. To firstly run CommandBox as the user 'cfbox' all together with the 'server start' command to launch the web application, enter the line as follows:
```
$ sudo -H -u cfbox box server start /var/www/wwwroot/server.json --console
```
*Explanation of the command **$ sudo -H -u cfbox box server start /var/www/wwwroot/server.json --console**':*
* '-H' : runs the command by using the target user's HOME environment variables. That will cause CommandBox to use the folder '/opt/CommandBox' as specified in 'Step 5 - Create a non-root/non-sudo user to run CommandBox as a service' and create CommandBox lib/deploy directory to '/opt/CommandBox'__.CommandBox/__'. 
* '-u cfbox' : runs the command as the user 'cfbox'.
* 'box server start /var/www/wwwroot/server.json --console' : run a CommandBox server instance with the settings of the server.json and output the data to console

As soon as you see the text in the console window "[INFO ] Runwar: Server is up - http-port:8080..." your server is up and running. After that you can quit the server with 
the Linux quit shortcut:
```
ctrl+c
```

If you are running CommandBox and you have access to the interactive CommandBox shell, simply enter the following to quit:
```
$ quit
```

__Addition information__: If in certain sitations you need to run CommandBox as the user 'cfbox' to access CommandBox shell (e.g. for debugging the service start), launch the CommandBox session with the following line:
```
$ sudo -H -u cfbox box
```

### Step 10 - Install CommandBox as a service to run as 'cfbox' 
To install a service (e.g. named '__MyserviceName__') in Ubuntu 20.04 LTS, a corresponding file with the extension *'.service'* needs to be created and saved to '/usr/lib/systemd/system/' (e.g. *'/usr/lib/systemd/system/__MyserviceName__.service'*). To run CommandBox as service we will create a service named *__'commandbox-myapp'__* and create the file */usr/lib/systemd/system/__commandbox-myapp.service__*:
1. Launch the file explorer 'thunar' with: 
```
$ sudo thunar
```
2. Then navigate to *'/usr/lib/systemd/system/'* and create a file named *'__commandbox-myapp.service__'* with the following content:
```
#Systemd unit file for CommandBox Site
[Unit]
Description=commandbox-myapp

[Service]
Type=forking

ExecStart=/usr/local/bin/box server start /var/www/wwwroot/server.json
ExecStop=/usr/bin/kill -15 $MAINPID

User=cfbox
Group=cfbox
UMask=0007
RestartSec=50
Restart=always

[Install]
WantedBy=multi-user.target
```

   3. Enable the service with:
   ```
   $ sudo systemctl enable commandbox-myapp.service
   ```

   4. Start the service with:
   ```
   $ sudo systemctl start commandbox-myapp.service
   ```

   5. Wait for CommandBox to deploy all server context dependencies and check the service status by entering:
   ```
   $ sudo systemctl status commandbox-myapp.service
   ```
   
### Step 11 - Open a browser and test the web application
Check if the CommandBox inbuilt server "Undertow" is serving the page correctly at: http://127.0.0.1:8080/index.cfm by entering firefox without sudo:
 ```
   $ firefox http://127.0.0.1:8080/index.cfm
 ```
If you have successfully tested the page and you are seeing the index.cfm page displayed with the data dump, we can proceed to Step 12 to connect CommandBox inbuilt servlet container enging 'Undertow' with the front end web server Apache2.

### Step 12 - Connect Apache2 with Commandbox inbuilt servler container engine "Undertow" with AJP
We have already enabled AJP in the server.json file. Still, Apache2 needs to be configured to intercept '.cfm/.cfc' files and forward the connection (also called to 'reverse proxy') to CommandsBox 'servlet container engine'. This is done with the Apache2 module mod_proxy_ajp (and mod_proxy). To configure Apache2 to for reverse proxy as AJP, you need to:

1. Enable the module by entering:
   ```
   $ sudo a2enmod proxy_ajp
   ```
2. Open 'thunar' and edit the file /etc/apache2/___apache2.conf__ and add at the end of the file:
  ```
  ...
  
 <IfModule mod_proxy.c>
	ProxyPreserveHost On
	# HTTP ProxyPassReverse
	#ProxyPassMatch ^/(.+\.cf[cm])(/.*)?$ http://127.0.0.1:8888/$1$2
	#ProxyPassMatch ^/(.+\.cfml)(/.*)?$ http://127.0.0.1:8888/$1$2
	
	# AJP  ProxyPassReverse
	ProxyPassMatch ^/(.+\.cf[cm])(/.*)?$ ajp://127.0.0.1:8009/$1$2
	ProxyPassMatch ^/(.+\.cfml)(/.*)?$ ajp://127.0.0.1:8009/$1$2
	
	# optional mappings
	#ProxyPassMatch ^/flex2gateway/(.*)$ http://127.0.0.1:8888/flex2gateway/$1
	#ProxyPassMatch ^/messagebroker/(.*)$ http://127.0.0.1:8888/messagebroker/$1
	#ProxyPassMatch ^/flashservices/gateway(.*)$ http://127.0.0.1:8888/flashservices/gateway$1
	#ProxyPassMatch ^/openamf/gateway/(.*)$ http://127.0.0.1:8888/openamf/gateway/$1
	#ProxyPassMatch ^/rest/(.*)$ http://127.0.0.1:8888/rest/$1
 
 # optional mappings as AJP
	#ProxyPassMatch ^/flex2gateway/(.*)$ ajp://127.0.0.1:8009/flex2gateway/$1
	#ProxyPassMatch ^/messagebroker/(.*)$ ajp://127.0.0.1:8009/messagebroker/$1
	#ProxyPassMatch ^/flashservices/gateway(.*)$ ajp://127.0.0.1:8009/flashservices/gateway$1
	#ProxyPassMatch ^/openamf/gateway/(.*)$ ajp://127.0.0.1:8009/openamf/gateway/$1
	#ProxyPassMatch ^/rest/(.*)$ ajp://127.0.0.1:8009/rest/$1
	
	# HTTP ProxyPassReverse
	#ProxyPassReverse / http://127.0.0.1:8888/

	# AJP	ProxyPassReverse
	ProxyPassReverse / ajp://127.0.0.1:8009/
</IfModule>

...
 ```
 
3. Adapt the virtual host configuration file '/etc/apache2/sites-available/000-default.conf' by changing your document root to your webroot at '/var/www/wwwroot' and add the *'DirectoryIndex'* directive with the value '__index.cfm__':  

 ```
<VirtualHost *:80>
...

	DocumentRoot /var/www/wwwroot
	DirectoryIndex index.cfm

...
</VirtualHost>
 ```

4. After that you only need to restart apache2 with:
```
$ sudo systemctl restart apache2
```
