# Documentation du Workflow "Create Release Tag"

## Introduction

Ce workflow GitHub Actions automatise la gestion des versions en créant des tags immuables pour suivre les évolutions du projet, conformément aux principes de Trunk Based Development. L'approche "Trunk Based Development" met l'accent sur un tronc principal unique (généralement main ou trunk) et minimise l'utilisation de branches de longue durée. Dans ce contexte, les releases sont gérées principalement par des tags, évitant ainsi la complexité des branches de release traditionnelles.

## Objectifs

- Adopter une stratégie de gestion des versions basée sur les principes de Trunk Based Development.
- Simplifier la gestion des versions en évitant les branches de release superflues.
- Assurer un suivi structuré des modifications à l'aide de tags immuables.
- Faciliter l'application de correctifs sur différentes versions grâce à des branches temporaires créées à partir des tags.
- Générer automatiquement des notes de release basées sur les commits, en suivant les conventions de [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/).
- **Garantir l'auto-documentation des modifications** en associant chaque commit et chaque tag à une tâche dans un outil de gestion de projet comme Azure DevOps, Jira ou GitHub Projects.

## Organisation du Workflow

### Philosophie des Tags (Inspirée de Trunk Based Development)

1. **Tags immuables** : Un tag pointe toujours sur le commit correspondant à la release. Il n'est jamais modifié après sa création. C'est un instantané précis du code à un moment donné.
2. **Absence de branches de release permanentes** : Conformément à Trunk Based Development, il n'y a pas de branches de release de longue durée.
3. **Branches temporaires pour les correctifs** : Si un correctif est nécessaire sur une ancienne version, une branche temporaire est créée à partir du tag correspondant. Une fois le correctif appliqué et testé, un nouveau tag est créé pour cette version corrigée.
4. **Gestion des correctifs (Hotfixes)** :
   - Tout correctif doit **d'abord être appliqué sur la branche principale** (`main` ou `develop` selon le workflow).
   - Ensuite, si le correctif concerne une version spécifique en production, une **branche temporaire est créée depuis le tag concerné** pour appliquer le correctif via un cherry-pick depuis la branche principale.
   - Une fois le correctif validé et testé, un **nouveau tag** est créé sur cette branche temporaire (ex: `v2.0.1`), garantissant la traçabilité.
   - **Si un correctif doit être appliqué à une version ultérieure sans affecter les versions intermédiaires** (ex: `v4.0.1` sans passer par `v3.0.0`), une branche temporaire est créée depuis le tag cible et le correctif est cherry-piqué ou reporté.
   - **Une fois le correctif intégré et le tag publié, la branche temporaire peut être supprimée** sans perte d'historique, car les commits restent accessibles via les tags et les références Git.

### Exemple d'Utilisation des Tags

#### Création d'un tag pour une release :
```sh
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin v1.0.0
```

#### Suppression d'un tag (Attention : Généralement déconseillé après publication) :

```sh
git tag -d v1.0.0
git push origin --delete v1.0.0
```

**Note Importante:**  La suppression d'un tag publié est généralement déconseillée car elle peut perturber les utilisateurs qui dépendent de cette version.  Si une suppression est absolument nécessaire, communiquez clairement l'impact aux utilisateurs concernés.

### Flux de travail

1.  **Développement continu sur le tronc principal** : Les développeurs travaillent directement sur la branche principale (`main` ou `trunk`).  Les fonctionnalités sont intégrées fréquemment.
2.  **Tagging pour les releases** : Lorsqu'une version est prête à être publiée, un tag est créé (ex: `v1.2.3`). Ce tag marque l'état du code au moment de la release.
3.  **Déclenchement du Workflow** : Le push du tag déclenche automatiquement le workflow GitHub Actions.
4.  **Génération des notes de release** : Basé sur les commits entre le tag actuel et le précédent, en utilisant les conventions de [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/).
5.  **Création de la release GitHub** : Une release en mode brouillon est générée avec les notes associées.


### Focus sur les Correctifs (Hotfixes) et le Cherry-Picking

Cette section détaille la procédure à suivre pour appliquer des correctifs sur des versions spécifiques, en utilisant le cherry-picking si nécessaire.

**Scénario : Application d'un correctif à la version v2.0.0 et potentiellement à d'autres versions (v4.0.0 par exemple).**

1. **Reproduire et corriger le bug sur le tronc principal**
    ```sh
    git checkout main
    git pull origin main
    ```
    - Reproduire le bug directement sur `main`
    - Développer le correctif avec tests associés
    - Committer avec un message clair :
    ```sh
    git commit -m "fix: résolution du bug X [RELATES #123]"
    ```

