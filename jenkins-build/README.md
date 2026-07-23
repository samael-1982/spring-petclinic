# Jenkins Build Image

A reusable Docker image for Jenkins Kubernetes agents.

The image contains all tools required by CI/CD pipelines in the Spring PetClinic GitOps project.

## Included software

| Tool | Version |
|------|---------|
| Java | 17 (Temurin) |
| Maven | 3.9.11 |
| Git | Latest from Debian repository |
| OpenSSH Client | Latest from Debian repository |
| curl | Latest from Debian repository |
| wget | Latest from Debian repository |
| yq | v4.48.1 |
| kubectl | v1.34.1 |
| Helm | v3.19.0 |

---

## Build image

```bash
docker build -t ghcr.io/<github-user>/jenkins-build:1.0 .
```

---

## Push to GHCR

```bash
docker push ghcr.io/<github-user>/jenkins-build:1.0
```

---

## Use in Jenkins

Replace the Maven container image in the Kubernetes pod template:

```yaml
image: ghcr.io/<github-user>/jenkins-build:1.0
```

---

## Included tools

The image provides:

- Java 17
- Maven
- Git
- SSH client
- curl
- wget
- yq
- kubectl
- Helm

No additional installation is required during pipeline execution.

---

## Advantages

- Reproducible builds with pinned tool versions.
- Faster Jenkins pipelines.
- Smaller Jenkinsfile.
- Ready for GitOps workflows.
- Suitable for Kubernetes-based Jenkins agents.
- Reusable across multiple CI/CD projects.

---

## Project

Created for the Spring PetClinic GitOps CI/CD project using:

- Jenkins
- Kubernetes
- Kaniko
- GitHub Container Registry (GHCR)
- Argo CD
- Helm