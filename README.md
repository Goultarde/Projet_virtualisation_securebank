# Projet Virtualisation Sécurisée - Application Bancaire SecureBank

## 📋 Description

Ce projet présente une application bancaire en C# (ASP.NET Core) sécurisée avec une infrastructure moderne utilisant Vagrant, Ansible et Docker. L'application a été conçue pour démontrer les bonnes pratiques de sécurité dans un environnement bancaire.

## 🏗️ Architecture

L'infrastructure se compose de 3 machines virtuelles :

- **NTP Server** (192.168.56.42) : Synchronisation temporelle sécurisée
- **PKI Server** (192.168.56.41) : Gestion des certificats CFSSL
- **SecureBank Web** (192.168.56.40) : Application principale avec HAProxy

## 🚀 Installation et Démarrage

### Prérequis

- **Vagrant** (version 2.2+)
- **VirtualBox** (version 6.0+)
- **Ansible** (version 2.9+)
- **Docker** (version 19.0+)
- **Git**

### Installation des Prérequis

*Dépend du système utiliser, veuillez lire les documentationt des différent prérequis.*

### Démarrage de l'Application

#### 1. Cloner le Repository
```bash
git clone https://github.com/Goultarde/Projet_virtualisation_securebank
cd Projet_virtualisation_securebank
```

#### 2. Lancer les Machines Virtuelles
```bash
cd vagrant
vagrant up
```

Cette commande va :
- Créer 3 VMs Ubuntu
- Configurer le réseau privé
- Installer les dépendances
- Déployer l'infrastructure sécurisée

#### 3. Vérifier le Déploiement
```bash
# Vérifier le statut des VMs
vagrant status

# Se connecter à la VM SecureBank
vagrant ssh securebank-web

# Vérifier les services
sudo systemctl status haproxy
sudo systemctl status cfssl
docker ps
```

#### 4. Accéder à l'Application

Une fois le déploiement terminé, vous pouvez accéder à :

- **Application Bancaire** : https://securebank.com
- **Interface MailDev** : https://securebank.com/mail

> **Note** : Ajoutez `securebank.com` dans votre fichier `/etc/hosts` :
> ```bash
> echo "192.168.56.40 securebank.com" | sudo tee -a /etc/hosts
> ```


## 📊 Monitoring et Logs

### Logs de l'Application
```bash
# Logs SecureBank
tail -f ansible/docker/logs/securebank/$(date +%Y-%m-%d)_general.json

# Logs StoreAPI
tail -f ansible/docker/logs/storeapi/$(date +%Y-%m-%d).json

# Logs d'accès
tail -f ansible/docker/logs/securebank/accessLog.json
```

### Monitoring des Services
```bash
# HAProxy
sudo systemctl status haproxy

# CFSSL
sudo systemctl status cfssl

# Docker
docker ps
docker logs securebank
```

## 🛠️ Maintenance

### Sauvegarde de la Base de Données
```bash
# Les sauvegardes sont automatiques toutes les 30 secondes
ls -la ansible/docker/backups/
```


### Redémarrage des Services
```bash
# Redémarrer HAProxy
sudo systemctl restart haproxy

# Redémarrer les conteneurs
cd ansible/docker
docker-compose restart
```

## 🚨 Dépannage

### Problèmes Courants

#### 1. VMs ne démarrent pas
```bash
# Vérifier VirtualBox
VBoxManage list vms
VBoxManage startvm <nom-vm>

# Vérifier Vagrant
vagrant status
vagrant reload
```

#### 2. Services non accessibles
```bash
# Vérifier les ports
netstat -tlnp | grep :80
netstat -tlnp | grep :443

# Vérifier les logs
sudo journalctl -u haproxy -f
docker logs securebank
```

#### 3. Problèmes de certificats
```bash
# Vérifier CFSSL
curl http://192.168.56.41:8888/api/v1/cfssl/health

# Régénérer les certificats
sudo rm /etc/haproxy/certs/securebank.pem
sudo /usr/local/bin/renew-cert.sh
```

## 📚 Documentation Complète

Pour plus de détails sur l'architecture et la sécurité, consultez :
- [Rapport d'Architecture Sécurisée](RAPPORT_ARCHITECTURE_SECURISEE.md)
- [Infrastructure Finale Souhaitée](INFRASTRUCTURE_FINALE.md)
- [Diagrammes d'Architecture](DIAGRAMME_ARCHITECTURE.md)

## ⚠️ Avertissements

- **NE PAS UTILISER EN PRODUCTION** : Cette application contient des vulnérabilités intentionnelles pour l'apprentissage
- **Environnement isolé** : Utilisez uniquement dans un environnement de développement contrôlé
- **Données de test** : Toutes les données sont factices et ne représentent pas de vraies transactions bancaires

## 📞 Support

Pour toute question ou problème :
1. Consultez la documentation
2. Vérifiez les logs
3. Contactez l'équipe de développement

---

**Projet réalisé dans le cadre d'un cours de sécurité informatique**  
**Technologies : ASP.NET Core, Docker, Vagrant, Ansible, HAProxy, CFSSL**
