# Documentation du Workflow "Create Release Branch"

## Introduction

Ce workflow GitHub Actions est conçu pour automatiser la gestion des versions dans un projet en créant des branches de release et en générant automatiquement des notes de version. Il repose sur l'utilisation des tags Git pour détecter les nouvelles versions et assurer un suivi structuré des changements.

## Objectifs

- Maintenir un contrôle total sur le versioning.
- Générer automatiquement des notes de release basées sur les commits.
- Créer automatiquement une branche de release à partir du dernier tag.
- Faciliter la gestion des tests et le suivi des modifications.

## Organisation du Workflow

### Flux de travail

1. **Développement** : Les développeurs travaillent sur une branche de développement, implémentant de nouvelles fonctionnalités et corrections.

2. **Tagging** : Lorsque le périmètre fonctionnel d'une release est atteint, un tag est créé (ex: v1.2.3) sur le dernier commit de la branche de développement.

3. **Déclenchement** : Le push du tag déclenche automatiquement le workflow GitHub Actions.

4. **Génération des notes** : Le workflow génère les notes de release en se basant sur les commits entre le tag actuel et le précédent.

5. **Création de la release** : Une nouvelle release GitHub est créée en mode brouillon, incluant les notes générées.

6. **Branche de release** : Une branche de release est automatiquement créée à partir du tag (ex: release/v1.2.3).

### Avantages de cette approche

1. **Contrôle du versioning** :
   - Décision manuelle du moment opportun pour créer une release.
   - Flexibilité dans la numérotation des versions selon les besoins du projet.

2. **Automatisation et cohérence** :
   - Génération automatique des notes de release basées sur les commits conventionnels.
   - Création systématique d'une branche de release pour chaque nouvelle version.

3. **Traçabilité et visibilité** :
   - Les notes de release fournissent un résumé clair des changements entre versions.
   - Les branches de release facilitent les tests finaux et les éventuelles corrections avant déploiement.

4. **Intégration avec le processus de développement** :
   - Encourage l'utilisation de commits conventionnels pour une meilleure organisation des changements.
   - Facilite la revue et la validation des changements avant la publication finale.


## Déclenchement

Le workflow est déclenché lorsqu'un nouveau tag correspondant au format `v*.*.*` est poussé dans le dépôt.

```yaml
on:
  push:
    tags:
      - "v*.*.*"
```

## Permissions

Le workflow nécessite les permissions suivantes :

```yaml
permissions:
    contents: write
    repository-projects: write
```

## Étapes du Workflow

### 1. Checkout du code

Récupère le code du dépôt, en s'assurant d'avoir tout l'historique des commits pour générer correctement les notes de release.

```yaml
- name: Checkout code
  uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

### 2. Configuration de Node.js

Installe Node.js en version 18 pour exécuter les scripts de génération des notes de version.

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'
```

### 3. Installation des dépendances

Installe l'outil `conventional-changelog-cli` pour générer les notes de release.

```yaml
- name: Install dependencies
  run: npm install conventional-changelog-cli
```

### 4. Génération des notes de release

Cette étape :
- Récupère les tags existants.
- Identifie le dernier et l'avant-dernier tag.
- Génère les notes de release entre ces deux tags.
- Stocke le contenu généré pour utilisation dans la création de la release.

```yaml
- name: Generate Release Notes
  id: release_notes
  run: |
    ALL_TAGS=$(git tag --sort=-creatordate)
    LATEST_TAG=$(echo "$ALL_TAGS" | head -n 1)
    PREVIOUS_TAG=$(echo "$ALL_TAGS" | sed -n '2p')
    
    if [ -z "$PREVIOUS_TAG" ]; then
      npx conventional-changelog-cli -p angular -r 0 > release_notes.md
    else
      npx conventional-changelog-cli -p angular -r 2 > release_notes.md
    fi
    
    RELEASE_NOTES=$(cat release_notes.md)
    RELEASE_NOTES_JSON=$(echo "$RELEASE_NOTES" | jq -R -s '.')
    echo "RELEASE_NOTES=$RELEASE_NOTES_JSON" >> $GITHUB_OUTPUT
```

### 5. Récupération du tag

Extrait le tag actuel à partir de la référence GitHub.

