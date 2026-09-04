# Audit initial du dépôt MicroCRM

**Projet 7 — Mettez en œuvre l'intégration et le déploiement continu d'une application Full-Stack (Option B)**

| | |
|---|---|
| Auteur | Clément Cirou |
| Date | 4 septembre 2026 |
| Version | 1.0 |
| Dépôt analysé | `clems2/P7-CICD-Full-Stack` (fork de `OpenClassrooms-Student-Center/P7-FSJA`) |
| Commit de référence | `323c2e1` |
| Statut | État initial, avant toute industrialisation |

---

## 1. Objet et méthode

Ce document restitue l'analyse préalable exigée par l'étape 1 des consignes. Il poursuit trois
objectifs : établir une **photographie mesurée** de l'état initial du projet, **hiérarchiser** les
constats selon leur criticité, et **anticiper** leurs conséquences sur les livrables des parties 1
et 2.

La méthode a combiné quatre approches complémentaires :

1. **Lecture du code source** — descripteurs de build, configuration applicative, Dockerfile,
   configuration des tests.
2. **Exécution locale mesurée** — build, tests et démarrage du back-end et du front-end, avec
   relevé systématique des durées et des volumétries.
3. **Exécution conteneurisée** — construction des images, démarrage des conteneurs, vérification
   de la communication front/back depuis un navigateur.
4. **Veille technologique** — statut de support des composants, versions courantes des actions
   GitHub, patterns de pipelines comparables.

Les mesures constituent la **baseline** du projet. Elles serviront de point de comparaison pour les
KPI de la partie 2 : sans elles, aucun gain ne pourra être démontré.

---

## 2. Périmètre analysé

### 2.1 Architecture réelle

MicroCRM est une application de démonstration de type CRM, organisée en **monorepo** :

| Composant | Répertoire | Technologie | Port |
|---|---|---|---|
| Back-end | `back/` | Spring Boot 3.2.5, Java 17, Gradle 8.7 | 8080 |
| Front-end | `front/` | Angular 17.3, TypeScript 5.4 | 80 / 443 |
| Configuration Docker | `misc/docker/` | Caddyfile, supervisor.ini | — |
| Conteneurisation | `Dockerfile` (racine) | Multi-stage, 5 étapes | — |

Le back-end expose une API REST HATEOAS (Spring Data REST) sur deux entités, `Person` et
`Organization`. Le front-end est une application Angular compilée en fichiers statiques, servie par
**Caddy** — et non par nginx comme on pourrait le supposer.

### 2.2 Écarts entre les consignes et le dépôt réel

Trois écarts ont été relevés et méritent d'être signalés, car ils invalident des hypothèses
formulées dans l'énoncé.

| Consignes | Réalité du dépôt | Conséquence |
|---|---|---|
| Outil de build cité : **Maven** | Le back-end est en **Gradle** (`./gradlew`) | Toute la chaîne CI back repose sur `gradle/actions/setup-gradle`, non sur Maven |
| Livrable attendu : « les **Dockerfiles** » | **Un seul** Dockerfile multi-stage à la racine | Décision à prendre : conserver ou scinder (voir §7) |
| — | Tests front-end en **Karma**, exigeant `CHROME_BIN` | Configuration navigateur headless obligatoire en CI |

Ces écarts ne sont pas des erreurs à corriger dans le dépôt : ce sont des **contraintes à intégrer**
dans la conception du pipeline, et à justifier dans la documentation technique.

---

## 3. Baseline mesurée

Toutes les mesures ont été relevées le 4 septembre 2026, sur WSL2 Ubuntu 24.04, avec Java 21.0.12,
Node 20.20.2 et Docker 29.6.1.

### 3.1 Build et tests

