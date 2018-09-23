# CognitiveFactory dev environment

## VisualStudio 2017, ASP.Net Core 2.2 and Blazor

1. Download [Visual Studio Community 2017](https://visualstudio.microsoft.com/fr/downloads/) [v15.8.5]

2. Install the ".Net Core cross-platform development" and the "ASP.Net and web development" workloads

3. Download and install the following Visual Studio Extensions
- [Github Extensions](https://visualstudio.github.com/) [v2.5.5.3913]
- [Markdown Editor](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.MarkdownEditor) [v1.12.236]

4. Install [.Net Core SDK 2.2](https://www.microsoft.com/net/download/dotnet-core/2.2) [v2.2.100-preview2]

5. Install the [ASP.Net Core Blazor Language Services](https://marketplace.visualstudio.com/items?itemName=aspnet.blazor) [v15.7.10384] for Blazor v0.5.1 

## OKD (Openshift origin) - minishift

0. Enable Virtualization (VT-X/AMD-v instructions) in the BIOS/UEFI setup of the machine
1. Download and install [VirtualBox for Windows hosts](https://www.virtualbox.org/wiki/Downloads) [v5.2.18]
2. Download [OKD Minishift](https://github.com/minishift/minishift/releases) [v1.24.0]
3. Read installation instructions from
- https://docs.okd.io/latest/minishift/getting-started/preparing-to-install.html
- https://www.middleware-solutions.fr/optimiser-minishift-sur-windows-10/
4. Extract zip in **C:\CognitiveFactory\minishift** (note : drive c:\ is required, don't choose any other path)

Add the "minishift" executable location to Windows PATH environment variable, for example C:\CognitiveFactory\minishift.

5. Open a command prompt (cmd)

- minishift config set vm-driver virtualbox
- minishift config set cpus 2
- minishift config set memory 2048
- minishift config set disk-size 10g
- minishift start

... creates virtual machine, downloads and installs [minishift-centos7.iso](https://github.com/minishift/minishift-centos-iso/releases/download/v1.12.0/minishift-centos7.iso) [v1.12.0], downloads and starts openshift/origin [v3.10.0] cluster ...

Note : in case of timeout error while pulling openshift container image, try again : minishift stop / minishift start.

Take note of the URL provided in the following message, which should appear on screen at the end of the startup process :

*The server is accessible via web console at:
    https://192.168.99.100:8443*

- minishift oc-env

Add the "oc" executable location to Windows PATH environment variable, for example C:\Users\*username*\.minishift\cache\oc\v3.10.0\windows
Close and reopen a command promt (cmd)

- oc login -u system:admin

6. Launch Notepad as an administrator.

- Open the Windows hosts file in the folder : C:\Windows\System32\drivers\etc.
- add one line (with the IP noted during the preceding step) and save : 192.168.99.100 [TAB] openshift.local       
- open Chrome web browser, point it to the URL : [https://openshift.local:8443](https://openshift.local:8443) 
- allow : unsafe connexion - continue to the unsafe website.
- log in as : admin/admin or developer/developer.

7. Later, to stop / delete / update your minishift instance :
- minishift stop
- minishift delete
- rm -rf ~/.minishift
- rm -rf ~/.kube
- minishift update

## MySQL database on Openshift

1. Login to the OKD web console by pointing Chrome at URL https://192.168.99.100:8443 and using developer/developer credentials
2. Click on "My Project" / Delete project on the right of the screen
3. Create a new project : "cognitivefactory", "Cognitive Factory", "Cognitive Factory platform"
3. Select MySQL in the services catalog, keep the default config parameters, except for
- MYSQL connection username/password : mysqluser/mysqluserpassword
- MYSQL root password : mysqlrootpassword
- MYSQL database name : cognitivefactorydb
4. Note the following parameters

Username: mysqluser
Password: mysqluserpassword
Database Name: cognitivefactorydb
Connection URL: mysql://mysql:3306/

5. Navigate to the "Cognitive Factory" project, in the Overview tab, open the details of the deployment config "mysql, #1" by clicking on >
- in the NETWORKING section, click on the "mysql" service
- actions => edit YAML
- replace "type: ClusterIP" with "type: NodePort"
- Save
- Just below the Traffic header, note the Route/NodePort number, for example : 32396

6. Install [MySQL for Visual Studio](https://dev.mysql.com/downloads/installer/) [v8.0.12]
- download and execute mysql-installer-web-community-8.0.12.0.msi
- select Custom setup type
- select "MySQL for Visual Studio 1.2.8" to be installed

7. Connect Visual Studio to MySQL running inside OpenShift
- Open Visual Studio
- View menu, display Server Explorer, right-click on Data connections, Add data connection
- select data source : MySQL Database
- server name : openshift.local
- user name : mysqluser
- user password : mysqluserpassword
- save my password
- Advanced => port : 32396 (port of the mysql service above)
- Test connection
- select database name : cognitivefactorydb
- OK