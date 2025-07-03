# Infrastructure Finale Souhaitée - SecureBank

## 📋 Vue d'Ensemble

L'infrastructure finale souhaitée pour l'application bancaire SecureBank est une architecture sécurisée multi-niveaux utilisant la virtualisation et la conteneurisation pour garantir la sécurité, la disponibilité et la scalabilité.

## 🏗️ Architecture Globale

### 1.1 Topologie Réseau

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INFRASTRUCTURE FINALE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   NTP Server    │    │   PKI Server    │    │  SecureBank     │         │
│  │ 192.168.56.42   │    │ 192.168.56.41   │    │ 192.168.56.40   │         │
│  │                 │    │                 │    │                 │         │
│  │ • NTP Service   │    │ • CFSSL API     │    │ • HAProxy       │         │
│  │ • Time Sync     │    │ • Certificates  │    │ • SSL/TLS       │         │
│  │ • Security      │    │ • PKI Mgmt      │    │ • WAF Rules     │         │
│  │ • Logs          │    │ • Auto Renewal  │    │ • Load Balance  │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
│           │                       │                       │                 │
│           └───────────────────────┼───────────────────────┘                 │
│                                   │                                         │
│  ┌─────────────────────────────────┼─────────────────────────────────────────┐
│  │                    RÉSEAU PRIVÉ VAGRANT (192.168.56.0/24)                │
│  └─────────────────────────────────┼─────────────────────────────────────────┘
│                                   │                                         │
│  ┌─────────────────────────────────┼─────────────────────────────────────────┐
│  │                        DOCKER COMPOSE STACK                              │
│  │                                                                           │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  │ SecureBank  │  │  StoreAPI   │  │   SQL       │  │   MailDev   │     │
│  │  │   App       │  │             │  │  Server     │  │             │     │
│  │  │ Port: 1337  │  │ Port: 1338  │  │ Port: 1433  │  │ Port: 1080  │     │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘     │
│  │                                                                           │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                      │
│  │  │ SQL Server  │  │ Replication │  │   Backup    │                      │
│  │  │ Secondary   │  │   Agent     │  │   Agent     │                      │
│  │  │ Port: 1434  │  │             │  │             │                      │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                      │
│  └───────────────────────────────────────────────────────────────────────────┘
└─────────────────────────────────────────────────────────────────────────────┘
```

## 🖥️ Machines Virtuelles (Vagrant)

### 2.1 NTP Server (192.168.56.42)

**Rôle :** Synchronisation temporelle sécurisée

**Services :**
- **NTP Service** : Synchronisation avec des serveurs NTP publics
- **Time Sync** : Maintien de l'heure précise pour tous les services
- **Security** : Configuration sécurisée du service NTP
- **Logs** : Journalisation des événements de synchronisation

**Configuration :**
```yaml
VM: ntp-server
IP: 192.168.56.42
OS: Ubuntu Bionic (18.04)
Playbook: ntp_main.yml
```

### 2.2 PKI Server (192.168.56.41)

**Rôle :** Infrastructure à clés publiques (PKI)

**Services :**
- **CFSSL API** : API REST pour la gestion des certificats
- **Certificates** : Génération et gestion des certificats SSL/TLS
- **PKI Mgmt** : Gestion de l'autorité de certification
- **Auto Renewal** : Renouvellement automatique des certificats

**Configuration :**
```yaml
VM: pki-server
IP: 192.168.56.41
OS: Ubuntu Bionic (18.04)
Playbook: pki_main.yml
```

### 2.3 SecureBank Web (192.168.56.40)

**Rôle :** Serveur web principal avec reverse proxy

**Services :**
- **HAProxy** : Reverse proxy et load balancer
- **SSL/TLS** : Termination SSL avec certificats CFSSL
- **WAF Rules** : Règles de pare-feu applicatif web
- **Load Balance** : Répartition de charge entre les services

**Configuration :**
```yaml
VM: securebank-web
IP: 192.168.56.40
OS: Ubuntu Bionic (18.04)
Playbook: main.yml
```

## 🐳 Services Conteneurisés (Docker)

### 3.1 SecureBank Application

**Rôle :** Application bancaire principale

**Configuration :**
```yaml
Service: securebank
Port: 1337:80
Dépendances: mssql, maildev, storeapi
Environnement: 
  - BaseUrl: https://securebank.com
  - StoreEndpoint: https://securebank.com/api/Store/
  - CTF: Enabled=false
```

**Fonctionnalités :**
- Interface utilisateur bancaire
- Gestion des comptes et transactions
- Authentification et autorisation
- Intégration avec StoreAPI

### 3.2 StoreAPI

**Rôle :** API de gestion de la boutique

**Configuration :**
```yaml
Service: storeapi
Port: 1338:80
Dépendances: mssql
Environnement:
  - Database: store
  - Seed: true