| Opération | Durée | Résultat |
|---|---|---|
| `./gradlew clean build` (back) | 42 s | Succès — 2 tests exécutés en 7,8 s |
| Démarrage de l'application (JAR local) | 7,5 s | Port 8080, données de démonstration chargées |
| `npm install` (front, sans cache) | 3 min 37 s | 925 paquets installés |
| `npm test` (ChromeHeadlessNoSandbox) | 35,7 s | Succès — 8 tests |
| `ng build` (front) | 9,2 s | 311,81 kB brut / 82,86 kB transféré |

### 3.2 Conteneurisation

| Image | Durée de build | Taille | Taille compressée |
|---|---|---|---|
| `orion-back` | 1 min 33 s | 374 MB | 115 MB |
| `orion-front` | 1 min 34 s | 78,7 MB | 21,5 MB |

### 3.3 Couverture fonctionnelle des tests

L'état initial du projet comporte **10 tests au total** :

- **Back-end (2)** : un test de chargement du contexte Spring (`contextLoads`) et un test
  d'intégration sur le dépôt `Person` (recherche par email).
- **Front-end (8)** : tests unitaires répartis sur les composants Angular.

Aucun outil de couverture n'est configuré, ni côté back (JaCoCo absent), ni côté front (le
`coverageReporter` de Karma ne produit ni `lcov` ni format exploitable par un analyseur externe).
**Le taux de couverture initial est donc inconnu et non mesurable en l'état.**

### 3.4 Validation fonctionnelle de bout en bout

Le fonctionnement complet a été vérifié en exécutant simultanément les deux conteneurs et en
ouvrant l'application dans un navigateur. Les appels `GET /persons` et `GET /organizations`
retournent un statut **200**, les données s'affichent, et aucune erreur CORS ou de contenu mixte
n'est remontée par la console. Le CORS est donc déjà correctement configuré côté back-end.

---

## 4. Constats classés par criticité

Vingt-cinq constats ont été relevés. Ils sont répartis en trois catégories selon leur impact sur la
suite du projet.

### 4.1 Bloquants — empêchent l'industrialisation

Ces constats doivent être traités avant ou pendant la mise en place du pipeline. Sans eux, la CI ne
peut pas fonctionner.

| Réf. | Constat | Impact | Statut |
|---|---|---|---|
| **B1** | `back/gradlew` est versionné en mode `100644` : le bit exécutable n'a jamais été committé | `./gradlew` échoue sous Linux et sur les runners `ubuntu-latest`. Le projet n'est buildable que sous Windows | **Résolu** — PR `build/gradlew-executable-bit` |
| **B2** | La CLI Angular affiche une invite d'acceptation des statistiques d'usage au premier lancement | Sur un runner vierge, le job peut rester bloqué jusqu'au timeout | À traiter au lot CI (`NG_CLI_ANALYTICS=false`) |
| **B3** | SonarQube Cloud n'accepte plus les analyses exécutées sur un runtime Java inférieur à 21 depuis le 20 juillet 2026, alors que `build.gradle` déclare `sourceCompatibility = '17'` | L'analyse de qualité exigée par les consignes est impossible en l'état | Décision prise : migration en toolchain Java 21 |
| **B4** | GitHub retire Node.js 20 des runners le 16 septembre 2026 ; les actions en `@v4` cesseront de fonctionner | Un pipeline écrit avec les versions d'actions couramment citées serait obsolète avant sa mise en service | Versions cibles arrêtées (voir §6.2) |

### 4.2 À corriger — défauts de qualité et de sécurité

Ces constats n'empêchent pas le pipeline de fonctionner, mais leur correction est attendue au titre
de la qualité et de la sécurité.