```yaml
- name: Get tag
  id: tag
  run: |
    TAG=${GITHUB_REF#refs/tags/}
    echo "TAG=$TAG" >> $GITHUB_OUTPUT
```

### 6. Création de la release GitHub

Utilise `softprops/action-gh-release` pour publier une release en mode brouillon avec les notes générées.

```yaml
- name: Create GitHub Release
  uses: softprops/action-gh-release@v2
  if: startsWith(github.ref, 'refs/tags/')
  with:
    body: ${{ fromJson(steps.release_notes.outputs.RELEASE_NOTES) }}
    tag_name: ${{ steps.tag.outputs.TAG }}
    name: Release ${{ steps.tag.outputs.TAG }}
    draft: true
    prerelease: false
    generate_release_notes: false
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 7. Création de la branche de release

Crée une nouvelle branche de release à partir du tag actuel et la pousse sur le dépôt.

```yaml
- name: Create Release Branch
  run: |
    git checkout -b release/${{ steps.tag.outputs.TAG }}
    git push origin release/${{ steps.tag.outputs.TAG }}
```

## Format des commits

Le workflow utilise `conventional-changelog-cli`, qui repose sur un format de commit structuré. Les principaux types de commits sont :

- `feat(scope): description` → Ajout d'une nouvelle fonctionnalité.
- `fix(scope): description` → Correction d'un bug.
- `docs(scope): description` → Mise à jour de la documentation.

Exemples :

```text
feat(auth): add support for Google authentication
fix(login): resolve issue with incorrect password validation
docs(readme): update installation instructions
```

Plus de détails sur le format des commits : [Conventional Changelog](https://github.com/conventional-changelog/conventional-changelog)

## Avantages de cette approche

1. **Contrôle du versioning** :
   - Permet de définir manuellement quand une version est prête pour une release.
   - Assure la cohérence du suivi des versions grâce aux tags.

2. **Automatisation des releases** :
   - Génère automatiquement une release et ses notes à partir des commits.
   - Structure les changements en catégories (features, fixes, documentation, etc.).

3. **Création automatique des branches de release** :
   - Facilite la gestion des tests et des validations avant mise en production.
   - Permet une meilleure visibilité sur les évolutions du projet.

## Outils utilisés

- **GitHub Actions** : Automatisation du workflow.
- **conventional-changelog-cli** : Génération des notes de release.
- **softprops/action-gh-release** : Création des releases GitHub.

## Liens utiles

- [Conventional Commits](https://www.conventionalcommits.org/) : Guide sur le format de commits conventionnels utilisé par ce workflow.
- [conventional-changelog-cli](https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-changelog-cli) : Documentation de l'outil utilisé pour générer les notes de release.
- [GitHub Actions: Creating releases](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#release) : Documentation officielle sur la création de releases avec GitHub Actions.
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) : Action GitHub utilisée pour créer les releases dans ce workflow.
- [Semantic Versioning](https://semver.org/) : Guide sur le versioning sémantique, utile pour comprendre la structure des tags.
- [GitHub: Managing releases in a repository](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository) : Guide général sur la gestion des releases dans GitHub.

### Ressources complémentaires

- [GitHub Actions: Workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions) : Référence complète sur la syntaxe des workflows GitHub Actions.
- [GitHub: About tags](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases#about-releases) : Informations sur l'utilisation des tags dans GitHub.


## Conclusion

Ce workflow offre une solution robuste et automatisée pour gérer les releases, en assurant un suivi clair des modifications grâce à un conventionnement strict des commits. Il permet d'améliorer la qualité et la traçabilité des versions tout en réduisant la charge manuelle liée à la gestion des releases.








Message format commit: https://github.com/conventional-changelog/conventional-changelog




https://github.com/marketplace/actions/build-and-push-docker-images
https://github.com/docker/metadata-action
https://docs.docker.com/docker-hub/quickstart/#step-3-build-and-push-an-image-to-docker-hub
https://github.com/docker/build-push-action
https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images#publishing-images-to-github-packages
https://github.com/actions/attest-build-provenance

# Liens Utiles
- [Azure Devoops: Configuration d'une pipeline de publication d'images docker vers un registry](https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/containers/push-image?view=azure-devops&tabs=yaml&pivots=docker-registry)
- [Github: Configuration d'une pipeline de publication d'images docker vers un registry](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images)