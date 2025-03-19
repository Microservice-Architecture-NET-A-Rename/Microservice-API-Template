## Documentation du Workflow GitHub Actions pour la Publication d'Images Docker

### Vue d'Ensemble

Ce workflow GitHub Actions automatise le processus de construction et de publication d'une image Docker vers Docker Hub lors de la publication d'une release sur GitHub.

### Déclencheur

Le workflow est déclenché lorsqu'une nouvelle release est publiée sur le repository GitHub. Voici comment cela est configuré :

```markdown
on:
  release:
    types: [published]
```

### Variables d'Environnement

Les variables d'environnement définissent le registre Docker et le nom de l'image :

```markdown
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

- **REGISTRY**: Définit le registre Docker (ici, GitHub Container Registry).
- **IMAGE_NAME**: Utilise le nom du repository GitHub comme nom d'image.

### Job: push_to_registry

#### Configuration

Le job s'exécute sur la dernière version d'Ubuntu avec des permissions spécifiques :

```markdown
runs-on: ubuntu-latest
permissions:
  packages: write
  contents: read
  attestations: write
  id-token: write
```

#### Étapes

##### 1. Checkout du Code

Cette étape récupère le code source du repository :

```markdown
- name: Check out the repo
  uses: actions/checkout@v4
```

##### 2. Connexion à Docker Hub

Authentifie le workflow auprès de Docker Hub en utilisant les secrets définis dans le repository :

```markdown
- name: Log in to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

##### 3. Extraction des Métadonnées

Extrait les métadonnées (tags, labels) pour l'image Docker :

```markdown
- name: Extract metadata (tags, labels) for Docker
  id: meta
  uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
  with:
    images: zaelyndra/microserviceapitemplate
```

##### 4. Construction et Publication de l'Image Docker

Construit l'image Docker à partir du Dockerfile spécifié et la pousse vers Docker Hub avec les tags et labels extraits :

```markdown
- name: Build and push Docker image
  id: push
  uses: docker/build-push-action@v6
  with:
    context: .
    file: './MicroserviceApiTemplate/Dockerfile'
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

##### 5. Génération de l'Attestation d'Artefact

Génère une attestation de provenance pour l'image Docker publiée, améliorant ainsi la sécurité et la traçabilité :

```markdown
- name: Generate artifact attestation
  uses: actions/attest-build-provenance@v2
  with:
    subject-name: index.docker.io/zaelyndra/microserviceapitemplate
    subject-digest: ${{ steps.push.outputs.digest }}
    push-to-registry: true
```

### Liens Utiles

- [Actions Marketplace: Build and push Docker images](https://github.com/marketplace/actions/build-and-push-docker-images)
- [Docker Metadata Action](https://github.com/docker/metadata-action)
- [Docker Hub Quickstart](https://docs.docker.com/docker-hub/quickstart/#step-3-build-and-push-an-image-to-docker-hub)
- [Docker Build and Push Action](https://github.com/docker/build-push-action)
- [GitHub: Publishing Docker images](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images#publishing-images-to-github-packages)
- [GitHub: Attest Build Provenance Action](https://github.com/actions/attest-build-provenance)
- [Azure DevOps: Configuring a pipeline to publish Docker images to a registry](https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/containers/push-image?view=azure-devops&tabs=yaml&pivots=docker-registry)
- [GitHub: Configuring a workflow to publish Docker images to a registry](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images)


































