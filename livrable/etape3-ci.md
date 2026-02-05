# Étape 3 — Configuration CI (GitHub Actions)

## Objectif
Automatiser la construction, les tests et l’analyse qualité (SonarCloud) pour le back et le front.

---

## Pipeline mis en place
Fichier : `.github/workflows/ci.yml`

### Déclencheurs
- **push** sur `main`
- **pull_request** vers `main`
- **workflow_dispatch** (manuel)

---

## Job Back-end (Build & Test)
**Pourquoi** : valider que l’API compile et que les tests passent à chaque changement.

**Commandes CI** :
- `./gradlew build`

**Résultat attendu** :
- JAR construit + tests JUnit OK.

---

## Job Front-end (Build & Test)
**Pourquoi** : garantir la stabilité du front et éviter les régressions UI.

**Commandes CI** :
- `npm ci`
- `npm run build`
- `npm test -- --watch=false --browsers=ChromeHeadless`

---

## Job SonarCloud (Back)
**Pourquoi** : analyse statique de qualité, détection de bugs et vulnérabilités.

**Conditions d’exécution** :
- secrets GitHub renseignés :
  - `SONAR_TOKEN`
  - `SONAR_ORGANIZATION`
  - `SONAR_PROJECT_KEY_BACK`

---

## Job SonarCloud (Front)
**Pourquoi** : analyse statique du code Angular.

**Conditions d’exécution** :
- secrets GitHub renseignés :
  - `SONAR_TOKEN`
  - `SONAR_ORGANIZATION`
  - `SONAR_PROJECT_KEY_FRONT`

---

## Checklist Étape 3
- [x] Build back automatisé
- [x] Tests back automatisés
- [x] Build front automatisé
- [x] Tests front automatisés
- [x] Analyse SonarCloud prévue et conditionnelle

---

## Notes
- La CI utilise **Java 17** (cohérent avec le projet).
- La CI utilise **Node 20** (compatible Angular 17).
- L’analyse SonarCloud est conditionnelle pour éviter l’échec si les secrets ne sont pas configurés.
