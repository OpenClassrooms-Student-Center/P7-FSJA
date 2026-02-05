# Étape 6 — Release automatique (SemVer)

> Étape optionnelle — mise en place d’un workflow de release versionnée.

## Objectif
Automatiser la création d’une release GitHub lorsqu’un tag SemVer `vX.Y.Z` est poussé.

---

## Workflow mis en place
Fichier : `.github/workflows/release.yml`

### Déclencheur
- `push` sur tag `v*.*.*`

### Ce que fait le workflow
1. **Build back-end** (JAR)
2. **Build front-end** (dist Angular)
3. **Packaging** des artefacts
4. **Création d’une release GitHub** + upload des artefacts

---

## Artefacts publiés
- `microcrm-0.0.1-SNAPSHOT.jar`
- `microcrm-front-dist.tar.gz`

---

## Politique de versioning (SemVer)
- **MAJOR** : breaking change (incompatibilité)
- **MINOR** : nouvelle fonctionnalité rétro‑compatible
- **PATCH** : correction de bug

Exemple : `v1.2.3`

---

## Checklist Étape 6
- [x] Workflow release automatisé
- [x] Déclenchement par tag SemVer
- [x] Artefacts générés et attachés
- [x] Release GitHub créée automatiquement
