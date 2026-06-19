# openproject-bim-nextcloud-eurooffice-opnsense
This repository demonstrates a self-hosted BIM collaboration and construction project management platform built using OpenProject BIM Edition, Nextcloud, EuroOffice and OPNsense HAProxy, enabling construction and engineering teams to plan, execute, review, and document projects while managing schedules, documents, and IFC/BIM models from a centralized environment with full ownership of project data.

The platform combines: 
OpenProject BIM for project planning and execution
Nextcloud for centralized document management
EuroOffice for online document editing
OPNsense HAProxy for secure external access & HAProxy
Docker Compose for application deployment and management

## Architecture
```text
Users
  ↓
OPNsense + HAProxy + SSL
  ↓
OpenProject BIM  ←→  Nextcloud+EuroOffice
  ↓                    ↓
Project Mgmt         IFC/Drawing Storage/Documents
  ↓                    ↓
Tasks/Gantt          DWG/PDF/IFC Files
  ↓
BCF + IFC Viewer
```
## Deployent Guide
1. Deploy Nextcloud
   
   To deploy Nextcloud on VM, create the directory structure and adjust the file ownership and permissions as follows:
```
mkdir -p /path/to/nextcloud/config/nextcloud
mkdir -p /path/to/nextcloud/data/{mariadb,redis,nextcloud-html,nextcloud-data}
chown -R www-data:www-data /path/to/nextcloud
find /path/to/nextcloud/ -type d -exec chmod 750 {} \;
find /path/to/nextcloud/ -type f -exec chmod 640 {} \;
```
change example.env to .env and change values. Also copy apache.conf, php.ini and procede with 
```
docker compose up -d
```
Configure HAProxy backend, frontend, SSL certificate, and domain mapping to publish the application and access it using: https://nextcloud.example.com

2. Deploy Euro-Office

   To deploy Euro-Office, create the directory structure and generate JWT secret and deploy
   
   Configure HAProxy backend, frontend, SSL certificate, and domain mapping to publish the application and access it using: https://eurooffice.example.com
   Make sure put following header in OPNsense HAproxy setting Option pass-through: 
```
# This tells eurooffice is running behind haproxy https
http-request set-header X-Forwarded-Proto https
http-request set-header X-Forwarded-Host eurooffice.example.com
http-request set-header X-Forwarded-For %[src]
```
2a. Nextcloud and Euro-Office Intigration
 
   To intigrate Nextcloud with Euro-Office
```
Go to nextcloud >> Admin >> Apps >> search onlyoffice >> Download and enable
Next Administration settings >> ONLYOFFICE
ONLYOFFICE Docs address: https://eurooffice.example.com/
Secret Key: Put JWT secret key used in Euro-Office
ONLYOFFICE Docs: http://ip:euroofficeport/
Server address: http://ip:nextcloudport/ 
```
2b. Create a document from Nextcloud and confirm browser-based editing works.

3. Deploy OpenProject BIM
   To deploy OpenProjectBIM, create the directory structure and adjust the file ownership and permissions as follows:
```
mkdir -p /path/to/openproject/assets
mkdir -p /path/to/openproject/postgresql
chown -R 1000:1000 /path/to/openproject/assets
chown -R 999:999 /path/to/openproject/postgresql
```
Changing example.env to .env and update and Run docker compose up -d
Configure HAProxy backend, frontend, SSL certificate, and domain mapping to publish the application and access it using: https://openprojectbim.example.com
Make sure put following header in OPNsense HAproxy setting Option pass-through:  
```
http-request set-header X-Forwarded-Proto https
http-request set-header X-Forwarded-Port 443
http-request set-header X-Forwarded-Host openprojectbim.example.com
http-request set-header Host openprojectbim.example.com
```
3a. Nextcloud and OpenProjectBIM Intigration
  To intigrate Nextcloud with OpenProjectBIM
```
Go to nextcloud >> Admin >> Apps >> search OpenProject >> Download and enable
Next Administration settings >> OpenProject >>
1 OpenProject server: https://openprojectbim.example.com

```