| Réf. | Constat | Domaine | Priorité |
|---|---|---|---|
| **C1** | Le Dockerfile déclare `FROM node` et `FROM gradle:jdk17` sans version figée. `FROM node` se résout en `node:latest`, soit une version non supportée par Angular 17 | Reproductibilité | Haute |
| **C2** | Le stage back compile en JDK 17 mais installe `openjdk21-jre-headless` à l'exécution | Cohérence | Haute |
| **C3** | Le stage back déclare `EXPOSE 4200` alors que l'API écoute sur **8080** (vérifié) | Exactitude | Haute |
| **C4** | Aucune directive `USER` : tous les stages s'exécutent en `root`, y compris Caddy et la JVM | Sécurité (OWASP A05) | Haute |
| **C5** | Le stage `standalone` exécute `COPY --from=front / /`, copiant l'intégralité du système de fichiers de l'étape précédente | Sécurité / taille | Haute |
| **C6** | Le cache Docker est inexploité : les sources sont copiées avant la résolution des dépendances, invalidant celle-ci à chaque modification | Performance | Haute |
| **C7** | Aucun `JaCoCo` configuré côté back : la couverture de tests n'est pas mesurable | Qualité | Haute |
| **C8** | Le `coverageReporter` de Karma ne produit pas de rapport `lcov`, requis par SonarQube Cloud | Qualité | Haute |
| **C9** | `build.gradle` déclare `spring-boot-starter-data-jpa` **deux fois** | Qualité | Moyenne |
| **C10** | Aucun `HEALTHCHECK` dans les images | Exploitabilité | Moyenne |
| **C11** | Le `.dockerignore` n'exclut pas `.git` | Taille / hygiène | Moyenne |
| **C12** | Cinq avertissements `FromAsCasing` remontés par le linter BuildKit | Qualité | Basse |
| **C13** | `spring.jpa.open-in-view` est activé par défaut (anti-pattern signalé au démarrage) | Qualité | Moyenne |
| **C14** | Aucun fichier `docker-compose.yml` : l'orchestration est entièrement à créer | Livrable manquant | Haute |
| **C15** | Aucun répertoire `.github/workflows` : aucune automatisation existante | Livrable manquant | Haute |

### 4.3 Risques identifiés — à documenter, hors périmètre de correction

Ces constats relèvent d'un arbitrage explicite : les corriger sortirait du périmètre du projet. Ils
doivent être **assumés et justifiés** dans le plan de sécurité et le plan de mise à jour.

| Réf. | Risque | Analyse |
|---|---|---|
| **R1** | **L'intégralité de la stack est en fin de support.** Angular 17 : EOL depuis le 14 mai 2025. Node 20 : EOL depuis le 30 avril 2026. Spring Boot 3.2.5 : fin du support open source le 31 décembre 2024 | Aucun correctif de sécurité ne sera publié pour ces versions. La migration représenterait une refonte majeure, hors périmètre. À documenter comme dette structurelle avec une trajectoire de mise à jour |
| **R2** | **88 vulnérabilités npm** (3 critiques, 56 hautes, 21 moyennes, 8 basses) | La correction complète imposerait une migration vers Angular 21 (changement majeur). Analyse à affiner en distinguant les dépendances de développement — non livrées au navigateur — des dépendances de production |
| **R3** | **Aucune persistance des données.** La base HSQLDB est en mémoire, sans datasource configurée ; les données sont régénérées à chaque démarrage par `InitialDataFixture` | Conditionne directement le plan de sauvegarde : soit on documente qu'il n'y a pas de données à sauvegarder, soit on introduit une base persistante |
| **R4** | **L'URL de l'API est figée à la compilation** (`front/src/app/config.ts` → `http://localhost:8080`) | Une image construite pour un environnement ne peut être redéployée ailleurs sans reconstruction. Contraire au principe *build once, deploy anywhere* |
| **R5** | **Configuration Caddy problématique.** Le Caddyfile déclare le site `*`, ce qui active l'HTTPS automatique : le contenu est servi sur le port 443 avec un certificat auto-signé, le port 80 ne faisant que rediriger. L'autorité de certification locale n'est pas persistée | Le `docker-compose` devra exposer le port 443. Un déploiement sur un nom de domaine réel échouerait à obtenir un certificat |
| **R6** | **Contenu mixte.** Le front est servi en HTTPS et appelle l'API en HTTP. Le fonctionnement observé ne tient qu'à l'exception accordée par les navigateurs à `localhost` | Bloquerait tout déploiement sur un domaine réel. Élément déterminant pour la décision relative à la cible de déploiement |

