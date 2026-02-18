| <img src="./img/logo_enset.png" width="150"> | <center width="500">**UNIVERSITÉ HASSAN II DE CASABLANCA**<br> <br> ENSET MOHAMMEDIA<br> <br> *DEPARTEMENT DE MATHEMATIQUES INFORMATIQUE*</center> | <img src="./img/arabe.png" width="150"> |
| :--- | :---: | ---: |
</center>



<center>

# 📊 Mise en œuvre d'une infrastructure cloud de supervision centralisée sous AWS
## Déploiement de Zabbix conteneurisé pour le monitoring d'un parc hybride (Linux & Windows)

</center>

<center> <img src ="./img/aws.png" width="150" >  <img src ="./img/zabbix.png" width="150">  </center>

## 🎓 Informations Générales :
* **Filière :** Ingénierie Informatique Big Data Cloud Computing (IIBDCC)
* **Module :** Sécurité des SI & Cyber Sécurité
* **Réalisé par :** TSEH Kokou Benoît
* **Encadré par :** Prof. Azeddine KHIAT
* **Année Universitaire :** 2025-2026

---

## 📝 1. Introduction
Dans le cadre de la gestion moderne des infrastructures informatiques, la surveillance (monitoring) est devenue un pilier essentiel pour garantir la haute disponibilité et la performance des services. Ce projet consiste en la **mise en œuvre d'une infrastructure de supervision centralisée** hébergée sur le cloud **Amazon Web Services (AWS)**.

L'objectif principal est de déployer une solution conteneurisée à l'aide de **Docker** pour assurer le monitoring en temps réel d'un parc informatique hybride, composé d'instances **Linux** et **Windows**. Le choix d'une architecture conteneurisée permet une plus grande flexibilité de déploiement et une gestion simplifiée des dépendances du serveur Zabbix.

Le projet s'articule autour de trois axes majeurs :


- **Configuration de l'infrastructure Cloud :** Mise en place d'un VPC, de sous-réseaux et de groupes de sécurité adaptés pour autoriser les flux de monitoring (ports 10050/10051) et l'accès web (ports 80/443).

- **Déploiement du serveur Zabbix :** Installation via Docker-Compose sur une instance Ubuntu (t3.large).
- **Supervision du parc :** Installation et configuration des agents Zabbix sur des clients Ubuntu et Windows Server pour la remontée de métriques CPU, RAM et réseau.
    

**Axes majeurs :**
* Déploiement sur AWS (EC2 & VPC).
* Orchestration des services via Docker-Compose.
* Supervision d'instances Linux (Ubuntu) et Windows Server.

---
## 🌐 Architecture globale du projet 
![architectue globale du  projet](./img/architecture.png)


## ☁️ 2. Architecture Réseau
L'infrastructure réseau est isolée au sein d'un VPC   spécifique (Région : `us-east-1`) avec une configuration stricte des flux de sécurité via les **Security Groups**.

- Pour  Ce faire , nous nous connectons au console de **AWS** dans la regions de  `us-east-1` où nous créons les  un **VPC**  avec le **CDIR Bloc: 10.0.0.0/16** que nous allons  nommer : **VPC_Projet_Zabbix**  

![creation vpc](./img/vpc_creation.png)

- Nous créons dans Ce réseau virtuel, un  sous-réseau **Subnet_VPC_Projet_Zabbix** avec le **CDIR Bloc: 10.0.0.0/24**
![création vpc](./img/creationsubnet.png)

- Ensuite nous créons deux Sécurité  groupe pour   pour filter l'accès aux instances: 
   - **Le premier est : Zabbix-Server-SG** :

     il a les règle suivante : 
        
        - **Port 22** : pour le protocole **SSH** permet de se connecter à l'instance  sur laquelle tourne Zabbix à distance .
        - **Ports 80 et 443** : pour le protocole **HTTP/HTTPS**, utilié  ppour l'affichage de l'interface web de **Zabbix**.
        - **Port 10051** : **Protocle TCP** , ce port est utilisé par Zabbix pour recevoir un les informations des agents.

       
         ![creation du croupe de securité zabbix](./img/zabbix-server-sg.png)
 

   - **Le deuxième est : Agents-SG** 

     - **Le port 10050**: TCP	Écoute de l'agent (Passive Mode)
     - **Le port 3389** :	RDP	Accès distant Windows
     - **le port 22** :  	SSH	Administration Linux à distance


     ![creation du groupe de sercurité agent](./img/agent-sg.png)


