# Étape 4 — Conteneurisation & préparation du déploiement

## Objectif
Vérifier/adapter les Dockerfiles et orchestrer le front + back avec docker-compose.

---

## Vérifications et adaptations réalisées

### Dockerfile (racine)
**Modifications** :
- Image Node **pinnée** (`node:20-alpine`) pour un build front reproductible.
- Build Angular via `npm run build` (commande officielle du projet).
- Runtime back aligné sur **Java 17** (`openjdk17-jre-headless`).
- Port back corrigé à **8080** (au lieu de 4200).

**Pourquoi** :
- cohérence avec la version Java cible,
- éviter les surprises liées à une image `node` non versionnée,
- respecter le port réel du back.

---

## Orchestration docker-compose
**Fichier créé** : `docker-compose.yml`

### Services
- **back** : build target `back`, expose `8080:8080`
- **front** : build target `front`, expose `80:80` et `443:443`

**Pourquoi** :
- démarrer l’application complète en une commande,
- isoler front/back tout en gardant un réseau commun Docker.

---

## Commandes utiles

### Construire et lancer
```
docker compose up --build
```

### Arrêter
```
docker compose down
```

---

## Checklist Étape 4
- [x] Dockerfiles vérifiés et adaptés
- [x] docker-compose.yml créé
- [x] Ports cohérents (front 80/443, back 8080)
