#  Wazuh SIEM Deployment — Security Supervision Platform

> **Stage d'initiation — ENSA Fès | Octobre 2025**  
> Mise en place d'une plateforme de supervision de sécurité avec Wazuh dans un environnement virtualisé.

---

##  Description

Ce projet consiste au déploiement complet d'une plateforme **Wazuh SIEM (Security Information and Event Management)** open source dans un environnement virtualisé (VirtualBox), permettant la collecte centralisée, l'analyse en temps réel et la visualisation des événements de sécurité sur des systèmes **Linux et Windows**.

**Problématique :**  
Face au volume croissant d'événements de sécurité et à l'impossibilité d'une analyse manuelle, comment mettre en place une solution de supervision permettant la détection rapide des incidents et la conformité aux normes (ISO 27001, PCI DSS) ?

---

##  Architecture Déployée

```
┌─────────────────────────────────┐     ┌──────────────────────────────┐
│      VM Ubuntu — MANAGER        │     │     VM Windows — AGENT       │
│                                 │     │                              │
│     Wazuh Manager               │◄────►   Wazuh Agent v4.9.2         │
│     Wazuh Indexer (OpenSearch)  │     │  Windows 10 Enterprise       │
│     Wazuh Dashboard             │     │  IP : 10.0.2.16              │
│  IP : 10.0.2.15                 │     │                              │
└─────────────────────────────────┘     └──────────────────────────────┘
         │
         │  Réseau NAT  →  Dashboard accessible via port 8443 → 443
         │  Réseau interne  →  Communication Manager ↔ Agent
```

---

##  Stack Technique

| Composant | Technologie | Version |
|---|---|---|
| SIEM Manager | Wazuh Manager | 4.9.2 |
| Moteur de recherche | OpenSearch (Wazuh Indexer) | 4.9.2 |
| Dashboard | Wazuh Dashboard | 4.9.2 |
| Agent | Wazuh Agent (Windows) | 4.9.2 |
| Virtualisation | VirtualBox | - |
| OS Manager | Ubuntu Server | - |
| OS Agent | Windows 10 Enterprise | - |

---

##  Installation & Déploiement

### 1. Installation du Wazuh Manager (All-in-One)

```bash
# Téléchargement du script d'installation
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh

# Installation automatique (Manager + Indexer + Dashboard + Certificats SSL)
sudo bash wazuh-install.sh -a
```

>  Durée d'installation : ~15-20 minutes  
> Installe automatiquement : Wazuh Manager, Wazuh Indexer, Wazuh Dashboard, Certificats SSL

### 2. Démarrage de l'Agent Windows

```powershell
# Démarrage du service Wazuh sur Windows
NET START WazuhSvc
```

### 3. Configuration de l'Agent (`ossec.conf`)

```xml
<ossec_config>
  <client>
    <server>
      <address>10.0.2.15</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
  </client>
</ossec_config>
```

---

##  Difficultés Rencontrées & Solutions

###  Problème : Erreur d'authentification au Dashboard
```
Error: "Invalid username or password"
Cause : Format de hash incompatible (SHA-512 vs BCrypt)
```

###  Solution appliquée

```python
# Étape 1 : Génération d'un hash BCrypt avec Python
import bcrypt
password = b"VotreMotDePasse"
hashed = bcrypt.hashpw(password, bcrypt.gensalt())
print(hashed.decode())
```

```bash
# Étape 2 : Modification du fichier de configuration
sudo nano /etc/wazuh-indexer/opensearch-security/internal_users.yml

# Étape 3 : Rechargement de la configuration de sécurité
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh \
  -cd /etc/wazuh-indexer/opensearch-security/ \
  -icl -nhnv -cacert /etc/wazuh-indexer/certs/root-ca.pem \
  -cert /etc/wazuh-indexer/certs/admin.pem \
  -key /etc/wazuh-indexer/certs/admin-key.pem

# Étape 4 : Redémarrage des services
sudo systemctl restart wazuh-manager wazuh-indexer wazuh-dashboard
```

**Résultat :  Authentification réussie !**

---

##  Résultats Obtenus

| Indicateur | Résultat |
|---|---|
| Dashboard Wazuh |  Accessible et opérationnel |
| Événements collectés | **137 événements** en temps réel |
| Agent Windows connecté |  Actif (ID: 001, IP: 10.0.2.16) |
| Modules de sécurité |  Actifs (FIM, IDS, SCA) |
| Score SCA (CIS Windows 10) | 29% — 113 passed / 276 failed |

---

##  Fonctionnalités Wazuh Couvertes

-  **Collecte de logs** — systèmes Linux et Windows
-  **Détection d'intrusions (IDS)** — règles de détection en temps réel
-  **Surveillance d'intégrité des fichiers (FIM)**
-  **Security Configuration Assessment (SCA)** — Benchmark CIS
-  **Dashboards OpenSearch/Kibana** — visualisation des événements
-  **Intégration MITRE ATT&CK** — classification des menaces

---

##  Perspectives d'Amélioration

- [ ] Déploiement d'agents supplémentaires (Linux, macOS)
- [ ] Configuration de règles de détection personnalisées
- [ ] Intégration avec threat intelligence externe
- [ ] Mise en place de la réponse active automatisée
- [ ] Déploiement en architecture haute disponibilité

---

##  Contexte Académique

| Élément | Détail |
|---|---|
| **Type** | Stage d'initiation |
| **École** | ENSA Fès — École Nationale des Sciences Appliquées |
| **Spécialité** | Systèmes Communicants & Sécurité Informatique |
| **Période** | Octobre 2025 |
| **Encadrant** | Pr. Younes Balboul |

---

##  Auteure

**Bzite Fatima-Zahra**  
Élève Ingénieure 4ème année — Systèmes Communicants & Sécurité Informatique  
ENSA Fès  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/votre-profil)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=flat&logo=github&logoColor=white)](https://github.com/fatimazahrabzite)