2. **Création des branches de hotfix**
    ```sh
    # Pour v2.0.0
    git checkout -b hotfix/v2.0.1 v2.0.0
    # Pour v4.0.0 
    git checkout -b hotfix/v4.0.1 v4.0.0
    ```

3. **Cherry-pick contrôlé depuis main**
    ```sh
    # Trouver le hash du commit de correctif
    git log main --oneline

    # Appliquer le correctif sur chaque branche
    git cherry-pick <commit-hash> -x
    ```
    *L'option `-x` ajoute une référence au commit original*

4. **Adaptation spécifique à la version (si nécessaire)**
    - Modifier **uniquement** ce qui est incompatible avec l'ancienne version
    - Conserver le même comportement fonctionnel que sur main

5. **Tagging**
    ```sh
    # Création des tags
    git tag -a v2.0.1 -m "Port du correctif #123 depuis main"
    git tag -a v4.0.1 -m "Port du correctif #123 depuis main"
    ```

6. **Propagation vers le dépôt**
    ```sh
    git push origin --tags
    git push origin hotfix/v2.0.1
    git push origin hotfix/v4.0.1
    ```

**Points-clés :**
1. 🚫 **Interdiction** de développer directement sur les branches de release
2. ✅ **Obligation** de valider le correctif sur main avant tout cherry-pick
3. 🔍 **Vérification** systématique des différences avec :
    ```sh
    git diff main..hotfix/v2.0.1 -- <fichiers-corrigés>
    ```
4. 🗑️ **Nettoyage** des branches après merge :
    ```sh
    git branch -d hotfix/v2.0.1 hotfix/v4.0.1
    ```

**Cas exceptionnel** (si le bug n'existe pas sur main) :
1. Développer le correctif sur la branche release
2. **Backport** obligatoire vers main avec :
    ```sh
    git checkout main
    git cherry-pick <commit-hash> --no-commit
    git commit -m "chore: backport du correctif #123 [FROM v2.0.1]"
    ```

---


### Génération et Traçabilité des Notes de Release

Le workflow inclut un mécanisme d'auto-documentation en générant des notes de release détaillées. Ces notes sont créées automatiquement à partir des messages de commit, en respectant la convention [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/). Chaque modification est associée à une tâche spécifique dans un outil de gestion de projet comme Azure DevOps, Jira ou GitHub Projects.

**Avantages :**
-   **Lien direct entre les modifications et les tâches projet** : Chaque commit doit inclure un identifiant de tâche (ex: `feat(core): ajout du support multi-langues [AB#1234]` pour Azure DevOps ou `fix(ui): correction du bug d'affichage (JIRA-456)`).
-   **Meilleure traçabilité** : Les équipes peuvent rapidement identifier les raisons des modifications et retrouver l'historique des décisions.
-   **Automatisation du reporting** : Les releases GitHub contiennent automatiquement des liens vers les tickets et tâches concernées, améliorant la transparence et la communication entre équipes.


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


## Avantages de cette approche (Trunk Based Development avec Tags)

1.  **Simplicité** : Pas de gestion de branches de release complexes, uniquement des tags immuables. Le code est intégré en continu sur le tronc principal.
2.  **Réduction des conflits** : En intégrant fréquemment le code, on réduit considérablement les risques de conflits de fusion (merge conflicts).
3.  **Flexibilité** : Facilité à appliquer des correctifs ciblés en créant des branches temporaires à partir des tags, sans impacter le développement principal.
4.  **Automatisation** : Génération des notes de release et publication de la release GitHub sans intervention manuelle, grâce à GitHub Actions.
5.  **Traçabilité** : Chaque version est clairement identifiée par un tag unique, offrant une traçabilité précise des changements.
6.  **Flux de développement rapide** : Les développeurs restent concentrés sur le développement de nouvelles fonctionnalités plutôt que sur la gestion de branches.

## Outils utilisés

-   **GitHub Actions** : Automatisation du workflow.
-   **conventional-changelog-cli** : Génération des notes de release basées sur les conventions de commit.
-   **softprops/action-gh-release** : Création des releases GitHub.

## Conclusion

Ce workflow optimise la gestion des versions en s'appuyant sur les principes de Trunk Based Development et l'utilisation de tags immuables.  Il élimine ainsi la nécessité de maintenir des branches de release complexes. Il simplifie le processus de release tout en garantissant un suivi précis des modifications et en facilitant l'application de correctifs ciblés.  L'adoption de Trunk Based Development favorise un flux de développement plus rapide et une collaboration plus efficace.





