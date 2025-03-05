


Création d'une release :
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin v1.0.0

// Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0




Format d'un commit:
<type>(<scope>): <description>

[corps du commit]

[pied de page du commit]

exemple: 
feat(auth): add support for Google authentication
fix(login): resolve issue with incorrect password validation
docs(readme): update installation instructions
feat: fonctionnalités impactaant plusieurs domaines fonctionnels

zlzlzl


Z
Z
Z
ZZ
dmdmdmdm




on test








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