# Étape 1 — Analyse du projet (résumé complet)

## Objectif
Comprendre la structure du projet, identifier les commandes de build/test et vérifier concrètement les versions et le bon fonctionnement.

---

## 1) Structure du projet
- **Back-end** : Spring Boot (Java) dans `back/`
- **Front-end** : Angular (TypeScript) dans `front/`

---

## 2) Commandes back-end (PowerShell)

### Build
```
cd "C:\Users\Utilisateur\projet VS code\P7-FSJA\back"
.\gradlew.bat build
```
**Pourquoi** : compile le code, exécute les tests et produit un JAR.
**Ce que ça fait** :
- télécharge Gradle si besoin,
- compile le projet,
- exécute les tests,
- génère `build/libs/microcrm-0.0.1-SNAPSHOT.jar`.

**Résultat** : ✅ Build OK, tests OK (2 tests)

### Tests seuls
```
.\gradlew.bat test
```
**Pourquoi** : lancer uniquement les tests sans reconstruire l’ensemble.
**Résultat** : ✅ Tests OK

---

## 3) Commandes front-end (PowerShell)

### Installer les dépendances
```
cd "C:\Users\Utilisateur\projet VS code\P7-FSJA\front"
npm install
```
**Pourquoi** : installer les packages nécessaires au build et aux tests.
**Résultat** : ✅ OK (npm a signalé 58 vulnérabilités — à traiter plus tard dans le plan sécurité)

### Build
```
npm run build
```
**Pourquoi** : générer l’application front optimisée pour la production.
**Résultat** : ✅ Build OK (sortie dans `front/dist/microcrm`)

### Tests (mode CI)
```
npm test -- --watch=false
```
**Pourquoi** : exécuter les tests une seule fois (mode CI, pas de watch).
**Résultat** : ✅ 24 tests OK

---

## 4) Versions vérifiées
- **Gradle wrapper** : 8.7
- **Java cible du projet** : 17 (sourceCompatibility)
- **Java utilisée par Gradle** : 21.0.9 (Eclipse Adoptium)
- **Node** : 22.21.1
- **npm** : 10.9.4
- **Angular** : 17.3.x

---

## 5) Contraintes techniques relevées
- Tests front nécessitent Chrome/Chromium (Karma).
- Back utilise HSQLDB embarquée (pas de base externe à configurer).

---

## Checklist Étape 1
- [x] Code back/front localisé
- [x] Commandes build/test back connues et testées
- [x] Commandes build/test front connues et testées
- [x] Versions Java/Node/Angular/Gradle notées
