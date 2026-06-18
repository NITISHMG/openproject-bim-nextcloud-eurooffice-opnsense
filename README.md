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
   To deploy Nextcloud on your VM, create the directory structure and adjust the file ownership and permissions as follows:
```
mkdir -p /path/to/nextcloud/config/nextcloud
mkdir -p /path/to/nextcloud/data/{mariadb,redis,nextcloud-html,nextcloud-data}
chown -R www-data:www-data /path/to/nextcloud
find /path/to/nextcloud/ -type d -exec chmod 750 {} \;
find /path/to/nextcloud/ -type f -exec chmod 640 {} \;
```
