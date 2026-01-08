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

![Connexion AWS Learner Lab](images/Aws_console.png)

*Accès à l'environnement AWS Learner Lab pour le déploiement de l'infrastructure*

---

### 2. Création du VPC et des sous-réseaux

#### Configuration du VPC

- **Nom du VPC** : `workshop`
- **CIDR** : `10.0.0.0/16`

![Création du VPC](images/creation_vpc.png)

![Configuration VPC](images/config_vpc.png)

#### Création réussie du VPC

![VPC créé avec succès](images/creation_vpc_succes.png)

#### Configuration du Subnet

![Configuration Subnet](images/Configuration_Subnet.png)

#### Table de Routes

![Table de Routes](images/Table_Routes.png)

#### Passerelle Internet

![Internet Gateway attachée au VPC](images/Passerelle_Internet.png)

---

### 3. Création des Security Groups

![Accès aux Security Groups](images/security_group.png)

#### Security Group pour les Clients (SG-Clients)

**Règles Entrantes** :
- Port **22/tcp** (SSH) - depuis votre IP
- Port **3389/tcp** (RDP) - depuis votre IP

**Règles Sortantes** :
- Tout le trafic vers toutes les IPs

![Configuration SG-Clients](images/SG-Clients.png)

![SG-Clients créé avec succès](images/SG-Clients_succes.png)

#### Security Group pour Wazuh Server (SG-Wazuh-Server)

**Règles Entrantes** :
- Port **22/tcp** - SSH depuis votre IP
- Port **443/tcp** - Dashboard HTTPS depuis votre IP
- Port **1514/tcp** - Communication agents depuis SG-Clients
- Port **1515/tcp** - Enrollment agents depuis SG-Clients

**Règles Sortantes** :
- Tout le trafic vers toutes les IPs

![Configuration SG-Wazuh-Server](images/SG_Wazuh_Server.png)

![SG-Wazuh-Server créé avec succès](images/SG_Wazuh_Server_succes.png)

---

### 4. Création des instances EC2

