
## How to compile and run.
1. First you need to install JAVA on the server.
```bash
 sudo apt-get update
 sudo apt-get install defaul-jdk
```
Check Java version:
```bash
 java -version
```
2. The Tomcat or Jetty servlet container. We will install Tomcat-9. Go to the folder opt and download the archive Tomcat-9:
```bash
 cd /opt
 sudo wget http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.12/bin/apache-tomcat-9.0.12.tar.gz
```
We unpack:
```bash
 sudo tar -zxf apache-tomcat-9.0.12.tar.gz
```
For convenience, change the name of the unpacked folder to tomcat using the following command:
```bash
 sudo mv apache-tomcat-9.0.12 tomcat
```
Now we need to add Tomcat to the services so that it can be easily started and stopped.
Create a new file ##tomcat.service
```bash
 sudo nano /etc/systemd/system/tomcat.service
```
Add the following lines to it:
```bash
[Unit]
Description=Tomcat9
After=network.target

[Service]
Type=forking

Environment=CATALINA_PID=/opt/tomcat/tomcat9.pid
Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment="CATALINA_OPTS=-Xms512m -Xmx512m"
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8 -Dnet.sf.ehcache.skipUpdateCheck=true -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC"
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

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
 sudo nano /opt/tomcat/conf/tomcat-users.xml
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
 sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
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
 sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
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
 sudo git clone https://github.com/AidakoMed/aidakomed.git
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
In the query field, create a password for the root user and remember.

Then in the browser at http://localhost:8080/openmrs the installer will prompt you for the next steps.

Starting the installer:

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot-768x491.png" alt="">

Language selection.

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot1-768x491.png" alt="">

Select the version of the installation.

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot2-768x491.png" alt="">

Create or select a database.

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot3-768x491.png" alt="">

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot7-768x491.png" alt="">

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot8-768x491.png" alt="">

The password for the admin user of the AidakoMED server.

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-Screenshot9-768x491.png" alt="">

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-03-AidakoMed-768x491.png" alt="">

AidakoMED is installed, now we will compile and install the administrative module legacy-ui:

```bash
 sudo wget https://github.com/AidakoMed/aidakomed-module-legacyui/archive/master.zip
 sudo unzip master.zip
 cd aidakomed
 sudo mvn clean package
```
The compiled .omod file will be located in the omod/target folder, copy this .omod file to the directory in the root of the server in the modules folder and restart tomcat:

```bash
 sudo cp legacyui-1.5.0-SNAPSHOT.omod  ~/.OpenMRS/modules
 sudo service tomcat restart
```

You can also compile and load other modules (https://github.com/AidakoMed)

As a result, we will see such a picture at

http://yuor_domain_name_or_IP:8080/openmrs 

<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/08/Screenshot_2018-08-04-%D0%90%D0%B4%D0%BC%D0%B8%D0%BD%D0%B8%D1%81%D1%82%D1%80%D0%B0%D1%86%D0%B8%D1%8F-768x491.png" alt="">

legacy-ui is installed, at the bottom you can change the locale (language) available-English, Russian, Kazakh, French, etc., now you can install modules via the web interface. They can be found on the portal github and gitlab. Also, the modules from OpenMRS, LibreHealth.

Done.

Perhaps you need a ready solution from the box, you can download the AidakoMed-Box distribution to your server from http://aidakomed.info.



<img class="img-responsive" src="http://aidakomed.info/wp-content/uploads/2018/11/aidakomed-mrs.png" alt="">
