
## How to compile and run.
1. First you need to install JAVA on the server.
```bash
 sudo apt-get update
 sudo apt-get install default-jdk
```
Check Java version:
```bash
 java -version
```
2. The Tomcat or Jetty servlet container. We will install Tomcat-9. Go to the folder opt and download the archive Tomcat-9:
```bash
 sudo useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
```
   
```bash
wget -c https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.tar.gz
```
We unpack:
```bash
sudo tar xf apache-tomcat-9.0.86.tar.gz -C /opt/tomcat
```
For convenience, change the name of the unpacked folder to tomcat using the following command:
```bash
sudo ln -s /opt/tomcat/apache-tomcat-9.0.86 /opt/tomcat/updated
```
```bash
sudo chown -R tomcat: /opt/tomcat/*
```
```bash
sudo sh -c 'chmod +x /opt/tomcat/updated/bin/*.sh'
```
Now we need to add Tomcat to the services so that it can be easily started and stopped.
Create a new file ##tomcat.service
```bash
 sudo nano /etc/systemd/system/tomcat.service
```
Add the following lines to it:
```bash
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment="JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64"
Environment="CATALINA_PID=/opt/tomcat/updated/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat/updated/"
Environment="CATALINA_BASE=/opt/tomcat/updated/"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
Environment="JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom"

ExecStart=/opt/tomcat/updated/bin/startup.sh
ExecStop=/opt/tomcat/updated/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
Save the file and restart the service manager:
```bash
 sudo systemctl daemon-reload
```
To add the Tomcat service to startup:
```bash
 sudo systemctl enable tomcat
```
We launch Tomcat through the service and check its status:
```bash
 sudo service tomcat start
 service tomcat status

Loaded: loaded (/etc/systemd/system/tomcat.service: disabled; vendor preset: enabled)
Active: active (running)...........
.....................
```
By default, Tomcat runs on port 8080. If desired, you can change it. Next, create users.
Open the file /opt/tomcat/conf/tomcat-users.xml:
```bash
  nano /opt/tomcat/apache-tomcat-9.0.86/conf/tomcat-users.xml
```
Create a new user named admin and the role of admin, manager, and manager-gui. This file must be protected, so you will need to open it as root, and add the entry like this:
```bash
<role rolename="tomcat"/>                                                    
<role rolename="admin"/>                                                     
<role rolename="manager"/>                                                   
<role rolename="manager-gui"/>
<user name="admin" password="XXXXXX" roles="tomcat,admin,manager,manager-gui"/>
```
Save and close.

To connect to the service from outside the comment lines in the files /opt/tomcat/webapps/manager/META-INF/context.xml and /opt/tomcat/webapps/host-manager/META-INF/context.xml:
```bash
 nano /opt/tomcat/updated/webapps/manager/META-INF/context.xml
```
```bash
. . . . . . 
<Context antiResourceLocking="false" privileged="true" >
 <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer$
</Context>
. . . . . .
```
```bash
 nano /opt/tomcat/updated/webapps/host-manager/META-INF/context.xml
```
```bash
. . . . . .
<Context antiResourceLocking="false" privileged="true" >
 <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer$
</Context>
. . . . . . 
```
To increase the amount of uploaded files, edit the file /opt/tomcat/webapps/manager/WEB-INF/web.xml:
```bash
 nano /opt/tomcat/updated/webapps/manager/WEB-INF/web.xml
```
```bash
. . . . .
<multipart-config>
      <!-- 80MB max -->
      <max-file-size>82428800</max-file-size>
      <max-request-size>82428800</max-request-size>
. . . . . .
```
And run Tomcat:
```bash
 sudo service tomcat start
```
<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-07-Apache-Tomcat-9-0-10-768x491.png" alt="">

3. Install Maven to build AidakoMed:

```bash
 sudo apt-get install maven
```
4. Install git:
```bash
 sudo apt-get install git
```
. Clone the aidakomed repository and compile:

```bash
 sudo git clone https://github.com/Aidako20/aidakomed.git
 cd aidakomed
 sudo mvn clean package
```
The compiled openmrs.war file will be located in the /aidakomed/webapp/target/folder and copy it to the /opt/tomcat/webapps/ folder and restart Tomcat:

```bash
 cd /aidakomed/webapp/target
 sudo cp openmrs.war /opt/tomcat/webapps/
 service tomcat restart
```
You can also download the war file via the web interface of tomcat:

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-07-manager-768x491.png" alt="">

6. Install Mysql:

```bash
 sudo apt-get install mysql-server mysql-client
```
```bash
 sudo mysql
```

mysql>
```bash
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

```bash
  exit
```
```bash
sudo mysql_secure_installation
```

In the query field, create a password for the root user and remember.

For PostgreSQL:
```bash
  sudo apt update
  sudo apt install postgresql postgresql-contrib
```
Type the following command to change the default password for the Postgres user:
```bash
  sudo -i -u postgres
  psql
```
```bash
  ALTER USER postgres WITH PASSWORD 'new_password';
  \q
  exit
```
Then in the browser at http://localhost:8080/openmrs the installer will prompt you for the next steps.

Starting the installer:

For PostgreSQL webinterface:
Set the database connection string as: 
```bash
jdbc:postgresql://localhost:5432/@DBNAME@?charSet=UNICODE
```
Set the database driver as: 
```bash
org.postgresql.Driver
```
