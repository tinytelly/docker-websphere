## docker-websphere
================

### Install websphere in Docker
These instructions assume you are installing Websphere 8.5.0 but will work for any version.

#### Before you start:
You will need to get the zip files from IBM for websphere at fix central : http://www.ibm.com/support/fixcentral/

Download the installation files to /tmp/websphere


#### Start docker 
 ```
 docker run -i -t -v /tmp/websphere:/tmpfromhost centos bash 
 ```

#### Run up websphere inside docker
 ```
yum -y install unzip
yum -y install gtk2.i686
yum -y install libXtst.i686

mkdir /opt/software/
mkdir /opt/software/IBM
mkdir /opt/software/IBM/installer-package
mkdir /opt/software/IBM/was
mkdir /opt/WebSphere85/

cd /tmpfromhost
unzip InstalMgr1.5.2_LNX_X86_WAS_8.5.zip -d /opt/software/IBM/installer-package
unzip   WAS_V8.5_1_OF_3.zip -d /opt/software/IBM/was
unzip   WAS_V8.5_2_OF_3.zip -d /opt/software/IBM/was
unzip   WAS_V8.5_3_OF_3.zip -d /opt/software/IBM/was

#Install Package Manager
cd /opt/software/IBM/installer-package
./installc  -acceptLicense -accessRights admin -installationDirectory "/opt/WebSphere85/IMGR" -dataLocation "/opt/WebSphere85/Imdata" -silent

#Install WAS
cd /opt/WebSphere85/IMGR/eclipse/tools
./imcl listAvailablePackages -repositories /opt/software/IBM/was/repository.config
[get the output and use that string in the next install command e.g. com.ibm.websphere.BASE.v85_8.5.0.20120501_1108]
./imcl -acceptLicense -showProgress install com.ibm.websphere.BASE.v85_8.5.0.20120501_1108 -repositories  /opt/software/IBM/was/repository.config

#create a default profile
cd /opt/IBM/WebSphere/AppServer/bin
./manageprofiles.sh -create -templatePath /opt/IBM/WebSphere/AppServer/profileTemplates/default/
 ```
 
#### Build the docker image
```
docker ps -a
```

get the containerId and use it in the next command

```
docker commit <containerId> websphere_8_5_0
```

#### Start the image and expose the port (interactive mode: meaning you end up inside the container)

``` 
docker run -i -t -p 9080:9080 -v /tmp/websphere:/tmpfromhost websphere_8_5_0 bash -c 'cd tmpfromhost; chmod 777 *.*; ./startWebsphere.sh;'
```

#### Start the image and expose the port (detached mode: meaning its running in the background)
```
docker run -d -p 9080:9080 -v /tmp/websphere:/tmpfromhost websphere_8_5_0 bash -c 'cd tmpfromhost; chmod 777 *.*; ./startWebsphere.sh;'
```

##### contents of startWebsphere.sh
```
cd /opt/IBM/WebSphere/AppServer/bin
./startServer.sh server1

while ( true )
    do
    echo "Detach with Ctrl-p Ctrl-q. Dropping to shell"
    sleep 1
    /bin/bash
done
```

#### Export the docker image to a tar ( to move it from server to server without using a shared repository)
```
docker save websphere_8_5_0 > websphere_8_5_0.tar
```
#### Import the tar as an image on a different server after you have copied over the tar file.
```
docker load -i websphere_8_5_0.tar
```

#### Test the server is responding (via lynx here or use a browser)
```
yum -y install lynx
lynx localhost:9080/snoop
```
