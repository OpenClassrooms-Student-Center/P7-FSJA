# Étape 2 — Plans requis (testing, sécurité, conteneurisation/déploiement)

## Objectif
Définir les règles et objectifs avant toute mise en œuvre technique (CI/CD).

---

## 1) Plan de testing périodique

### Décisions
- **Back-end** : tests unitaires + intégration (Gradle/JUnit).
- **Front-end** : tests unitaires Angular (Karma/Jasmine).

### Quand exécuter
- **À chaque push** sur `main`.
- **À chaque pull request** vers `main`.
- **Optionnel** : exécution quotidienne (nightly) via cron.

### Pourquoi
- **Push/PR** : prévenir les régressions avant fusion.
- **Nightly** : détecter les problèmes liés aux dépendances ou aux environnements.

---

## 2) Plan de sécurité

### Décisions
- **Analyse statique** : SonarQube Cloud.
- **Bonnes pratiques CI** : secrets gérés via GitHub Secrets, pas de credentials en clair.

### Quand exécuter
- **À chaque push et PR**.

### Problèmes surveillés
- vulnérabilités,
- bugs,
- code smells,
- dette technique.

### Points d’attention
- dépendances front : audit npm (vulnérabilités déjà détectées),
- dépendances back : surveiller les versions et CVE.

---

## 3) Principes de conteneurisation & déploiement

### Conteneurisation
- **Back-end** : image Java 17 officielle et légère (slim).
- **Front-end** : build Node + serveur web léger pour servir les fichiers statiques.

### Orchestration
- **docker-compose** pour lancer front + back ensemble.

### Déploiement (préparation)
- stratégie de **publication d’images Docker** (ex: registre GitHub Container Registry),
- déclenchement après succès de la CI.

---

## Checklist Étape 2
- [x] Plan de testing périodique rédigé
- [x] Plan de sécurité rédigé
- [x] Principes de conteneurisation/déploiement rédigés
