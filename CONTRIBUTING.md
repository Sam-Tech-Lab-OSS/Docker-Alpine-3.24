# Contributing — Docker Alpine 3.24

Thanks for your interest in improving this project.

---

## Before you start

- **Security vulnerability?** Do not open a public issue — follow the process in [`SECURITY.md`](./SECURITY.md) instead.
- **Single source of truth:** the image is built from one file, [`Dockerfile-multi-arch`](./Dockerfile-multi-arch). There are no separate per-architecture Dockerfiles — please don't reintroduce that pattern without discussing it in an issue first.

## Reporting bugs or suggesting changes

Open an issue describing:
- the affected file (`Dockerfile-multi-arch`, a workflow in `.github/workflows/`, or documentation)
- the problem or suggestion
- if relevant, the `docker build` / `docker run` output that reproduces it

## Submitting a change

1. Fork the repository and create a branch from `main`.
2. Make your change.
3. Lint the Dockerfile locally before opening a PR:
   ```bash
   docker run --rm -i ghcr.io/hadolint/hadolint:latest hadolint --failure-threshold error - < Dockerfile-multi-arch
   ```
4. Build locally to confirm the image still builds and runs:
   ```bash
   docker buildx build -f Dockerfile-multi-arch -t alpine-3.24 .
   docker run -it --rm alpine-3.24 /bin/bash
   ```
5. Open a pull request against `main` describing what changed and why.

## CI on pull requests

- `build-multi-arch.yml` runs **hadolint and the integration tests on every pull request**, so a PR that changes the Dockerfile or a workflow is fully checked before merge. The tests run as a matrix over **amd64 and arm64** — arm64 builds and runs under QEMU emulation. Publishing waits on both, and only happens on a push to `main` (path-filtered to `Dockerfile-multi-arch`), on the monthly schedule, or via manual dispatch — never from a pull request.
- Both registries are served by a **single build**, so they publish the same digest. Adding a tag means adding it to the `Composer la liste des tags` step, not adding a second build.
- Do **not** quote values in the `labels:` block. Each line is passed verbatim to `--label` and buildx splits on the first `=` without stripping anything, so quotes end up inside the published value. A post-publish step fails the build if any label carries them.
- `vuln-scan.yml` scans the published image weekly, and after any build that actually published one — it isn't part of PR review either.
- **The Docker Hub description** is `README-dockerhub.md`, not `README.md`: Docker Hub caps the overview at 25 000 bytes, strips most raw HTML and does not resolve repository-relative links. It has its own workflow, `dockerhub-description.yml`, triggered when that file changes — a documentation edit does not require an image rebuild to reach Docker Hub. Keep both READMEs bilingual and in agreement on facts.

## Style

- Keep documentation bilingual (English section, then a parallel French section), matching the existing `README.md` and `SECURITY.md`.
- Prefer minimal, targeted changes over refactors — this is a small, single-purpose base image.

## Code of Conduct

This project follows the [Code of Conduct](./CODE_OF_CONDUCT.md).

---

## Français

Merci de votre intérêt pour ce projet.

### Avant de commencer

- **Vulnérabilité de sécurité ?** N'ouvrez pas d'issue publique — suivez la procédure décrite dans [`SECURITY.md`](./SECURITY.md).
- **Source unique de vérité :** l'image est construite à partir d'un seul fichier, [`Dockerfile-multi-arch`](./Dockerfile-multi-arch). Il n'y a plus de Dockerfiles séparés par architecture — merci de ne pas réintroduire ce schéma sans en discuter d'abord dans une issue.

### Signaler un bug ou proposer un changement

Ouvrez une issue en précisant :
- le fichier concerné (`Dockerfile-multi-arch`, un workflow de `.github/workflows/`, ou la documentation)
- le problème ou la suggestion
- si pertinent, la sortie `docker build` / `docker run` permettant de reproduire le problème

### Proposer une modification

1. Forkez le dépôt et créez une branche depuis `main`.
2. Effectuez votre modification.
3. Vérifiez le Dockerfile localement avant d'ouvrir une PR :
   ```bash
   docker run --rm -i ghcr.io/hadolint/hadolint:latest hadolint --failure-threshold error - < Dockerfile-multi-arch
   ```
4. Construisez l'image localement pour confirmer qu'elle build et fonctionne toujours :
   ```bash
   docker buildx build -f Dockerfile-multi-arch -t alpine-3.24 .
   docker run -it --rm alpine-3.24 /bin/bash
   ```
5. Ouvrez une pull request vers `main` en décrivant ce qui a changé et pourquoi.

### CI sur les pull requests

- `build-multi-arch.yml` exécute **hadolint et les tests d'intégration sur chaque pull request** : une PR modifiant le Dockerfile ou un workflow est donc entièrement vérifiée avant merge. Les tests tournent en matrice sur **amd64 et arm64**, l'arm64 étant construit et exécuté sous émulation QEMU. La publication attend les deux, et n'a lieu que sur un push vers `main` (filtré sur `Dockerfile-multi-arch`), sur la planification mensuelle, ou via déclenchement manuel — jamais depuis une pull request.
- Les deux registres sont servis par un **build unique**, afin qu'ils publient le même digest. Ajouter un tag consiste à l'ajouter à l'étape `Composer la liste des tags`, pas à ajouter un second build.
- N'encadrez **pas** les valeurs du bloc `labels:` de guillemets. Chaque ligne est passée telle quelle à `--label`, et buildx découpe au premier `=` sans rien retirer : les guillemets se retrouvent dans la valeur publiée. Une étape post-publication fait échouer le build si un label en porte.
- `vuln-scan.yml` scanne l'image publiée chaque semaine, et après tout build en ayant réellement publié une — il ne fait pas non plus partie de la revue de PR.
- **La description Docker Hub** est `README-dockerhub.md`, et non `README.md` : Docker Hub plafonne l'aperçu à 25 000 octets, retire l'essentiel du HTML brut et ne résout pas les liens relatifs au dépôt. Elle a son propre workflow, `dockerhub-description.yml`, déclenché quand ce fichier change — une modification de documentation ne nécessite pas de reconstruction d'image pour atteindre Docker Hub. Gardez les deux README bilingues et d'accord sur les faits.

### Style

- Gardez la documentation bilingue (section anglaise, puis section française parallèle), comme dans `README.md` et `SECURITY.md`.
- Préférez des changements minimes et ciblés aux refontes — c'est une image de base petite et à but unique.

### Code de conduite

Ce projet suit le [Code de conduite](./CODE_OF_CONDUCT.md).
