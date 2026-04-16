# 1. Introduction

  Le système présenté dans cette documentation permet la collecte et la centralisation de logs provenant de machines Apache, Linux, Windows et Docker.
  Les logs sont collectés par l’agent Grafana Alloy, qui assure leur traitement et leur enrichissement, avant de les transmettre à une plateforme de centralisation basée sur Grafana Loki.
  Cette dernière est déployée sous forme de conteneur Docker et utilise MinIO comme solution de stockage objet.

# 2. Architecture globale

  Le système repose sur une architecture de collecte distribuée des logs.

  Chaque machine à superviser (serveur Apache, système Linux/Windows) embarque un agent Grafana Alloy chargé de collecter et transmettre les logs. Alloy assure la lecture des journaux locaux, leur enrichissement (labels)    et leur envoi vers une plateforme centralisée.

  Les logs sont ensuite envoyés via HTTP vers Grafana Loki, déployé dans un conteneur Docker. Loki centralise et stocke les données, en s’appuyant sur MinIO pour le stockage objet.
  Le flux de données est le suivant :
    ```
    Sources de logs → Alloy → Loki → MinIO
    ```
  Cette architecture permet une solution scalable, flexible et facilement déployable sur plusieurs machines.

# 3. Présentation des composants
  ## Alloy
  Grafana Alloy est un agent de collecte de télémétrie permettant de récupérer, traiter et transmettre des données telles que des logs, métriques ou traces. Dans ce projet, il est utilisé comme agent de collecte de logs     déployé sur chaque machine à superviser.

  Alloy permet de lire différentes sources de logs (fichiers système, journaux applicatifs, conteneurs Docker), puis d’appliquer des traitements tels que le parsing et l’ajout de labels, avant d’envoyer les données vers     une plateforme centralisée comme Grafana Loki.

  Dans ce projet, Grafana Alloy agit comme agent de collecte de logs déployé sur chaque machine à superviser.
  Il est chargé de récupérer les logs locaux (notamment ceux du serveur Apache), de les enrichir (ajout de labels) puis de les transmettre à Grafana Loki pour leur centralisation et leur exploitation.
  Alloy constitue ainsi l’élément intermédiaire essentiel entre les sources de logs et la plateforme de stockage.

  L’utilisation de Grafana Alloy est particulièrement pertinente dans ce projet pour plusieurs raisons :

  - <ins>Flexibilité</ins> : prise en charge de multiples sources de logs (Apache, système, Docker)
  - <ins>Centralisation simplifiée</ins> : envoi direct vers Loki via HTTP
  - <ins>Architecture distribuée</ins> : un agent par machine, facilitant le passage à l’échelle
  - <ins>Capacités de traitement</ins> : enrichissement et structuration des logs avant ingestion

  Ainsi, Alloy constitue un composant clé permettant d’unifier et de standardiser la collecte des logs au sein de l’infrastructure.
  ## Loki
  Grafana Loki est le composant central du système de centralisation des logs. Il est responsable de la réception, du stockage et de la mise à disposition des logs provenant des différentes machines supervisées via les      agents Grafana Alloy.

  Les logs sont envoyés à Loki via une API HTTP d’ingestion exposée sur l’endpoint suivant :
    ```
    http://<IP_LOKI_HOST>:3100/loki/api/v1/push
    ```
  Loki ne se contente pas de stocker les logs, il les indexe principalement via leurs labels, ce qui permet des requêtes rapides et efficaces dans Grafana sans nécessiter une indexation complète du contenu.

  Dans ce projet, Loki est déployé sous forme de conteneur Docker, ce qui simplifie son installation, sa portabilité et son intégration dans l’infrastructure existante. Il constitue ainsi le point de convergence de tous     les flux de logs issus des différents agents Alloy.