---

## 5. Points positifs relevés

L'audit ne relève pas que des défauts. Trois éléments du dépôt initial sont conformes aux bonnes
pratiques et seront conservés :

- Le Dockerfile utilise **`npm ci`** et non `npm install` pour le front-end, garantissant une
  installation stricte et reproductible depuis le `package-lock.json`.
- Le fichier `karma.conf.js` définit déjà un launcher **`ChromeHeadlessNoSandbox`**, directement
  exploitable en CI sans modification.
- Le **CORS est configuré** côté back-end, ce que la validation fonctionnelle a confirmé.

---

## 6. Veille technologique

### 6.1 Statut de support des composants

| Composant | Version du projet | Fin de support | Versions encore supportées |
|---|---|---|---|
| Angular | 17.3 | 14 mai 2025 | 20, 21, 22 |
| Node.js | 20.x (imposé par Angular 17) | 30 avril 2026 | 22, 24, 26 |
| Spring Boot | 3.2.5 | 31 décembre 2024 | 4.0, 4.1 |

Angular applique un cycle de support de 18 mois — 6 mois de support actif, puis 12 mois de LTS —
ce qui explique le rythme d'obsolescence observé.

### 6.2 Versions d'actions GitHub retenues

Le retrait de Node.js 20 des runners au 16 septembre 2026 impose d'écarter toutes les actions en
version `@v4`.

| Action | Version retenue | Justification |
|---|---|---|
| `actions/checkout` | `@v6` | Version utilisée par la documentation officielle Gradle |
| `actions/setup-java` | `@v5` | Les versions v1 à v4 sont dépréciées |
| `actions/setup-node` | `@v5` | Compatible Node 24 |
| `gradle/actions/setup-gradle` | `@v6` | Avec `cache-provider: basic` (voir ci-dessous) |
| `SonarSource/sonarqube-scan-action` | `@v8` | Remplace `sonarcloud-github-action`, déprécié |
| `docker/setup-buildx-action` | `@v4` | — |
| `docker/login-action` | `@v4` | — |
| `docker/metadata-action` | `@v6` | Gestion déclarative du taggage |
| `docker/build-push-action` | `@v7` | — |

**Point de vigilance sur `gradle/actions` v6** : cette version modifie la licence du composant de
cache. Un provider alternatif, gratuit pour tous les dépôts publics comme privés, doit être activé
explicitement via `cache-provider: basic` afin d'éviter toute dépendance à une offre commerciale.

**Renommage** : SonarCloud s'appelle désormais **SonarQube Cloud**. L'action citée dans les
consignes (`sonarcloud-github-action`) est dépréciée au profit de `sonarqube-scan-action`.

### 6.3 Patterns de pipelines comparables

**Monorepo et filtrage par chemins.** Le schéma dominant consiste à faire précéder les jobs de build
d'un job de détection des changements, exposant des sorties booléennes consommées par des conditions
`if`. Les filtres natifs de GitHub ne conviennent pas, car ils s'appliquent au niveau du workflow et
non des jobs. Un piège documenté doit être anticipé : lorsqu'un workflow est intégralement ignoré
par un filtre, les *status checks* obligatoires peuvent rester en attente et bloquer le merge. La
parade consiste à ajouter un job d'agrégation en `if: always()`.

**Publication d'images.** Le pattern stable repose sur `login-action` avec le `GITHUB_TOKEN`
intégré, `metadata-action` pour le taggage déclaratif (SHA court, nom de branche, SemVer, `latest`
conditionné à la branche par défaut) et `build-push-action` pour la construction. Deux avantages
directs : **aucun secret à créer** pour publier sur GHCR, et absence de facturation du stockage et
de la bande passante pour les images de conteneurs.