- Nous allons créer une Passerelle poour permmetre au ressouce de pourvoir accéder à l'internet. Pour cela nous créeons Internet gateway:
 ![internet gateway créations](./img/igw_zabbix.png)
 
 -  On ajoute une route :

   ![creation de route](./img/route.png)
     

    
    

<img src="./img/architecture1.png"></img>
<img src="./img/architecture2.png"></img>

*(Figure 1 : Schéma de l'infrastructure Cloud AWS et des Security Groups)*





---

## 🖥️ 3. Architecture des Instances EC2
| Rôle | Type d'instance | OS | Usage |
| :--- | :--- | :--- | :--- |
| **Serveur Zabbix** | t3.large | Ubuntu 22.04 LTS | Serveur Docker & Dashboard |
| **Client Linux** | t3.medium | Ubuntu 22.04 LTS | Monitoring Agent Linux |
| **Client Windows** | t3.large | Windows Server 2022 | Monitoring Agent Windows |

![Instances EC2 Running](./img/instance-ec2.png)
*(Figure 2 : Capture d'écran des instances EC2 en état 'Running' dans la console AWS)*

---

## 🚀 4. Déploiement du Serveur Zabbix (Docker)
Nous Connectons à l'instance **Zabbix-server**  afin de deployer Le serveur zabbix. Comme nous volous faire un deploiement via  conteneur docker, nous allons installer dokcer engine  et préparer par après un fichier *docker-compose.yml* :
- Nous installons  **docker engine** via le commande( nous installons aussi docker-compose pour faciliter  le deploiement):

```bash
sudo apt update && sudo apt install docker.io docker-compose -y
```

![installation de doc](./img/installationdocker.png)
![verification installation](./img/verificationInstallation.png)

On a notre fichier [docker-compose.yml](./docker-compose.yml) avec les service  de **zabbix serveur** , **zabbix web**  avec une base de données MySQL persistante. Pour les questios de sécurité nous avons crée un  fichier `.env` pour les varible sensiblie: 

**`docker-compose.yaml`**
```yaml
services:
  zabbix-db:
    image: mysql:8.0
    container_name: zabbix-db
    restart: always
    command: --character-set-server=utf8 --collation-server=utf8_bin --default-authentication-plugin=mysql_native_password
    --log_bin_trust_function_creators=1
    environment:
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
    volumes:
      - ./zabbix-db-data:/var/lib/mysql

  zabbix-server:
    image: zabbix/zabbix-server-mysql:ubuntu-6.4-latest
    container_name: zabbix-server
    restart: always
    ports:
      - "10051:10051"
    environment:
      - DB_SERVER_HOST=zabbix-db
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
    depends_on:
      - zabbix-db

  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:ubuntu-6.4-latest
    container_name: zabbix-web
    restart: always
    ports:
      - "80:8080"
    environment:
      - ZBX_SERVER_HOST=zabbix-server
      - DB_SERVER_HOST=zabbix-db
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - PHP_TZ=${ZBX_PHP_TZ}
    depends_on:
      - zabbix-db
      - zabbix-server

volumes:
  zabbix-db-data:
```

**.env :**

```.conf

# Base de données
MYSQL_ROOT_PASSWORD=
MYSQL_USER=
MYSQL_PASSWORD=
MYSQL_DATABASE=

# Configuration Zabbix
ZBX_PHP_TZ=

```

Avec la commande `docker-compose up -d` on  on deploie le server zabbix:
![deploiement](./img/deploiementzabbix.png)
![verification deploiement](./img/verificationconteneur.png)

On peut voir l'interface de  comme suit :
![zabbix](./img/interfaceZabbix.png)


---
## 5. Configuration des agents :

### - Configuration de l'agent  Linxu-client :

 Pour configurer cet agent , on va executer  onon télécharge et on installe Linux agent sur l'instance **Linux-client-zabbix**  on executer les commandes suivantes :
 
 ```bash
 # Téléchargement des paquet
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb

# Installation du  dépôt
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb

# Mise à jour la liste des paquets
sudo apt update
 ```


![installation](./img/agentlinuxIntallation.png)

**Installation**
```bash

sudo apt install zabbix-agent -y
```

![installation](./img/installation2.png)

Après  on fait la     change le fichier de configuration : 

```bash

sudo nano /etc/zabbix/zabbix_agentd.conf
```

on modifie les variable suivante :

```bash
Server=Ip du serveur

ServerActive=ip du serveur

Hostname=Nom_De_Cette_Instance
```

Et on redemarre le service :

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```
![image](./img/agentRestartzabbix.png)


On se connecter a l'interface de  Zabbix pour faire   complèter la configuration :
![config](./img/creationClientConfig.png)

![linux client](./img/interfaceMoniterLinuxClient.png)

### - Configuration d l'agent Windows-client :

Nous telechargeons l'angents sur le client windows a l'adresse : [liens vers le  l'angents  zabbix windows](https://www.zabbix.com/download_agents)


on passe en suite la 'installtion 

- installation :
  ![installation de windows](./img/zabbixWindowsInstallation.png)


-  On verifie la connectivité :
  ![connectivité](./img/ConnnectiviteWindowsClient.png)

- Dans l'interface de client zabbix on ajoute le nouveau client en créant un host
  ![creation de hoste](./img/creationWindowsHost.png)

 - verifiaction de l'envoi des données :
  ![ confirmation de la reception des données](./img/InterfaceWindowDonneerecu.png)

---

# Monitoring et Validation

## -Création de dashboard  Global :
Nous créons un Dashboard  global pour  affcichier  les metrics de nos clients.

- Nous allons dans Dashboards  on clique sur  sur **Create dashboard**, On e nomme **Global Infrastructure Monitoring** :

    - Le graphique CPU :

        ->  Add widget et  on choisit le type Graph.

        -> Dans le champ Item patterns, on cherche  **CPU utilization**.

        -> Dans Host patterns, nous selectionnons la fois **Linux-client** et **Windows-client.**
        
        ![creation graph cpu](./img/creationCPUMonoitoring.png)
       

    - Graph de RAM :

         -> on ajoute un widget de type graph.

         -> On selectionne l'item Memory utilization.
        
        ![création graph ram](./img/creationMonitorigRAM.png)
    On sauvegarde 


## - Mise en place d'un Trigger (Alerte Proactive)

Cela prouve que ton système est capable d'auto-surveillance. Nous allons simuler une alerte de charge CPU élevée.

   -  Dans **Data Collection > Hosts**

   - On choisi un  hôte (ex: Linux-client).

   - on  Clique sur Create trigger en haut à droite.

   - Configuration :

       - Name : High CPU utilization on {HOST.NAME}.

       - Severity : Choisissez High (Rouge).

       - Expression : Clique sur Add, on cherche l'item CPU utilization, on choisit la fonction last() et mets > 10.
       - Cliquez sur Add.

       ![ttigger](./img/creationTriggers.png)
   

 On  peut voir un Dashboard Global qui est la fusion ici :
 ![dashboard gloabl](./img/dashborglobal.png)




---

# Conclusion

Le déploiement de cette infrastructure de monitoring hybride sur AWS constitue une démonstration concrète de l'interopérabilité entre les technologies Cloud, la conteneurisation et l'administration système multiplateforme.

## 🎯 Bilan technique et objectifs atteints
L'objectif principal, qui était de centraliser la surveillance d'un parc hétérogène (**Linux et Windows**) au sein d'un environnement réseau sécurisé, a été pleinement atteint. L'utilisation de **Zabbix 7.0 sous Docker** a permis de garantir une séparation stricte des services (Serveur, Base de données, Interface Web) tout en assurant une portabilité maximale de la solution.

## 🛠️ Analyse des défis et solutions apportées
Le projet a présenté plusieurs défis techniques majeurs qui ont nécessité une analyse approfondie du fonctionnement des réseaux AWS :

* **Gestion de l'adressage dynamique** : L'absence d'adresses IP publiques statiques sur les clients a été contournée avec succès par l'implémentation du mode **Active Agent**. Cette approche a permis de maintenir la remontée de métriques sans exposer les instances inutilement sur Internet.
* **Sécurisation des flux** : La configuration granulaire des *Security Groups* et du *Windows Advanced Firewall* a permis de respecter le principe du moindre privilège tout en assurant une connectivité robuste sur les ports `10050` et `10051`.
* **Connectivité hybride** : L'utilisation temporaire d'adresses IP Élastiques a démontré une compréhension du cycle de vie des ressources Cloud pour les phases de maintenance et de provisionnement.

## 🚀 Perspectives d'évolution
Cette architecture pose les jalons d'une supervision plus avancée. À l'avenir, le projet pourrait être valorisé par :

1.  **L'intégration de Grafana** pour créer des visualisations de données de niveau "Business Intelligence".
2.  **La mise en place d'un système d'alerte externe** (via Telegram ou Slack) pour notifier les administrateurs en temps réel.
3.  **L'automatisation du déploiement** des agents via des outils d'Infrastructure as Code (IaC) comme **Ansible** ou **Terraform**.

---
> **Bilan final :** Ce projet m'a permis de consolider mes compétences en gestion de réseaux complexes et de confirmer l'efficacité de Zabbix comme outil de référence pour la supervision d'infrastructures modernes.








