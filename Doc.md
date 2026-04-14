# 1. Introduction
```
  objectif : centralisation de logs
```
# 2. Architecture globale
```
  tu présentes :

  Apache → source
  Alloy → agent (focus)
  Loki → backend
```
# 3. Présentation des composants
## Alloy (détaillé)
```
  grosse partie
```
## Loki (léger)
```
  juste :

  rôle
  endpoint
  docker rapide
```
# 4. Mise en place
## 4.1 Déploiement Loki (rapide)
```
  Exemple :

  docker-compose
  port 3100
  endpoint

  1 page max
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
