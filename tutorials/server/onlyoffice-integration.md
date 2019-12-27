---
id: onlyoffice-for-integration
summary: This guide will show you three alternative ways to install ONLYOFFICE Document Server and all the dependencies it needs for further integration, on Ubuntu
categories: server
tags: onlyoffice, integration, online editors, docu;ent server
difficulty: 2
status: published
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published: 2020-12-27
author: Maria Pashkina <maria.pashkina@onlyoffice.com>
---
# How to install ONLYOFFICE for integration on Ubuntu
## 1. INTRODUCTION
Duration: 2:00

[ONLYOFFICE](http://onlyoffice.com/)  is an open-source collaborative office suite. It contains three editors for text documents, spreadsheets, and presentations, fully compatible with Office Open XML formats and enabling collaborative editing in real time. It can be integrated with: ONLYOFFICE collaboration platform that includes CRM, projects, document management system, mail, calendar, blogs, forums and chat (to learn more about its modules, refer to [this tutorial](https://tutorials.ubuntu.com/tutorial/install-onlyoffice-on-ubuntu1804#0) OR third-party web services, like Nextcloud, ownCloud, Alfresco, Confluence, SharePoint, Liferay, HumHub, or your own web app. 

This last solution allows you to install ONLYOFFICE Document Server on your local server and integrate it with a whole variety of sharing services via ONLYOFFICE ready-to-use connectors or third-party integration apps.


ONLYOFFICE Document Server features the following:
* Compatibility with MS Office and OpenDocument formats : .docx, .xlsx, .pptx, .odt, .ods, .odp, etc.
* Abundance of editing and styling features: hyperlinks, tables and charts, pictures, autoshapes, formulas or text objects, a bulleted or numbered list, and more.
* Powerful collaborating capabilities: two co-editing modes (fast and strict), commenting, reviewing, tracking changes, integrated chat.
* Document sharing permissions: full access, read only, review, comment, form filling, download and modify filter.
* Extending functionality with existing plugins or building your own add-ons using the API.

ONLYOFFICE comes with a dual-license model. This means that as long as you respect the GNU AGPL v.3 license, you can use the ONLYOFFICE open source solution available on [GitHub](https://github.com/ONLYOFFICE/DocumentServer). For bigger teams, using ONLYOFFICE as a part of their SaaS or on-premise service, a commercial license is required. 

### WHAT YOU'LL LEARN
This guide will show you three alternative ways to install ONLYOFFICE Document Server and all the dependencies it needs for further integration, on Ubuntu:
* using deb. package
* using Docker image
* from snap package

### WHAT YOU'LL NEED
* System requirements:
CPU: dual core 2 GHz or better
RAM: 2 GB or more
HDD: at least 40 GB of free space
* Software requirements: Ubuntu 18.04
* Docker: version 1.10 or later
* At least 4 GB of swap
---
## 2. INSTALLATION
Duration: 20:00

### 2.1 Install ONLYOFFICE Document Server on Ubuntu using .deb package
For ONLYOFFICE Document Server correct work your machine should meet the requirements above and also have some additional components installed:
* mono: version 4.0.0 or later
* MySQL: version 5.6.4 or later
* nginx: version 1.3.13 or later
* nodejs: version 0.10 or later
* libstdc++6: version 4.9 or later

Open Terminal using Ctrl+Alt+T

Download ONLYOFFICE GPG signing key:
```sudo wget http://download.onlyoffice.com/repo/onlyoffice.key```
And add it to the system:
```sudo apt-key add onlyoffice.key```
Add ONLYOFFICE repository to the list stored in the /etc/apt/sources.list file.To do that, open the /etc/apt/sources.list file using any available text editor (e.g. nano):
```sudo nano /etc/apt/sources.list```
Add the following record:
```deb http://download.onlyoffice.com/repo/debian squeeze main```
Update the package cache:
```sudo apt-get update```
Execute the following command to install online editors:
```sudo apt-get install onlyoffice-documentserver```

### 2.2 Install ONLYOFFICE Document Server on Ubuntu using the official Docker Image
Install the latest Docker version:
```sudo apt-get install docker-ce```
Execute the following command to install Document Server and all the dependencies:
```sudo docker run -i -t -d -p 80:80 onlyoffice/documentserver```

### 2.3 Install ONLYOFFICE Document Server on Ubuntu from snap package
To install ONLYOFFICE Document Server from the command line, just run the following command:
```sudo snap install onlyoffice-ds```
---
## 3.INSTALLATION COMPLETE
Duration: 02:00
Congratulations! ONLYOFFICE Document Server has been successfully installed. You can easily run it using secure connection at http://localhost.
![](https://serenity-networks.com/wp-content/uploads/2017/03/Selection_023.png)

Now, ONLYOFFICE can be easily integrated into your sync&share solution or a web application via [the integration apps](https://www.onlyoffice.com/en/all-connectors.aspx). Bring to your users online editors and collaborate on text docs, spreadsheets and presentations within the interface of you favorite platform. For example: 
![](https://www.onlyoffice.com/blog/wp-content/uploads/2019/09/onlyoffice_nextcloud_connector_watermark.png)







