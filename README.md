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
#change example.env to .env and change values. Also copy apache.conf, php.ini and procede with 
cd /path/to/nextcloud
chown -R 33:33 /path/to/nextcloud/data/nextcloud-html
chown -R 33:33 /path/to/nextcloud/data/nextcloud-data
docker compose up -d
```
  Configure HAProxy backend, frontend, SSL certificate, and domain mapping to publish the application and access it using: https://nextcloud.example.com
  Access Nextcloud UI and Install nextcloud by providing user and password.
```
# post install
docker exec -u www-data nextcloud_app php occ background:cron
docker exec -u www-data nextcloud_app php occ db:add-missing-indices
docker exec -u www-data nextcloud_app php occ db:convert-filecache-bigint
docker exec -u www-data nextcloud_app php occ config:system:set default_phone_region --value=IN
```
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
Login with default user: admin and passwd: admin then reset strong_password

3a. Nextcloud and OpenProjectBIM Intigration
  
  To intigrate Nextcloud with OpenProjectBIM
```
Go to nextcloud >> Admin >> Apps >> search OpenProject >> Download and enable
Go to OpenProjectBIM >>  Administration >> Files >> External file storages
Name: Filestorage_name
Host: https://my-file-storage.com
Authentication Method: Two-way OAuth 2.0 authorization code flow
Save and continue 
# copy OpenProject OAuth Client ID  and OpenProject OAuth Client Secret  which we use in nextcloud

Next Administration settings >> OpenProject
1 OpenProject server: https://openprojectbim.example.com
save
2 Authentication method
select: Two-way OAuth 2.0 authorization code flow   and save
3. OpenProject OAuth settings
OpenProject OAuth client ID: past_Oauth_from_openproject_to_here
OpenProject OAuth client secret: past_Oauth_from_openproject_to_here
4. Nextcloud OAuth client
copy Nextcloud OAuth client ID and Nextcloud OAuth client secret which we past in Openproject
# On OpenProject past 
Nextcloud OAuth Client ID:
Nextcloud OAuth Client Secret:
Save and continue
# On nextcloud: Yes, I have copied these values
5. Project folders (recommended)
click Automatically managed folders and Setup OpenProject user, group and folder
6. Project folders application connection
Copy Application Password from nextcloud
# On Openproject
Application password: past here and Finish setup

# On Openproject >> Administration >> Files >> nextcloud >> Projects >> Add projects >> Login to Nextcloud >> Grant access >> Put nextcloud admin password
Access granted
# On nextcloud you will see Shared folder as OpenProject
```

## Sample Construction Project Workflow
To demonstrate how OpenProjectBIM and Nextcloud can be used throughout the complete lifecycle of a construction project management

4. The Project is divided into the following phases:
```
| Phase                | Start Date | Finish Date | Main Work                                                  |
| -------------------- | ---------- | ----------- | ---------------------------------------------------------- |
| 01_Foundation        | 2026-07-01 | 2026-07-10  | Site survey, excavation, footing, foundation concrete      |
| 02_GroundFloorSlab   | 2026-07-11 | 2026-07-20  | Formwork, reinforcement, concrete pouring, curing          |
| 03_StructuralFrame   | 2026-07-21 | 2026-08-15  | Columns, beams, slabs, structural inspection               |
| 04_MasonryPlastering | 2026-08-16 | 2026-08-31  | Block work, internal and external plastering               |
| 05_MEPInstallation   | 2026-09-01 | 2026-09-15  | Electrical, plumbing, fire protection, HVAC                |
| 06_FloorFinish       | 2026-09-16 | 2026-09-25  | Flooring, ceiling, painting, doors and windows             |
| 07_Final             | 2026-09-26 | 2026-09-30  | Quality inspection, defect fixing, documentation, handover |
```
4a. To create project management flow on OpenProjectBIM: 
```
Project >> New Project >> 
Name: Building Construction Project
Identifier: building-construction-project
Descritption: building-construction-project management by OpenProjectBIM  
Complete
```
4b. To connect Nextcloud to this Project
```   
   Building Construction Project >> Project settings >> Files >> +Storage >> Save
```
4c. To Create new work package
```
Building Construction Project >> work packages >> Create new work package
```  

   