**Vulnérabilité de configuration à éviter.** Déclarer la permission `packages: write` au niveau du
workflow alors que celui-ci se déclenche également sur `pull_request` autorise du code issu d'une
PR non fiable à publier ou écraser des images. Cette permission doit être restreinte au seul job de
publication. Ce point sera intégré au plan de sécurité.

---

## 7. Décisions

### 7.1 Décisions actées

| Réf. | Décision | Justification |
|---|---|---|
| **D1** | Travail sur un **fork** du dépôt OpenClassrooms, dépôt public | Le caractère public conditionne la gratuité de SonarQube Cloud et la disponibilité de la protection de branche |
| **D2** | Stratégie de branches **trunk-based / GitHub Flow** : `main` protégée, branches courtes, merge par pull request. Pas de branche `develop`, pas de branche de release | Les consignes ne mentionnent jamais `develop` et ne citent que les déclencheurs `push` et `pull_request`. Gitflow contredirait l'instruction « ne créez pas de complexité inutile » et allongerait mécaniquement le Lead Time mesuré en partie 2 |
| **D3** | **Matrice de déclencheurs** : `pull_request` → build, tests et quality gate ; `push` sur `main` → build, tests, analyse et publication d'images ; `schedule` nocturne → tests complets ; tag `v*` → release SemVer | Se mappe directement sur le §4.2 du template de documentation et rend démontrable l'indicateur d'autoévaluation relatif au déclenchement des tests |
| **D4** | Migration de la toolchain vers **Java 21**, avec alignement de l'image de build Docker | Le runtime 21 est déjà utilisé à l'exécution par le Dockerfile existant. Surtout, SonarQube Cloud l'impose depuis le 20 juillet 2026 |
| **D5** | Messages de commit au format **Conventional Commits** | Prérequis à l'automatisation des releases (étape 6 facultative) ; doit être appliqué dès le premier commit |
| **D6** | Environnement de développement sous **WSL2 Ubuntu** | Parité avec l'image `ubuntu-latest` des runners GitHub, garantissant la reproductibilité entre local et CI |

### 7.2 Décisions ouvertes

| Réf. | Question | Options | Échéance |
|---|---|---|---|
| **O1** | Cible du déploiement continu | Publication d'images sur GHCR seul / déploiement sur PaaS gratuit / VPS avec accès SSH | Avant la rédaction du plan de déploiement |
| **O2** | Structure des Dockerfiles | Conserver le multi-stage unique / scinder en `back/Dockerfile` et `front/Dockerfile` | Lot conteneurisation |
| **O3** | Découpage des workflows | Fichier unique / séparation CI et CD | Lot CI |
| **O4** | Exécution des tests dans le build Docker | Conserver (filet de sécurité) / retirer via `-x test` (rapidité, la CI ayant déjà validé) | Lot conteneurisation |
| **O5** | Filtrage par chemins en CI | Adopter (gain de temps) / écarter (simplicité) | Lot CI, à arbitrer sur mesures |
| **O6** | Introduction d'une base de données persistante | Conserver HSQLDB en mémoire / introduire une base persistante | Avant le plan de sauvegarde |

Les décisions O1 et O6 sont les plus structurantes : elles conditionnent respectivement le plan de
déploiement et le plan de sauvegarde, tous deux exigés en partie 2.

---

## 8. Anticipation des exigences de la partie 2

La partie 2 ne se fabrique pas après coup : elle exploite les traces produites par la partie 1.
Quatre dispositions doivent être prises dès maintenant, faute de quoi les livrables de la seconde
moitié du projet seront incomplets ou artificiels.

**Couverture de tests.** JaCoCo côté back et un rapport `lcov` côté front doivent être configurés
dès la mise en place de la CI. Sans eux, SonarQube Cloud affichera une couverture nulle et
l'indicateur d'autoévaluation portant sur le taux de couverture restera sans réponse.

**Format des logs.** Les logs Spring Boot doivent être structurés en JSON dès la conteneurisation.
Les consignes de la partie 2 recommandent explicitement ce format pour l'intégration à la stack ELK.
Différer ce choix imposerait de reconstruire les images et ferait perdre l'historique de logs déjà
accumulé.

