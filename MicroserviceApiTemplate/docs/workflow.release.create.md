# Documentation du Workflow "Create Release Tag"

## Introduction

Ce workflow GitHub Actions automatise la gestion des versions en créant des tags immuables pour suivre les évolutions du projet. Contrairement aux approches traditionnelles qui maintiennent des branches de release, cette méthode repose uniquement sur les tags, évitant ainsi la gestion de branches inutiles tout en assurant un suivi clair des modifications.

## Objectifs

- Simplifier la gestion des versions en évitant les branches de release superflues.
- Assurer un suivi structuré des modifications à l'aide des tags immuables.
- Faciliter l'application de correctifs sur différentes versions grâce à des branches temporaires.
- Générer automatiquement des notes de release basées sur les commits.

## Organisation du Workflow

### Philosophie des Tags

1. **Les tags sont immuables** : Un tag pointe toujours sur le commit correspondant à la release. Il n'est pas modifié après sa création.
2. **Suppression des branches non nécessaires** : Une fois un tag créé, il n'est plus nécessaire de maintenir une branche associée.
3. **Création de branches temporaires si nécessaire** : En cas de correctif ou d'ajout de fonctionnalité, une branche temporaire est créée à partir du tag concerné.
4. **Gestion des correctifs** :
    - Un correctif sur une version spécifique implique la création d'une branche temporaire depuis le tag concerné.
    - Une fois les correctifs terminés et validés, un nouveau tag est créé (ex: `v2.0.1`).
    - Si un correctif doit être appliqué à une version ultérieure mais pas intermédiaire (ex: `v4.0.1` sans passer par `v3.0.0`), une branche est créée à partir du tag cible et les correctifs sont cherry-pickés.

### Exemple d'Utilisation des Tags

#### Création d'un tag pour une release :
```sh
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin v1.0.0
```

#### Suppression d'un tag :
```sh
git tag -d v1.0.0
git push origin --delete v1.0.0
```

### Flux de travail

1. **Développement** : Les développeurs travaillent sur la branche principale.
2. **Tagging** : Lorsqu'une version est prête, un tag est créé (ex: `v1.2.3`).
3. **Déclenchement du Workflow** : Le push du tag déclenche automatiquement le workflow GitHub Actions.
4. **Génération des notes de release** : Basé sur les commits entre le tag actuel et le précédent.
5. **Création de la release GitHub** : Une release en mode brouillon est générée avec les notes associées.

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

Récupère le code du dépôt avec l'historique complet.

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

Installe `conventional-changelog-cli` pour générer les notes de release.

```yaml
- name: Install dependencies
  run: npm install conventional-changelog-cli
```

### 4. Génération des notes de release

Identifie les tags existants et génère les notes de release entre le dernier tag et le précédent.

```yaml
- name: Generate Release Notes
  id: release_notes
  run: |
    LATEST_TAG=$(git describe --tags --abbrev=0)
    PREVIOUS_TAG=$(git describe --tags --abbrev=0 $LATEST_TAG^ || echo "")
    
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

Crée une release GitHub en mode brouillon avec les notes générées.

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

## Avantages de cette approche

1. **Simplicité** : Pas de gestion de branches de release, uniquement des tags immuables.
2. **Flexibilité** : Facilité à appliquer des correctifs ciblés sans maintenir des branches inutiles.
3. **Automatisation** : Génération des notes de release et publication de la release GitHub sans intervention manuelle.
4. **Traçabilité** : Chaque version est clairement identifiée par un tag unique.

## Outils utilisés

- **GitHub Actions** : Automatisation du workflow.
- **conventional-changelog-cli** : Génération des notes de release.
- **softprops/action-gh-release** : Création des releases GitHub.

## Conclusion

Ce workflow optimise la gestion des versions en s'appuyant sur des tags immuables, éliminant ainsi la nécessité de maintenir des branches de release. Il simplifie le processus tout en garantissant un suivi précis des modifications et en facilitant l'application de correctifs ciblés.

