# Wazuh-SIEM-AWS-Project
Projet de supervision de sécurité avec Wazuh et AWS

# Projet Architecture Cloud - Sécurité des Endpoints et Supervision SIEM avec Wazuh

## 📋 Table des matières

1. [Introduction](#introduction)
2. [Objectifs du Projet](#objectifs)
3. [Préparation de l'environnement AWS](#preparation-aws)
4. [Installation Wazuh All-in-One](#installation-wazuh)
5. [Enrôlement des clients](#enrolement-clients)
6. [Démonstrations SIEM et EDR](#demonstrations)
7. [Conclusion](#conclusion)

---

## 🎯 Introduction <a name="introduction"></a>

Dans un contexte de cybersécurité en constante évolution, les organisations font face à une multiplication des cybermenaces ciblant leurs infrastructures informatiques. Les attaques deviennent de plus en plus sophistiquées, exploitant les vulnérabilités des systèmes d'information pour compromettre les données sensibles et perturber les opérations critiques.

Les environnements IT modernes sont souvent **hétérogènes**, combinant des systèmes Linux et Windows, des infrastructures on-premise et Cloud, ce qui complexifie la tâche de supervision. La visibilité sur l'ensemble de l'infrastructure devient un **enjeu stratégique** pour les équipes de sécurité.

---

## 🎯 Objectifs du Projet <a name="objectifs"></a>

Ce projet vise à mettre en œuvre une plateforme complète de supervision de la sécurité basée sur **Wazuh**, intégrant les approches **SIEM** et **EDR** dans un environnement Cloud AWS.

### Objectifs spécifiques :

- ✅ **Déployer** une infrastructure de sécurité sur AWS (serveur Wazuh + agents multi-OS)
- ✅ **Configurer** la collecte et centralisation des événements de sécurité
- ✅ **Implémenter** des scénarios de test réalistes pour générer des alertes
- ✅ **Analyser** les événements via le dashboard Wazuh
- ✅ **Comprendre** la complémentarité SIEM/EDR dans la détection des menaces
- ✅ **Pratiquer** le Threat Hunting à travers des requêtes ciblées
- ✅ **Développer** une compréhension opérationnelle d'un SOC

---

## ☁️ Préparation de l'environnement AWS <a name="preparation-aws"></a>

### 1. Connexion à AWS Learner Lab

![Connexion AWS Learner Lab](media/image2.png)

*Accès à l'environnement AWS Learner Lab pour le déploiement de l'infrastructure*

---

### 2. Création du VPC et des sous-réseaux

#### Configuration du VPC

- **Nom du VPC** : `workshop`
- **CIDR** : `10.0.0.0/16`

![Création du VPC](media/image3.png)

![Configuration VPC](media/image4.png)

#### Création réussie du VPC

![VPC créé avec succès](media/image5.png)

#### Configuration du Subnet

![Configuration Subnet](media/image6.png)

#### Table de Routes

![Table de Routes](media/image7.png)

#### Passerelle Internet

![Internet Gateway attachée au VPC](media/image8.png)

---

### 3. Création des Security Groups

![Accès aux Security Groups](media/image9.png)

#### Security Group pour les Clients (SG-Clients)

**Règles Entrantes** :
- Port **22/tcp** (SSH) - depuis votre IP
- Port **3389/tcp** (RDP) - depuis votre IP

**Règles Sortantes** :
- Tout le trafic vers toutes les IPs

![Configuration SG-Clients](media/image10.png)

![SG-Clients créé avec succès](media/image11.png)

#### Security Group pour Wazuh Server (SG-Wazuh-Server)

**Règles Entrantes** :
- Port **22/tcp** - SSH depuis votre IP
- Port **443/tcp** - Dashboard HTTPS depuis votre IP
- Port **1514/tcp** - Communication agents depuis SG-Clients
- Port **1515/tcp** - Enrollment agents depuis SG-Clients

**Règles Sortantes** :
- Tout le trafic vers toutes les IPs

![Configuration SG-Wazuh-Server](media/image12.png)

![SG-Wazuh-Server créé avec succès](media/image13.png)

---

### 4. Création des instances EC2

![Lancement d'instance EC2](media/image14.png)

#### Instance Wazuh-Server

**Spécifications** :
- **OS** : Ubuntu 22.04 LTS
- **Type** : t3.large (recommandé)
- **Stockage** : 30 GB

![Configuration Wazuh-Server](media/image15.png)

##### Création de la paire de clés

![Création paire de clés Wazuh](media/image16.png)

![Paire de clés créée](media/image17.png)

##### Configuration réseau

![Configuration réseau Wazuh-Server](media/image18.png)

![Paramètres réseau finaux](media/image19.png)

##### Lancement de l'instance

![Instance Wazuh-Server lancée](media/image20.png)

---

#### Instance Linux-Client

**Spécifications** :
- **OS** : Ubuntu 22.04
- **Type** : t3.micro

![Configuration Linux-Client](media/image21.png)

##### Création de la paire de clés

![Paire de clés Linux-Client](media/image22.png)

![Clés créées](media/image23.png)

##### Configuration réseau

![Réseau Linux-Client](media/image24.png)

##### Lancement

![Linux-Client lancé](media/image25.png)

![Confirmation lancement](media/image26.png)

---

#### Instance Windows-Client

**Spécifications** :
- **OS** : Windows Server

![Configuration Windows-Client](media/image27.png)

##### Création de la paire de clés

![Paire de clés Windows](media/image28.png)

![Clés Windows créées](media/image29.png)

##### Configuration réseau

![Réseau Windows-Client](media/image30.png)

##### Lancement

![Windows-Client lancé](media/image31.png)

![Confirmation Windows](media/image32.png)

---

#### Vue d'ensemble des trois instances

![Les trois instances EC2 déployées](media/image33.png)

---

## 🛡️ Installation Wazuh All-in-One <a name="installation-wazuh"></a>

### 1. Connexion SSH au serveur Wazuh

![Connexion SSH Wazuh](media/image34.png)

![Terminal SSH connecté](media/image35.png)

---

### 2. Mise à jour du système

```bash
sudo apt update && sudo apt -y upgrade
```

![Mise à jour des paquets](media/image36.png)

![Upgrade en cours](media/image37.png)

---

### 3. Téléchargement du script d'installation Wazuh

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

![Téléchargement du script](media/image38.png)

#### Vérification du téléchargement

```bash
ls -lh wazuh-install.sh
```

![Fichier téléchargé](media/image39.png)

---

### 4. Lancement de l'installation

```bash
sudo bash wazuh-install.sh --a
```

![Installation Wazuh en cours](media/image40.png)

![Installation complétée](media/image41.png)

#### 🔐 Identifiants générés

À la fin de l'installation, le script génère les identifiants d'administration :

- **URL Dashboard** : `https://<wazuh-dashboard-ip>:443`
- **User** : `admin`
- **Password** : `QFsi?vvUedlsgXbupZ*RVaixAf5oYhN4`

> ⚠️ **Important** : Ces credentials sont sauvegardés de manière sécurisée pour l'accès au dashboard.

---

### 5. Vérification des services

#### Wazuh Manager

```bash
sudo systemctl status wazuh-manager
```

![Statut Wazuh Manager](media/image42.png)

✅ Le service est **actif** et en fonctionnement depuis 28 minutes.

---

#### Wazuh Indexer

```bash
sudo systemctl status wazuh-indexer
```

![Statut Wazuh Indexer](media/image43.png)

✅ Le service est actif et prêt à **indexer les événements** envoyés par les agents.

---

#### Wazuh Dashboard

```bash
sudo systemctl status wazuh-dashboard
```

![Statut Wazuh Dashboard](media/image44.png)

✅ Le Dashboard est opérationnel et peut recevoir les connexions HTTPS depuis un navigateur.

---

## 👥 Enrôlement des clients <a name="enrolement-clients"></a>

### Connexion à l'interface Wazuh

![Page de connexion Wazuh](media/image45.png)

![Dashboard Wazuh](media/image46.png)

---

### Navigation vers la section Agents

![Section Agents](media/image47.png)

![Ajout d'agent](media/image48.png)

![Configuration agent](media/image49.png)

---

### ⚠️ Note importante sur les adresses IP

**Il faut utiliser l'adresse IP privée du serveur Wazuh** car :

- Dans un **VPC AWS**, les instances dans le même VPC **ne communiquent pas via l'IP publique** par défaut
- Le serveur ne reçoit donc **jamais la demande d'enrôlement** avec l'IP publique
- C'est pourquoi le Dashboard montrait **0 agents** initialement

#### Correction effectuée :

![Correction IP privée](media/image50.png)

![Configuration IP privée](media/image51.png)

![Commandes d'installation](media/image52.png)

---

## 🐧 Enrôlement du client Linux

### 1. Connexion SSH au Linux-Client

![Connexion Linux-Client](media/image53.png)

![Terminal Linux-Client](media/image54.png)

---

### 2. Installation de l'agent Wazuh

#### Téléchargement et installation

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.5-1_amd64.deb && \
sudo WAZUH_MANAGER='54.196.143.108' \
WAZUH_AGENT_GROUP='default' \
WAZUH_AGENT_NAME='Linux-client' \
dpkg -i ./wazuh-agent_4.7.5-1_amd64.deb
```

![Installation agent Linux](media/image55.png)

✅ **Wazuh Agent installé avec succès sur Linux-Client**

---

#### Recharge de systemd

```bash
sudo systemctl daemon-reload
```

![Daemon reload](media/image56.png)

---

#### Activation du service

```bash
sudo systemctl enable wazuh-agent
```

![Enable service](media/image57.png)

---

#### Démarrage du service

```bash
sudo systemctl start wazuh-agent
```

![Start service](media/image58.png)

---

#### Vérification du statut

```bash
sudo systemctl status wazuh-agent
```

![Statut agent Linux](media/image59.png)

✅ **Agent Linux actif et connecté au serveur Wazuh**

---

### 3. Vérification dans le Dashboard

![Agent Linux dans Dashboard](media/image60.png)

![Détails agent Linux](media/image61.png)

✅ **L'agent Linux-client apparaît bien dans le Dashboard Wazuh**

---

## 🪟 Enrôlement du client Windows

### 1. Connexion RDP au Windows-Client

#### Lancement de la connexion Bureau à distance

```
Commande : mstsc (Windows + R)
```

![Commande mstsc](media/image62.png)

---

#### Saisie de l'IP publique

**IP publique du Windows-Client** : `34.230.78.148`

![Connexion RDP](media/image63.png)

![Avertissement certificat](media/image64.png)

---

### 2. Récupération du mot de passe

#### Téléchargement de la paire de clés

![Téléchargement clés Windows](media/image65.png)

---

#### Déchiffrement du mot de passe

![Décryption mot de passe](media/image66.png)

![Mot de passe généré](media/image67.png)

---

#### Authentification RDP

![Saisie mot de passe](media/image68.png)

![Chargement session](media/image69.png)

![Windows Server démarré](media/image70.png)

✅ **Connexion réussie au Windows Server**

---

### 3. Installation de l'agent Wazuh sur Windows

![Configuration agent Windows](media/image71.png)

![Commandes PowerShell](media/image72.png)

![Script d'installation](media/image73.png)

![Installation en cours](media/image74.png)

---

#### Exécution dans PowerShell (Admin)

![PowerShell Admin](media/image75.png)

✅ **Agent Wazuh installé avec succès sur Windows-Client**

---

### 4. Vérification dans le Dashboard

![Agent Windows dans Dashboard](media/image76.png)

![Détails agent Windows](media/image77.png)

✅ **Les deux agents (Linux et Windows) sont maintenant enrôlés et actifs**

---

## 🔍 Démonstrations SIEM et EDR <a name="demonstrations"></a>

### 🐧 Démo SIEM côté Linux

#### Scénario 1 : Tentatives SSH échouées (Brute Force simulé)

```bash
ssh fakeuser@172.31.26.179
```

![Tentatives SSH](media/image78.png)

**Résultat dans Wazuh** :

![Alertes brute force SSH](media/image79.png)

✅ **Wazuh détecte les tentatives d'authentification SSH échouées**

---

#### Scénario 2 : Élévation de privilèges

```bash
sudo su
```

![Élévation privilèges](media/image80.png)

**Alerte Wazuh** :

![Alerte sudo](media/image81.png)

✅ **Wazuh détecte l'utilisation de sudo et l'élévation de privilèges**

---

#### Scénario 3 : Modification fichier sensible (FIM)

```bash
echo "test" | sudo tee -a /etc/passwd
```

![Modification /etc/passwd](media/image82.png)

**Alerte FIM (File Integrity Monitoring)** :

![Alerte FIM](media/image83.png)

✅ **Wazuh détecte la modification du fichier sensible /etc/passwd**

---

### 🪟 Démo EDR côté Windows

#### Scénario 1 : Échecs de login RDP (Event ID 4625)

**Action** : Tentatives de connexion RDP avec mauvais mot de passe (5 fois)

![Tentatives RDP échouées](media/image84.png)

![Logon failed](media/image85.png)

**Alertes dans Wazuh** :

![Filtrage par agent Windows](media/image86.png)

![Alertes échecs authentification](media/image87.png)

✅ **Wazuh détecte les tentatives d'authentification RDP échouées (Event ID 4625)**

---

#### Scénario 2 : Création d'un utilisateur local

```powershell
net user labuser P@ssw0rd! /add
net localgroup administrators labuser /add
```

![Création utilisateur](media/image88.png)

**Alerte Wazuh** :

![Alerte création utilisateur](media/image89.png)

✅ **Wazuh détecte la création d'un nouvel utilisateur et son ajout au groupe Administrateurs**

---

#### Scénario 3 : Installation de Sysmon (EDR enrichi)

**Sysmon** permet une surveillance approfondie des activités système Windows.

```powershell
Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile C:\Sysmon.zip
Expand-Archive -Path C:\Sysmon.zip -DestinationPath C:\Sysmon
Invoke-WebRequest -Uri https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile C:\Sysmon\sysmonconfig.xml
C:\Sysmon\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig.xml
```

![Installation Sysmon](media/image90.png)

![Sysmon installé](media/image91.png)

✅ **Sysmon installé et configuré pour une supervision EDR enrichie**

---

## 🎓 Conclusion <a name="conclusion"></a>

Ce projet nous a permis de **déployer une infrastructure de supervision de la sécurité** basée sur **Wazuh** dans un environnement **Cloud AWS**. La plateforme mise en place supervise des machines **Linux** et **Windows Server** à travers une solution **SIEM/EDR centralisée**.

### 📊 Résultats obtenus

Les différents scénarios de test réalisés ont généré des alertes pertinentes :

- ✅ **Brute force SSH** détecté
- ✅ **Échecs d'authentification RDP** identifiés
- ✅ **Élévations de privilèges** surveillées
- ✅ **Création de comptes** alertée
- ✅ **Modifications de fichiers sensibles (FIM)** détectées

Ces résultats démontrent l'**efficacité de la détection en temps réel** des activités suspectes.

### 🎯 Compétences développées

Sur le plan pédagogique, ce laboratoire nous a permis d'approfondir :

- 🔧 **Déploiement d'infrastructure Cloud** (AWS)
- 🖥️ **Administration système** multi-OS (Linux/Windows)
- 📈 **Analyse de logs** et corrélation d'événements
- 🔍 **Threat Hunting** et investigation
- 🛡️ **Complémentarité SIEM/EDR** dans la détection des menaces
- 🏢 **Fonctionnement opérationnel d'un SOC**

### 🚀 Perspectives

Ce projet constitue une **expérience pratique enrichissante** et une **base solide** pour notre future carrière en cybersécurité. Il nous familiarise avec les principes et le fonctionnement d'un **Security Operations Center (SOC)** moderne, tout en nous donnant une vision concrète des défis de la supervision de sécurité dans des environnements hybrides.

---

## 📚 Références

- [Documentation Wazuh](https://documentation.wazuh.com/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Sysmon Documentation](https://docs.microsoft.com/sysinternals/downloads/sysmon)

---

**Département Mathématique et Informatique**  
**Filière : II-BDCC**  
**Projet : Architecture Cloud - Sécurité des Endpoints et Supervision SIEM**