# 4. Mise en place
  ## 4.1 Déploiement de Grafana Loki et du Docker Driver
  Dans le cadre de ce projet, Grafana Loki est déployé sous forme de conteneur Docker via Docker Compose, afin de simplifier son installation et assurer sa portabilité.

  Les autres méthodes d’installation sont disponibles dans la [documentation officielle de Grafana](https://grafana.com/docs/loki/latest/setup/install/docker/). Toutefois, il est important d’utiliser les fichiers de configuration fournis dans ce dépôt afin de garantir la compatibilité avec l’architecture mise en place.

  1. Création de l’environnement de travail
  Créer un répertoire dédié à Loki puis s’y placer :
  ```
  mkdir loki
  cd loki
  ```
  2. Installation du Docker Logging Driver (Loki)
  Installer le plugin Docker permettant la redirection des logs vers Loki :
  ```
  docker plugin install grafana/loki-docker-driver:3.7.0-arm64 --alias loki --grant-all-permissions
  ```
  Vérifier son installation :
  ```
  docker plugin ls
  ```
  En cas de désactivation du plugin :
  ```
  docker plugin enable loki
  systemctl restart docker
  ```
  3. Récupération des fichiers de configuration

  Les fichiers nécessaires au fonctionnement du système ([Docker Compose](Config/docker-compose.yaml), [Alloy](Config/alloy-local-config.yaml) et [Loki](Config/loki-config.yaml) doivent être placés dans le répertoire loki.
  Ils peuvent être récupérés directement depuis ce dépôt :
  ```
  wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/docker-compose.yaml -O docker-compose.yaml
  wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/alloy-local-config.yaml -O alloy-local-config.yaml
  wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/loki-config.yaml -O loki-config.yaml
  ```
  4. Démarrage de la stack
  Lancer l’ensemble des services via Docker Compose :
  ```
  docker-compose -f docker-compose.yaml up -d
  ```
  5. Vérification du fonctionnement

  Une fois les services démarrés, vérifier que Loki est opérationnel en accédant à l’endpoint de santé :
  ```
  http://localhost:3101/ready
  ```
  Une réponse ready confirme le bon fonctionnement du service.

## 4.2 Déploiement de Grafana Alloy

Grafana Alloy doit être déployé sur chaque machine source de logs. Il agit comme un agent chargé de collecter, traiter et transmettre les logs vers Grafana Loki.

### Installation d’Alloy

  L’installation d’Alloy dépend du système cible. Il est recommandé de suivre la documentation officielle de Grafana :

  - Linux : https://grafana.com/docs/alloy/latest/set-up/install/linux/
    Vous pouvez installer Alloy en tant que service `systemd` sur Linux.

    ### Avant de commencer
    Certaines machines virtuelles cloud basées sur Debian ne disposent pas de GPG installé par défaut.
    Pour installer GPG sur votre machine virtuelle Linux, exécutez la commande suivante dans un terminal.
    ```
    sudo apt install gpg
    ```
    ### Installation
    Pour installer Alloy sur Linux, exécutez les commandes suivantes dans un terminal.
    1. Importer la clé GPG et ajouter le dépôt Grafana
    ```
    sudo mkdir -p /etc/apt/keyrings
    sudo wget -O /etc/apt/keyrings/grafana.asc https://apt.grafana.com/gpg-full.key
    sudo chmod 644 /etc/apt/keyrings/grafana.asc
    echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
    ```
    2. Update the repositories.
    ```
    sudo apt-get update
    ```
    3. Installer Alloy
    ```
    sudo apt-get install alloy
    ```
    
  - Docker : https://grafana.com/docs/alloy/latest/set-up/install/docker/
    
  - Windows : https://grafana.com/docs/alloy/latest/set-up/install/windows/
    Vous pouvez installer Alloy sur Windows via une installation graphique standard, avec WinGet, ou en mode silencieux.
    ### Installation graphique standard

    Pour effectuer une installation graphique standard d’Alloy sur Windows, suivez les étapes suivantes :

    1. Accédez à la page des releases sur GitHub.
    2. Faites défiler jusqu’à la section Assets.
    3. Téléchargez `alloy-installer-windows-amd64.exe` ou téléchargez et extrayez `alloy-installer-windows-amd64.exe.zip`.
    4. Double-cliquez sur `alloy-installer-windows-amd64.exe` pour installer Alloy.

    L’installateur place Alloy dans le répertoire par défaut :
    `%PROGRAMFILES%\GrafanaLabs\Alloy`
    ### Installation avec WinGet

    Pour installer Alloy avec WinGet, suivez les étapes suivantes :

    1. Assurez-vous que le gestionnaire de paquets WinGet est installé.
    2. Exécutez la commande suivante.
       ```
       winget install GrafanaLabs.Alloy
       ```
    
    ### Installation silencieuse

    Pour effectuer une installation silencieuse d’Alloy sur Windows, suivez les étapes suivantes :

    1. Accédez à la page des releases sur GitHub.
    2. Faites défiler jusqu’à la section Assets.
    3. Téléchargez `alloy-installer-windows-amd64.exe` ou téléchargez et extrayez `alloy-installer-windows-amd64.exe.zip`.
    4. Exécutez la commande suivante en tant qu’administrateur.
       ```
       <PATH>\alloy-installer-windows-amd64.exe /S
       ```
       Remplacer les éléments suivants :
       <PATH_TO_INSTALLER> : le chemin vers l’exécutable de l’installateur décompressé.
       
    ### Options d’installation silencieuse
    - `/CONFIG=<path>` : chemin vers le fichier de configuration.
      Par défaut : `$INSTDIR\config.alloy`
    - `/DISABLEREPORTING=<yes|no>` : désactive la collecte de données.
      Par défaut : `no`
    - `/DISABLEPROFILING=<yes|no>` : désactive le point de terminaison de profiling.
      Par défaut : `no`
    - `/ENVIRONMENT="KEY=VALUE\0KEY2=VALUE2"` : définit les variables d’environnement pour le service Windows.
      Par défaut : ` `
    - `/FORCEREGISTRY=yes` : au début de l’installation, supprime toutes les clés de registre Alloy, puis lance l’installation normalement afin que les options comme `/STABILITY` et `/DISABLEREPORTING` soient réécrites          proprement.
      Par défaut, l’installateur conserve les valeurs existantes lors d’une mise à jour. Utilisez cette option pour les réinitialiser et supprimer les anciennes clés inutilisées.
    - `/RUNTIMEPRIORITY="normal|below_normal|above_normal|high|idle|realtime"` : définit la priorité d’exécution du processus Alloy.
      Par défaut : `normal`
    - `/STABILITY="generally-available|public-preview|experimental"` : définit le niveau de stabilité d’Alloy.
      Par défaut : `generally-available`
    - `/USERNAME="<username>"` : définit l’utilisateur Windows utilisé pour exécuter le service.
      Par défaut : `NT AUTHORITY\LocalSystem`
    - `/PASSWORD="<password>"` : définit le mot de passe de l’utilisateur utilisé pour exécuter le service.
      Non requis pour les comptes standards comme LocalSystem.
      Par défaut : ` `
    ### Configuration d’Alloy

Une fois Alloy installé, il doit être configuré à l’aide du fichier config.alloy fourni dans ce dépôt.

- Linux
  Le fichier de configuration peut être récupéré directement via la commande suivante :
  ```
  sudo curl -o /etc/alloy/config.alloy https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/config.alloy
  ```
- Windows
  Télécharger le fichier de configuration config.alloy depuis le dépôt, puis le placer dans le répertoire d’installation d’Alloy, généralement :
  ```
  %PROGRAMFILES%\GrafanaLabs\Alloy\
  ```
- docker

  Le fichier récupéré doit ensuite être modifié pour correspondre à la machine :
    - Adaptation des labels à l’environnement cible

    Les valeurs `job` et `hostname` doivent être adaptées à chaque machine afin d’identifier précisément la source des logs dans Loki.
    ```
    sudo nano /etc/alloy/config.alloy
    ```

    ```hcl
    loki.process "apache_labels" {
      forward_to = [loki.write.loki.receiver]

      stage.static_labels {
        values = {
          job  = "<MACHINE_TYPE>",  # ex : apache, linux, windows
          hostname = "<MACHINE_NAME>",  # nom ou identifiant de la machine
        }
      }
    }
    ```
    - Rechargement de Alloy
      ```
      sudo systemctl reload alloy
      ```
  
# 5. Pipeline de logs
  Le pipeline de logs décrit le cheminement des données depuis leur génération jusqu’à leur stockage.

  Dans cette architecture, le traitement des logs s’effectue en plusieurs étapes :

  ### Génération des logs
  Les logs sont produits par les différentes sources, notamment le serveur Apache (logs d’accès et d’erreurs) ainsi que les systèmes hôtes.
  
  ### Collecte par Grafana Alloy
  Alloy lit les fichiers de logs présents sur la machine (ex : /var/log/apache2/access.log) et détecte les nouvelles entrées en temps réel.
  
  ###Traitement et enrichissement
  Les logs sont ensuite traités par Alloy, qui peut :
    - ajouter des labels (ex : job=apache, host=...)
    - structurer certaines informations
    - préparer les données pour leur ingestion
    
  ### Transmission vers Loki
  Les logs sont envoyés via HTTP à Grafana Loki à l’aide de son endpoint d’ingestion.
  
  ### Stockage et indexation
  Loki stocke les logs en s’appuyant principalement sur leurs labels, permettant une recherche rapide et efficace.
  
# 6. Problèmes rencontrés
# 7. Guide de déploiement

  Ce guide résume les étapes essentielles pour déployer rapidement la solution de centralisation des logs.

  ## Étape 1 : Déployer Grafana Loki
  ### Créer un dossier dédié :
  ```
  mkdir loki
  cd loki
  ```
  ### Récupérer les fichiers de configuration :
  ```
  wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/docker-compose.yaml -O docker-compose.yaml
  wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/loki-config.yaml -O loki-config.yaml
  wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/alloy-local-config.yaml -O alloy-local-config.yaml
  ```
  ### Installer le driver Docker Loki :
  ```
  docker plugin install grafana/loki-docker-driver:3.7.0-arm64 --alias loki --grant-all-permissions
  ```
  ### Lancer la stack :
  ```
  docker-compose up -d
  ```
  ### Vérifier que Loki fonctionne :
  ```
  http://localhost:3101/ready
  ```

  ## Étape 2 : Déployer et configurer Grafana Alloy
  ### Installer Alloy sur la machine source (Linux / Windows / Docker)
  Télécharger la configuration :
  ```
  sudo curl -o /etc/alloy/config.alloy https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/config.alloy
  ```
  ### Adapter les labels à la machine

  Chaque instance Alloy doit être personnalisée afin d’identifier clairement la source des logs dans Loki :
    ```
    sudo nano /etc/alloy/config.alloy
    ```
  ```hcl
  loki.process "apache_labels" {
    forward_to = [loki.write.loki.receiver]

    stage.static_labels {
      values = {
        job  = "<MACHINE_TYPE>",  # ex : apache, linux, windows
        host = "<MACHINE_NAME>",  # identifiant de la machine
      }
    }
  }
  ```

  Ces labels permettent de filtrer efficacement les logs dans Grafana Loki (par machine et par type de service)

  Démarrer Alloy :
  ```
  sudo systemctl start alloy
  ```

  ## Étape 3 : Vérifier l’ingestion des logs
  - Générer du trafic (ex : requête HTTP sur Apache)
  - Vérifier dans Loki (via Grafana ou API) :
  - http://<IP_LOKI_HOST>:3100/loki/api/v1/query
  - Exemple de requête :
    ```
      {job="apache"}
    ```
