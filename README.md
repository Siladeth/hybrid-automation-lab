# Hybrid Infrastructure Automation & SAML 2.0 SSO Lab

Automated deployment, configuration management, and security hardening of a multi-tier web architecture (Apache + PostgreSQL) orchestrated via **Ansible** and **Rundeck**, featuring federated Cloud identity authentication with **Microsoft Entra ID (SAML 2.0)**.

## Architecture

\\\mermaid
flowchart LR
    User[User / Browser] -->|1. HTTPS / SAML AuthnRequest| Web[web-node\n192.168.56.11\nApache + mod_auth_mellon]
    Web <-->|2. SAML 2.0 Redirect & Assertion| Entra[Microsoft Entra ID\nCloud IdP]
    Web -->|3. SQL Queries| DB[db-node\n192.168.56.12\nPostgreSQL]
    Control[control-node\n192.168.56.10\nAnsible + Rundeck] -->|SSH / Playbooks| Web
    Control -->|SSH / Playbooks| DB
\\\

### Node Inventory (Vagrant / VirtualBox)

| Node | Private IP | OS / RAM | Role & Services |
| :--- | :--- | :--- | :--- |
| control-node | 192.168.56.10 | Debian 12 (1 GB) | Orchestration (Ansible, Rundeck) |
| web-node | 192.168.56.11 | Debian 12 (512 MB) | Web Server / Reverse Proxy (Apache2, mod_auth_mellon) |
| db-node | 192.168.56.12 | Debian 12 (512 MB) | Relational Database (PostgreSQL) |

## Repository Structure

* Vagrantfile: Provisions the 3 Debian Bookworm virtual machines on a private Host-Only network.
* nsible/inventory.ini: Static inventory grouping target hosts (webservers, dbservers).
* nsible/deploy.yml: Idempotent deployment of the Apache and PostgreSQL stack.
* nsible/secure_web.yml: Configures HTTPS (self-signed OpenSSL TLS certificate) and SAML 2.0 SSO integration via mod_auth_mellon.
* nsible/secure_db.yml: Hardens network access and configures PostgreSQL authentication.

## Key Engineering Challenges & Troubleshooting

During the SAML 2.0 integration between Apache (mod_auth_mellon) and Microsoft Entra ID, a Lasso signature verification failure ([-111]) and post-redirect session drops were diagnosed through /var/log/apache2/error.log:
1. **Certificate Alignment:** Matched the exact IdP signing certificate from idp-metadata.xml with the Service Provider configuration.
2. **Cookie & Session Handling:** Tuned MellonSecureCookie and SameSite directives to preserve session state over self-signed HTTPS endpoints.

## Production SaaS Scalability Considerations

In this local lab environment, **Rundeck** and **Ansible** are co-located on control-node to optimize memory footprint. In a multi-tenant SaaS production environment:
* **Control Plane Isolation:** The orchestrator is decoupled into a centralized, containerized **Management Plane** (Docker / Kubernetes).
* **GitOps Workflow:** Playbooks are never stored locally on execution nodes; each Rundeck job dynamically pulls version-controlled playbooks from Git and targets client environments via dynamic cloud inventories and hardened bastions.
