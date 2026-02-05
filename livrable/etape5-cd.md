# Étape 5 — Déploiement continu (CD)

## Objectif
Automatiser une étape de déploiement en publiant les images Docker après validation CI.

---

## Workflow CD mis en place
Fichier : `.github/workflows/cd.yml`

### Déclencheurs
- **Manuel** : `workflow_dispatch`
- **Tag SemVer** : `vX.Y.Z`

### Ce que fait le CD
- Build des images Docker **back** et **front**
- Push vers **GitHub Container Registry (GHCR)**

---

## Pourquoi ce choix
- Simple et conforme à la consigne (publication d’images)
- Ne nécessite pas d’infrastructure de déploiement externe
- Permet un vrai flux CD reproductible

---

## Secrets et sécurité
- Utilise `GITHUB_TOKEN` (fourni par GitHub) pour se connecter à GHCR
- Aucun secret en clair dans le repo

---

## Checklist Étape 5
- [x] Workflow CD créé
- [x] Déclenchement par tag SemVer
- [x] Publication d’images Docker automatisée
- [x] Sécurité respectée (secrets GitHub)
