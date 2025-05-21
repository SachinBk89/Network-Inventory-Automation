Network Inventory Automation
🧭 Purpose
This project automates the generation of a structured, categorized inventory of network devices by querying NetBox—a powerful network source-of-truth and IPAM tool. The goal is to build a dynamic inventory grouped by site, vendor, or device role, which can be used by other Ansible automation tasks, audits, or reporting pipelines.

📁 Folder Structure Overview
graphql
Copy
Edit
02-network-inventory/
├── inventories/
│   └── netbox_inventory.yml        # NetBox plugin-based inventory config
├── playbook.yml                    # Generic playbook (placeholder)
├── generate_inventory.yml          # Main playbook to query & structure NetBox data
├── output/                         # Folder storing generated YAML inventory
│   ├── inventory_by_site.yml
│   ├── inventory_by_vendor.yml
│   └── inventory_by_role.yml
├── plugins/
│   └── inventory/
│       └── netbox_inventory.yml    # Optional NetBox dynamic inventory plugin definition
└── README.md                       # Documentation
📘 How It Works (generate_inventory.yml)
This playbook is designed to be run from localhost, as it queries the NetBox API directly. Here's a step-by-step explanation of its logic:

1. 🔐 Authenticate with NetBox
yaml
Copy
Edit
- name: Query NetBox and group devices by site
  uri:
    url: "{{ netbox_url }}/api/dcim/devices/?limit=0"
    method: GET
    headers:
      Authorization: "Token {{ netbox_token }}"
This task authenticates with the NetBox API using a personal API token.

It retrieves all network devices (/api/dcim/devices/ endpoint).

2. 📦 Group Devices by Site
yaml
Copy
Edit
grouped_by_site: >-
  {{ netbox_response.json.results | groupby('site.slug') | dict }}
The list of devices is grouped based on the site slug, representing physical locations or data centers.

3. 💾 Save Output (Site)
yaml
Copy
Edit
- name: Save site-based inventory
  copy:
    content: "{{ grouped_by_site | to_nice_yaml }}"
    dest: output/inventory_by_site.yml
The site-based inventory is saved in YAML format under the output/ folder.

4. 🏭 Group and Save by Vendor
yaml
Copy
Edit
grouped_by_vendor: >-
  {{ netbox_response.json.results | groupby('device_type.manufacturer.name') | dict }}
Groups by hardware vendor (e.g., Cisco, Juniper, Arista).

Output saved to inventory_by_vendor.yml.

5. 🧱 Group and Save by Role
yaml
Copy
Edit
grouped_by_role: >-
  {{ netbox_response.json.results | groupby('device_role.name') | dict }}
Groups devices by their assigned role (e.g., edge-router, access-switch, core-switch).

Output saved to inventory_by_role.yml.



┌────────────────────────────┐
│ 1. Ansible Control Node    │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ 2. Connect to NetBox API   │
│    - GET /api/dcim/devices │
│    - Auth via API Token    │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ 3. Fetch All Device Data   │
│    (JSON format)           │
└────────────┬───────────────┘
             │
             ▼
┌──────────────────────────────────────────────┐
│ 4. Parse Devices and Group by:               │
│    - Site (site.slug)                        │
│    - Vendor (device_type.manufacturer.name)  │
│    - Role (device_role.name)                 │
└────────────┬─────────────────────────┬───────┘
             │                         │
             ▼                         ▼
┌────────────────────┐      ┌────────────────────┐
│ 5a. Save site.yml   │      │ 5b. Save vendor.yml│
│  → output/inventory_by_site.yml  │
└────────────────────┘      └────────────────────┘
             │                         │
             ▼                         ▼
        ┌────────────────────────────┐
        │ 5c. Save role.yml          │
        │  → output/inventory_by_role.yml │
        └────────────────────────────┘
             │
             ▼
┌────────────────────────────┐
│ 6. Dynamic Inventory Ready │
└────────────────────────────┘


