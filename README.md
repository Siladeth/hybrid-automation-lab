# Hybrid Infrastructure Automation & SAML 2.0 SSO Lab

Automatisation du déploiement et du durcissement d'une architecture web multi-tiers (Apache + PostgreSQL) orchestrée par **Ansible** et **Rundeck**, avec authentification fédérée via **Microsoft Entra ID (SAML 2.0)**.

## Architecture

\\\mermaid
flowchart LR
    User[Utilisateur / Navigateur] -->|1. HTTPS / SAML AuthnRequest| Web[web-node\n192.168.56.11\nApache + mod_auth_mellon]
    Web <-->|2. Redirection & Assertion SAML 2.0| Entra[Microsoft Entra ID\nCloud IdP]
    Web -->|3. Requêtes SQL| DB[db-node\n192.168.56.12\nPostgreSQL]
    Control[control-node\n192.168.56.10\nAnsible + Rundeck] -->|SSH / Playbooks| Web
    Control -->|SSH / Playbooks| DB
\\\

### Inventaire des nœuds (Vagrant / VirtualBox)

| Nœud | IP Privée | OS / RAM | Rôle & Services |
| :--- | :--- | :--- | :--- |
| control-node | 192.168.56.10 | Debian 12 (1 Go) | Orchestration (Ansible, Rundeck) |
| web-node | 192.168.56.11 | Debian 12 (512 Mo) | Serveur Web / Reverse Proxy (Apache2, mod_auth_mellon) |
| db-node | 192.168.56.12 | Debian 12 (512 Mo) | Base de données (PostgreSQL) |

## Structure du projet

* Vagrantfile : Provisioning des 3 machines virtuelles Debian Bookworm sur un réseau privé Host-Only.
* nsible/inventory.ini : Inventaire statique groupant les cibles (webservers, dbservers).
* nsible/deploy.yml : Déploiement idempotent de la stack Apache et PostgreSQL.
* nsible/secure_web.yml : Configuration HTTPS (certificat TLS OpenSSL) et intégration SSO SAML 2.0 via mod_auth_mellon.
* nsible/secure_db.yml : Durcissement de l'accès et configuration de PostgreSQL.

## Défis techniques résolus (Troubleshooting SAML)

Lors de l'intégration entre Apache (mod_auth_mellon) et Microsoft Entra ID, une erreur de validation de signature Lasso ([-111]) ainsi qu'une perte de session post-redirection ont été diagnostiquées via /var/log/apache2/error.log :
1. **Alignement des certificats :** Mise en correspondance exacte du certificat de signature IdP (idp-metadata.xml) avec la configuration du Service Provider.
2. **Gestion des cookies :** Ajustement des directives MellonSecureCookie et SameSite pour maintenir le contexte de session sur HTTPS autosigné.

## Évolution vers un environnement de Production SaaS

Dans ce labo local, **Rundeck** et **Ansible** sont mutualisés sur le control-node afin d'optimiser l'empreinte mémoire. En production SaaS multi-clients :
* **Séparation des plans :** L'orchestrateur est isolé sur un *Management Plane* centralisé et conteneurisé (Docker / Kubernetes).
* **Approche GitOps :** Les playbooks ne résident pas sur le serveur d'exécution ; chaque job tire dynamiquement la dernière version validée depuis Git et cible les environnements clients via des inventaires dynamiques et des bastions sécurisés.