```

**Fonctionnalités :**
- Gestion des produits
- Historique des achats
- API REST sécurisée

### 3.3 SQL Server Primary

**Rôle :** Base de données principale

**Configuration :**
```yaml
Service: mssql
Port: 1433:1433
Image: mcr.microsoft.com/mssql/server
Environnement:
  - ACCEPT_EULA: y
  - SA_PASSWORD: Your_Password123
```

**Fonctionnalités :**
- Base de données SecureBank
- Base de données Store
- Sauvegarde automatique

### 3.4 SQL Server Secondary

**Rôle :** Base de données secondaire (réplication)

**Configuration :**
```yaml
Service: mssql-secondary
Port: 1434:1433
Image: mcr.microsoft.com/mssql/server
Environnement:
  - ACCEPT_EULA: y
  - SA_PASSWORD: Your_Password123
```

**Fonctionnalités :**
- Réplication des données
- Haute disponibilité
- Récupération après sinistre

### 3.5 Replication Agent

**Rôle :** Agent de réplication automatique

**Configuration :**
```yaml
Service: replication-agent
Image: mcr.microsoft.com/mssql-tools
Dépendances: mssql, mssql-secondary
```

**Fonctionnalités :**
- Sauvegarde automatique toutes les 30 secondes
- Restauration sur le serveur secondaire
- Monitoring de la réplication

### 3.6 MailDev

**Rôle :** Serveur mail de développement

**Configuration :**
```yaml
Service: maildev
Ports: 
  - 127.0.0.1:1080:80
  - 1025:25
Image: maildev/maildev:1.1.0
```

**Fonctionnalités :**
- Interface web pour visualiser les emails
- Serveur SMTP pour les tests
- Capture des emails de l'application

## 🔒 Sécurité

### 4.1 Couches de Sécurité

1. **Périmètre** : HAProxy avec WAF
2. **Réseau** : Réseau privé Vagrant
3. **Application** : Authentification et autorisation
4. **Données** : Chiffrement et réplication

### 4.2 Certificats SSL/TLS

- **Génération** : CFSSL sur le serveur PKI
- **Renouvellement** : Automatique via scripts cron
- **Distribution** : HAProxy pour la termination SSL

### 4.3 Monitoring et Logs

- **Logs d'accès** : HAProxy
- **Logs applicatifs** : SecureBank et StoreAPI
- **Logs de base de données** : SQL Server
- **Logs de réplication** : Agent de réplication

## 📊 Flux de Données

### 5.1 Flux d'Authentification

```
Client → HAProxy (SSL/TLS) → SecureBank → SQL Server
   ↓           ↓                ↓            ↓
Validation → Headers Sec → Session Mgmt → Encryption
```

### 5.2 Flux de Transactions

```
Client → HAProxy → SecureBank → StoreAPI → SQL Server
   ↓        ↓          ↓           ↓           ↓
HTTPS → WAF → Auth → Validation → Audit → Backup
```

### 5.3 Flux de Réplication

```
SQL Primary → Backup → Replication Agent → SQL Secondary
     ↓           ↓            ↓               ↓
   Data → .bak files → Restore → Synchronized
```

## 🚀 Déploiement

### 6.1 Étapes de Déploiement

1. **Préparation** : Installation des prérequis
2. **Infrastructure** : Déploiement des VMs avec Vagrant
3. **Services** : Configuration des services avec Ansible
4. **Applications** : Déploiement des conteneurs Docker
5. **Validation** : Tests de sécurité et de performance

### 6.2 Commandes de Déploiement

```bash
# Déploiement complet
cd vagrant
vagrant up

# Vérification
vagrant status
vagrant ssh securebank-web
docker ps
```

## 📈 Avantages de cette Architecture

### 7.1 Sécurité
- **Isolation** : VMs séparées pour chaque rôle critique
- **Chiffrement** : SSL/TLS end-to-end
- **Monitoring** : Logs centralisés et alertes
- **Réplication** : Protection contre la perte de données

### 7.2 Scalabilité
- **Conteneurs** : Déploiement rapide et reproductible
- **Load Balancing** : HAProxy pour la répartition de charge
- **Microservices** : Architecture modulaire

### 7.3 Maintenabilité
- **Infrastructure as Code** : Vagrant et Ansible
- **Configuration centralisée** : Variables d'environnement
- **Documentation** : Procédures détaillées

## 🔧 Maintenance

### 8.1 Sauvegardes
- **Automatiques** : Toutes les 30 secondes
- **Stockage** : Volume Docker persistant
- **Récupération** : Procédures documentées

### 8.2 Monitoring
- **Services** : Statut des conteneurs et VMs
- **Performance** : Métriques de charge
- **Sécurité** : Alertes sur les événements suspects

### 8.3 Mises à Jour
- **Applications** : Nouveaux builds Docker
- **Infrastructure** : Playbooks Ansible
- **Sécurité** : Renouvellement des certificats

---

**Cette infrastructure finale représente une solution bancaire sécurisée, scalable et maintenable, adaptée aux exigences modernes de sécurité informatique.** 