![Lancement d'instance EC2](images/Lancement_instance.png)

#### Instance Wazuh-Server

**Spécifications** :
- **OS** : Ubuntu 22.04 LTS
- **Type** : t3.large (recommandé)
- **Stockage** : 30 GB

![Configuration Wazuh-Server](images/Configuration_Wazuh-Server.png)

##### Création de la paire de clés

![Création paire de clés Wazuh](images/Création_paire_clés_Wazuh.png)

![Paire de clés créée](images/Paire_clés_créée.png)

##### Configuration réseau

![Configuration réseau Wazuh-Server](images/Configuration_réseau_Wazuh-Server.png)


##### Lancement de l'instance

![Instance Wazuh-Server lancée](images/Instance_Wazuh-Server_lancée.png)

---

#### Instance Linux-Client

**Spécifications** :
- **OS** : Ubuntu 22.04
- **Type** : t3.micro

![Configuration Linux-Client](images/Configuration_Linux-Client.png)

![Configuration Linux-Client](images/Configuration_Linux-Client2.png)

##### Création de la paire de clés

![Paire de clés Linux-Client](images/Paire_clés_Linux-Client.png)


##### Configuration réseau

![Réseau Linux-Client](images/RéseauLinux-Client.png)



---

#### Instance Windows-Client

**Spécifications** :
- **OS** : Windows Server

![Configuration Windows-Client](images/image27.png)

##### Création de la paire de clés

![Paire de clés Windows](images/Paire_clés_Windows.png)


##### Configuration réseau

![Réseau Windows-Client](images/Réseau_Windows-Client.png)

##### Lancement

![Windows-Client lancé](images/Windows-Client_lancé.png)

![Confirmation Windows](image/Confirmation_Windows.png)

---

#### Vue d'ensemble des trois instances

![Les trois instances EC2 déployées](images/Trois_instances_EC2_déployées.png)

---

## 🛡️ Installation Wazuh All-in-One <a name="installation-wazuh"></a>

### 1. Connexion SSH au serveur Wazuh

![Connexion SSH Wazuh](images/Connexio_SSH_Wazuh.png)

![Terminal SSH connecté](images/Terminal_SSH_connecté.png)

---

### 2. Mise à jour du système

```bash
sudo apt update && sudo apt -y upgrade
```

![Mise à jour des paquets](images/MAJ_paquets.png)


---

### 3. Téléchargement du script d'installation Wazuh

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

![Téléchargement du script](images/Téléchargement_script.png)

#### Vérification du téléchargement

```bash
ls -lh wazuh-install.sh
```

![Fichier téléchargé](images/Fichier_téléchargé.png)

---

### 4. Lancement de l'installation

```bash
sudo bash wazuh-install.sh --a
```

![Installation Wazuh en cours](images/Installation_Wazuh_encours.png)

![Installation complétée](images/Installation_complété.png)

#### 🔐 Identifiants générés

À la fin de l'installation, le script génère les identifiants d'administration :

- **URL Dashboard** : `https://<wazuh-dashboard-ip>:443`
- **User** : `admin`
- **Password** : ``

> ⚠️ **Important** : Ces credentials sont sauvegardés de manière sécurisée pour l'accès au dashboard.

---

### 5. Vérification des services

#### Wazuh Manager

```bash
sudo systemctl status wazuh-manager
```

![Statut Wazuh Manager](images/Statut_Wazuh_Manager.png)

✅ Le service est **actif** et en fonctionnement depuis 28 minutes.

---

#### Wazuh Indexer

```bash
sudo systemctl status wazuh-indexer
```

![Statut Wazuh Indexer](images/Statut_Wazuh_Indexer.png)

✅ Le service est actif et prêt à **indexer les événements** envoyés par les agents.

---

#### Wazuh Dashboard

```bash
sudo systemctl status wazuh-dashboard
```

![Statut Wazuh Dashboard](images/Statut_Wazuh_Dashboard.png)

✅ Le Dashboard est opérationnel et peut recevoir les connexions HTTPS depuis un navigateur.

---

## 👥 Enrôlement des clients <a name="enrolement-clients"></a>

### Connexion à l'interface Wazuh

![Page de connexion Wazuh](images/Page_connexion_Wazuh.png)

![Dashboard Wazuh](images/Dashboard_Wazuh.png)

---

### Navigation vers la section Agents

![Section Agents](images/Section_Agents.png)

![Ajout d'agent](images/Ajout_agent.png)

![Configuration agent](images/Configuration_agent.png)

---

### ⚠️ Note importante sur les adresses IP

**Il faut utiliser l'adresse IP privée du serveur Wazuh** car :

- Dans un **VPC AWS**, les instances dans le même VPC **ne communiquent pas via l'IP publique** par défaut
- Le serveur ne reçoit donc **jamais la demande d'enrôlement** avec l'IP publique
- C'est pourquoi le Dashboard montrait **0 agents** initialement

#### Correction effectuée :

![Correction IP privée](images/Correction_IP_privée.png)

![Commandes d'installation](images/Commandes_installation.png)
![Commandes d'installation](images/Commandes_installation2.png)


---

## 🐧 Enrôlement du client Linux

### 1. Connexion SSH au Linux-Client

![Connexion Linux-Client](images/Connexion_Linux-Client.png)

![Terminal Linux-Client](images/Terminal_Linux-Client.png)

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

![Installation agent Linux](images/Installation_agent_Linux.png)

✅ **Wazuh Agent installé avec succès sur Linux-Client**

---

#### Recharge de systemd

```bash
sudo systemctl daemon-reload
```

![Daemon reload](images/Daemon_reload.png)

---

#### Activation du service

```bash
sudo systemctl enable wazuh-agent
```

![Enable service](images/Enable_service.png)

---

#### Démarrage du service

```bash
sudo systemctl start wazuh-agent
```

![Start service](images/Start_service.png)

---

#### Vérification du statut

```bash
sudo systemctl status wazuh-agent
```

![Statut agent Linux](images/Statut_agent_Linux.png)

✅ **Agent Linux actif et connecté au serveur Wazuh**

---

### 3. Vérification dans le Dashboard

![Agent Linux dans Dashboard](images/Agent_Linux_Dashboard.png)

![Détails agent Linux](images/Détails_agent_Linux.png)

✅ **L'agent Linux-client apparaît bien dans le Dashboard Wazuh**

---

## 🪟 Enrôlement du client Windows

### 1. Connexion RDP au Windows-Client

#### Lancement de la connexion Bureau à distance

```
Commande : mstsc (Windows + R)
```

![Commande mstsc](images/Commande_mstsc.png)

---

#### Saisie de l'IP publique

**IP publique du Windows-Client** : `34.230.78.148`

![Connexion RDP](images/Connexion_RDP.png)

![Avertissement certificat](images/Avertissement_certificat.png)

---

### 2. Récupération du mot de passe

#### Téléchargement de la paire de clés

![Téléchargement clés Windows](images/Téléchargement_clés_Windows1.png)

---

#### Déchiffrement du mot de passe

![Décryption mot de passe](images/Téléchargement_clés_Windows.png)

![Mot de passe généré](images/Mot_de_passe_généré.png)

---

#### Authentification RDP

![Saisie mot de passe](images/Saisie_MDP.png)

![Chargement session](images/Chargement_session.png)

![Windows Server démarré](images/Windows_Server_démarré.png)

✅ **Connexion réussie au Windows Server**

---

### 3. Installation de l'agent Wazuh sur Windows

![Configuration agent Windows](images/Configuration_agent_Windows.png)
![Configuration agent Windows](images/Configuration_agent_Windows1.png)


![Commandes PowerShell](images/Commandes_PowerShel.png)

![Script d'installation](images/Script_installation.png)


---

#### Exécution dans PowerShell (Admin)

![PowerShell Admin](images/Installation_encours.png)

✅ **Agent Wazuh installé avec succès sur Windows-Client**

---

### 4. Vérification dans le Dashboard

![Agent Windows dans Dashboard](images/Agent_Windows_Dashboard.png)


✅ **Les deux agents (Linux et Windows) sont maintenant enrôlés et actifs**

---

## 🔍 Démonstrations SIEM et EDR <a name="demonstrations"></a>

### 🐧 Démo SIEM côté Linux

#### Scénario 1 : Tentatives SSH échouées (Brute Force simulé)

```bash
ssh fakeuser@172.31.26.179
```

![Tentatives SSH](images/Tentatives_SSH.png)

**Résultat dans Wazuh** :

![Alertes brute force SSH](images/Alertes_brute_SSH.png)

✅ **Wazuh détecte les tentatives d'authentification SSH échouées**

---

#### Scénario 2 : Élévation de privilèges

```bash
sudo su
```

![Élévation privilèges](images/Élévation_privilèges.png)

**Alerte Wazuh** :

![Alerte sudo](images/Alerte_sudo.png)

✅ **Wazuh détecte l'utilisation de sudo et l'élévation de privilèges**

---

#### Scénario 3 : Modification fichier sensible (FIM)

```bash
echo "test" | sudo tee -a /etc/passwd
```

![Modification /etc/passwd](images/Modification_etc_passwd.png)

**Alerte FIM (File Integrity Monitoring)** :

![Alerte FIM](images/Alerte_FIM.png)

✅ **Wazuh détecte la modification du fichier sensible /etc/passwd**

---

### 🪟 Démo EDR côté Windows

#### Scénario 1 : Échecs de login RDP (Event ID 4625)

**Action** : Tentatives de connexion RDP avec mauvais mot de passe (5 fois)

![Tentatives RDP échouées](images/Tentatives_RDP_échouées.png)

![Logon failed](images/Logon_failed.png)

**Alertes dans Wazuh** :


![Alertes échecs authentification](images/Alertes_échecs_authentification.png)

✅ **Wazuh détecte les tentatives d'authentification RDP échouées (Event ID 4625)**

---

#### Scénario 2 : Création d'un utilisateur local

```powershell
net user labuser P@ssw0rd! /add
net localgroup administrators labuser /add
```

![Création utilisateur](images/Création_utilisateur.png)

**Alerte Wazuh** :

![Alerte création utilisateur](images/Alerte_création_utilisateur.png)

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

![Installation Sysmon](images/Installation_Sysmon.png)

![Sysmon installé](images/Sysmon_installé.png)

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