**Historique du pipeline.** Les métriques DORA se calculent sur des exécutions réelles. Les
consignes demandent un relevé sur trois exécutions au minimum ; une quinzaine constitue un
échantillon nettement plus significatif. Cela implique un travail par petits incréments, avec des
commits et des pull requests étalés dans le temps, plutôt qu'une livraison massive en fin de projet.

**Données d'échec.** Sans aucune exécution en échec, le *Change Failure Rate* vaut zéro et le *Mean
Time to Restore* est incalculable. Un ou deux échecs de pipeline suivis d'une correction doivent
être conservés dans l'historique et documentés — ce sont des données de mesure, non des défauts.

---

## 9. Traçabilité avec la fiche d'autoévaluation

Le tableau ci-dessous relie les constats et décisions du présent audit aux indicateurs de réussite
de la fiche d'autoévaluation, afin de garantir qu'aucun critère ne reste sans réponse.

| Indicateur d'autoévaluation | Éléments de l'audit concernés |
|---|---|
| L'environnement de tests permet d'exécuter les tests automatisés et d'en récupérer les résultats | §3.1, §3.3, constats B1, B2, C7, C8 |
| Les actions, outils ou scripts utilisés pour chaque étape sont justifiés | §6.2 (tableau des versions et justifications), §7.1 |
| Les outils et actions utilisés sont adaptés au projet full-stack | §2.1, §2.2, §6.2 |
| Les scripts ou actions ne comportent pas d'étapes inutiles ou manquantes | §7.1 D2, décisions ouvertes O3 et O5 |
| Le plan de conteneurisation est clair et permet de comprendre, exécuter et maintenir le pipeline | §4.2 (constats C1 à C6, C10 à C12), §7.2 O2 et O4 |
| Les dépendances sont installées et les tests positionnés au bon moment dans le workflow | §7.1 D3 |
| Les tests sont déclenchés selon les règles prévues, en cohérence avec le plan de testing périodique | §7.1 D3 (matrice de déclencheurs) |
| Le plan de sécurité est cohérent avec le code et le workflow réels | §4.3 (R1, R2, R4, R5, R6), §6.3 (permissions du token) |
| Les plans présentent les risques pouvant survenir lors de la mise en production | §4.3 dans son ensemble |
| Les éléments contribuant à la dette technique sont identifiés | §4.2, §4.3 R1 et R2 |
| Au moins un point critique est mis en évidence | Constats B1, B3, B4 et R1 |

---

## 10. Conclusion et prochaines étapes

Le dépôt fourni est fonctionnel mais **non industrialisable en l'état**. Un défaut bloquant a déjà
été corrigé — l'absence de bit exécutable sur `gradlew`, qui rendait le projet inconstructible sous
Linux et donc sur les runners GitHub. Trois autres contraintes bloquantes ont été identifiées et
leurs solutions arrêtées.

L'analyse fait ressortir un enjeu qui dépasse les défauts techniques ponctuels : **l'intégralité de
la stack applicative est en fin de support**. Ce constat ne sera pas corrigé dans le cadre du
projet, mais il constitue l'axe principal du plan de mise à jour et un élément central du plan de
sécurité.

Les prochaines étapes, par ordre de dépendance :

1. Formaliser les trois plans exigés par l'étape 2 des consignes — testing périodique, sécurité,
   conteneurisation et déploiement — **avant** toute écriture de workflow.
2. Trancher les décisions ouvertes O1 et O6, qui conditionnent respectivement le plan de
   déploiement et le plan de sauvegarde.
3. Mettre en place la CI avec la configuration de couverture, sans laquelle les indicateurs de la
   partie 2 resteront vides.

---

*Ce document constitue la version 1.0 de l'audit. Il a vocation à être enrichi au fil du projet,
notamment par les résultats de l'analyse SonarQube Cloud et par la résolution des décisions
ouvertes.*
