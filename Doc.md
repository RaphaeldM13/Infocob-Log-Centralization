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
  ## 4.1 Déploiement Loki et Docker Driver

  Dans le cadre de ce projet, loki étant conteneurisé dans docker, l'installation par docker-compose est possible. Les autres moyens d'installation sont disponibles dans la documentation officielle de Grafana. Faites        cependant attention à utiliser les fichiers de configuration présents dans ce repository

  1. Créer un dossier loki et y entrer
     ```
     mkdir loki /
     cd loki
     ```
  2. installer le client docker driver
     ```
     docker plugin install grafana/loki-docker-driver:3.7.0-arm64 --alias loki --grant-all-permissions
     ```
     pour vérifier l'état du plugin, utiliser
     ```
     docker plugin ls
     ```
     si le plugin est disabled
     ```
     docker plugin enable loki
     systemctl restart docker
     ```

  4. Placez dans le dossier loki les fichiers docker-compose.yaml, alloy-local-config.yaml, et loki-config.yaml présent dans le dossier Config de ce repository.
     ```
     wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/docker-compose.yaml -O docker-compose.yaml
     wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/alloy-local-config.yaml -O alloy-local-config.yaml
     wget https://raw.githubusercontent.com/RaphaeldM13/Infocob-Log-Centralization/refs/heads/main/Config/loki-config.yaml -O loki-config.yaml
     ```
  5. Lancer le Docker
     ```
     docker-compose -f docker-compose.yaml up
     ```
  6. Vérifications
     pour vérifier que Loki tourne correctement, accédez à
     ```
     http://localhost:3101/ready
     ```

## 4.2 Déploiement Alloy (détaillé)
```
  là tu mets :

  config
  pipeline
  labels
  debug
```
# 5. Pipeline de logs
```
  centré Alloy mais mention Loki
```
# 6. Problèmes rencontrés
```
  surtout Alloy + intégration Loki
```
# 7. Guide de déploiement (IMPORTANT)
```
  structuré comme ça :
```
## Étape 1 : lancer Loki
## Étape 2 : config Alloy
## Étape 3 : vérifier ingestion
