# openproject-bim-nextcloud-eurooffice-opnsense
OpenProject BIM, Nextcloud, EuroOffice/OnlyOffice, and OPNsense HAProxy integration using Docker Compose with reverse proxy, document collaboration, and self-hosted infrastructure.
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
