<h1 align="center">Docker Alpine 3.24</h1>

<p align="center"><strong>Minimal • Hardened • Multi-Architecture • Non-Root • FROM scratch</strong></p>

<table align="center">
  <tr>
    <td align="center" width="50%">
      <a href="https://github.com/Sam-Tech-Lab-OSS" target="_blank">
        <img src="https://raw.githubusercontent.com/Sam-Dz-Devops/Images/main/Sam-Tech-Site-Web.png"
             alt="Sam Tech Lab Logo" width="300"/>
      </a>
    </td>
    <td align="center" width="50%">
      <a href="https://hub.docker.com/r/samtechlab/alpine-3.24" target="_blank">
        <img src="https://raw.githubusercontent.com/Sam-Tech-Lab-Git/Images/refs/heads/main/Alpine-logo.png?sanitize=true"
             alt="Alpine Logo" width="180"/>
      </a>
    </td>
  </tr>
</table>

<p align="center">
<a href="https://hub.docker.com/r/samtechlab/alpine-3.24"><img src="https://img.shields.io/docker/pulls/samtechlab/alpine-3.24?style=for-the-badge&logo=docker"/></a>
<a href="https://hub.docker.com/r/samtechlab/alpine-3.24"><img src="https://img.shields.io/docker/stars/samtechlab/alpine-3.24?style=for-the-badge"/></a>
<img src="https://img.shields.io/badge/Alpine-3.24-0D597F?style=for-the-badge&logo=ubuntu&logoColor=white"/>
<img src="https://img.shields.io/badge/Multi--Arch-amd64%20%7C%20arm64-success?style=for-the-badge"/>
<a href="https://github.com/Sam-Tech-Lab-OSS" target="_blank"><img src="https://img.shields.io/static/v1?label=SamTechLab&message=GitHub&color=94398d&labelColor=555555&style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/></a>
<a href="https://github.com/sponsors/Sam-Tech-Lab-OSS" target="_blank"><img src="https://img.shields.io/badge/Sponsor-GitHub-ea4aaa.svg?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="Sponsor"/></a>
<a href="https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/LICENSE" target="_blank"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg?style=for-the-badge" alt="License: Apache 2.0"/></a>
</p>

---

<p align="center">

  <a href="https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/actions/workflows/build-multi-arch.yml" target="_blank">
      <img src="https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/actions/workflows/build-multi-arch.yml/badge.svg" alt="Build multi-arch"/>
  </a>
  <a href="https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/actions/workflows/vuln-scan.yml" target="_blank">
      <img src="https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/actions/workflows/vuln-scan.yml/badge.svg" alt="Vulnerability Scan"/>
  </a>
</p>

<p align="center">
  <b>English</b> · <a href="#documentation-française">Version française ↓</a>
</p>

---

## Quickstart

```bash
# Run a shell (as the unprivileged appuser)
docker run -it --rm ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

# Or from Docker Hub
docker run -it --rm samtechlab/alpine-3.24:latest
```

Build on top of it:

```dockerfile
FROM ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

# The image runs as appuser; switch to root to install, then switch back.
USER root
RUN apk add --no-cache your-package
USER appuser
```

**Contents:** [Overview](#overview) · [Image reference](#image-reference) ·
[Getting started](#getting-started) · [Complete example NGINX](#complete-example-nginx) ·
[Security model](#security-model) · [Troubleshooting](#troubleshooting) ·
[Maintenance](#maintenance)

---

## Overview

A **minimal, hardened, multi-architecture Alpine 3.24 base image**, built
`FROM scratch` from the **official Alpine OCI rootfs** — for authenticity, small size and
reproducibility.

It is designed as a **foundation for your own images**: a complete Alpine system (without a kernel — containers
share the host's), a non-root user,
and a hardened system baseline — then it gets out of your way.

> **Supported architectures:** `linux/amd64`, `linux/arm64`
> **Automatic monthly rebuilds** pick up the security updates Alpine publishes for 3.24.

### Key features

- ✅ **Built `FROM scratch`** from the official Alpine OCI rootfs — no third-party base layer
- ✅ **Multi-arch** published as a single manifest (`amd64`, `arm64`)
- ✅ **Runs as a non-root user** (`appuser`) end to end — there is no privileged process
- ✅ **System hardening** — `root` account locked, SUID/SGID bits stripped, world-writable bits
  removed, `umask 027`
- ✅ **Supply-chain integrity** — Alpine builder pinned by digest, CI actions pinned by commit SHA
- ✅ **No package cache left behind** — apk runs with `--no-cache`: neither index nor downloaded
  package survives in the image
- ✅ **Service managers neutralised** (`systemd`, `upstart`) so packages do not try to start daemons
- ✅ **Character encoding and timezone set** (`C.UTF-8`, `UTC`) — musl implements no locales
- ✅ **SBOM, SLSA provenance and a Cosign signature** attached to every published image
- ✅ **Continuously verified** — hadolint, 10 container integration tests run on **both
  architectures**, weekly Trivy scans

---

## Image reference

### Registries and tags

| Registry | Image | Architectures |
|---|---|---|
| Docker Hub | `samtechlab/alpine-3.24:latest` | amd64 + arm64 |
| Docker Hub | `samtechlab/alpine-3.24:YYYY.MM` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-oss/alpine-3.24:latest` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-oss/alpine-3.24:YYYY.MM` | amd64 + arm64 |

Tags point at a multi-architecture manifest — Docker automatically selects the right image for
the host platform. `latest` tracks the monthly rebuild.

**Neither tag is immutable.** A `YYYY.MM` tag names a month, not one specific build: any
build run during that month republishes it.

For a genuinely fixed image, **pin by digest** — see
[Verifying what you are running](#verifying-what-you-are-running).

### Included packages

| Category | Packages |
|---|---|
| Shell & base | `bash`, `coreutils`, `curl`, `findutils`, `jq`, `netcat-openbsd`, `tzdata` |
| System support | `alpine-release`, `ca-certificates`, `shadow` |

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `HOME` | `/config` | Home directory of `appuser` |
| `TZ` | `UTC` | Timezone |
| `LANG` | `C.UTF-8` | Character encoding, also `LC_ALL` — musl implements no locales |
| `TERM` | `xterm` | Terminal type |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Standard system path |

> **`PUID` and `PGID` are build arguments, not runtime settings.** They are baked in when
> `appuser` is created (`1000:1000`), and nothing applies them at container start — this image has
> no init system. Passing `-e PUID=1001` to `docker run` has **no effect**. To use different IDs:
>
> ```bash
> docker buildx build -f Dockerfile-multi-arch \
>   --build-arg PUID=1001 --build-arg PGID=1001 -t my-3.24 .
> ```
>
> …or align permissions on the host side instead (see [Troubleshooting](#troubleshooting)).

### Filesystem layout

| Path | Purpose |
|---|---|
| `/config` | Home of `appuser`, mode `750`, owned by `appuser` — mount your persistent data here |

### Default behaviour

| Setting | Value |
|---|---|
| User | `appuser` (UID `1000`, GID `1000`) |
| Shell | `SHELL ["/bin/bash", "-c"]` |
| Command | `CMD ["/bin/bash"]` |
| Entrypoint | none |

---

## Getting started

### Run a container

```bash
docker run -it --rm ghcr.io/sam-tech-lab-oss/alpine-3.24:latest
```

You get a `bash` shell as `appuser`. There is no root process in the container.

### Build your own image

The image ends with `USER appuser`, so **any derived image must switch back to `root` to install
packages**, then drop privileges again:

```dockerfile
FROM ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

USER root
RUN apk add --no-cache nginx
USER appuser

CMD ["nginx", "-g", "daemon off;"]
```

> **Do not install packages at container start** (e.g. an `apk add` in `command:`).
> The container runs unprivileged, so APT fails with `Permission denied` on
> `/var/lib/apt/lists`. Install at build time, as above.

### Running processes and signals

The image has no init system: your `CMD` runs directly as PID 1. That is fine for a single
well-behaved process, but PID 1 has special duties in Linux — it must reap orphaned child
processes and handle signals itself. A process that does neither leaves zombies behind, or ignores
`SIGTERM` so `docker stop` has to wait for its timeout and kill it.

If your process is not designed for that role, let Docker supply a minimal init:

```bash
docker run --init ghcr.io/sam-tech-lab-oss/alpine-3.24:latest your-command
```

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-oss/alpine-3.24:latest
    init: true
```

If you need to supervise **several** processes in one container, a plain `CMD` is not enough —
use a real init system, or split them into separate containers.

---

## Complete example NGINX

A complete, working derived image. Since the container runs unprivileged, NGINX cannot bind a
port below 1024 and cannot write to its usual runtime locations — hence the adjustments below.

`Dockerfile`
```dockerfile
FROM ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

USER root
RUN apk add --no-cache nginx && \
    # Listen on an unprivileged port (IPv4 and IPv6 lines both end in
    # "80 default_server;", so only that suffix is replaced).
    sed -i 's/80 default_server;/8080 default_server;/g' /etc/nginx/sites-enabled/default && \
    # The "user" directive only applies when the master runs as root.
    sed -i '/^user /d' /etc/nginx/nginx.conf && \
    # /run is not writable by appuser.
    sed -i 's#pid /run/nginx.pid;#pid /tmp/nginx.pid;#' /etc/nginx/nginx.conf && \
    # Send logs to the container's stdout/stderr.
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    # Cache and temp directories must be writable by appuser.
    chown -R appuser:appuser /var/lib/nginx
USER appuser

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

`docker-compose.yml`
```yaml
services:
  web:
    build: .
    container_name: nginx-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./html:/var/www/html
    environment:
      TZ: "Europe/Paris"
```

```bash
mkdir -p html && echo '<h1>It works</h1>' > html/index.html
docker compose up -d --build
curl http://localhost:8080
```

> A bind mount **replaces** the directory's contents in the image. Mounting an empty `./html`
> over `/var/www/html` removes NGINX's default page, and NGINX answers `403 Forbidden` because it
> has no `index.html` to serve. Put a file there first, as above.

---

## Security model

The container has **no privileged process**: the image ends with `USER appuser`, so everything —
including PID 1 — runs unprivileged.

| Control | Implementation |
|---|---|
| Container user | `appuser` (UID `1000`), set with `USER` — no root process |
| `root` account | Password locked (`passwd -l`), `/root` mode `700` |
| Login shell for `appuser` | `/usr/sbin/nologin` |
| SUID/SGID binaries | Stripped image-wide at build time |
| World-writable files | Write bit removed image-wide at build time |
| Default umask | `027` |
| `/config` | Mode `750`, owned by `appuser` |
| Service managers | `policy-rc.d` and `initctl` neutralised |
| Supply chain | Alpine builder pinned by digest; CI actions pinned by commit SHA |

Recommended runtime hardening for your deployments:

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-oss/alpine-3.24:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

### Base image support status

Alpine 3.24 receives security updates until **1 June 2028**, for both the `main` and the
`community` repository.

Alpine cuts a new release branch from edge every May and November and supports `main` for
about two years. `community` is supported only until the next stable release — for 3.24
that means until 3.25 ships, expected November 2026. Before relying on a package — one this
image ships or one you add on top — check which repository it comes from:
`apk info -a <package>` names its origin.

Monthly rebuilds pick up the security updates Alpine publishes for 3.24: a fix released
upstream reaches this image at the next rebuild, or immediately if you rebuild it yourself.

Vulnerability scans report findings that have no fix available alongside those that do, so
the published reports reflect the full exposure of the image rather than only what is
actionable today.

### Verifying what you are running

Every image carries OCI provenance labels — the exact commit it was built from, and when:

```bash
docker inspect --format '{{json .Config.Labels}}' \
  ghcr.io/sam-tech-lab-oss/alpine-3.24:latest | jq
```

Pin by digest to guarantee byte-for-byte reproducibility:

```bash
docker pull ghcr.io/sam-tech-lab-oss/alpine-3.24@sha256:<digest>
```

Each published image is signed. The signature establishes that the image was built by this
repository's publish workflow and has not been replaced since — something the SBOM and the
provenance attestation, on their own, do not show. Verify it with
[Cosign](https://docs.sigstore.dev/cosign/system_config/installation/):

```bash
cosign verify \
  --certificate-identity-regexp '^https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/\.github/workflows/' \
  --certificate-oidc-issuer 'https://token.actions.githubusercontent.com' \
  ghcr.io/sam-tech-lab-oss/alpine-3.24@sha256:<digest>
```

Both flags matter: without them Cosign only confirms that *a* signature exists, not that it is
this repository's. Anyone can sign any public image.

Vulnerability reporting is covered in [`SECURITY.md`](./SECURITY.md).

---

## Troubleshooting

### Getting a shell inside a running container

```bash
docker exec -it <container> bash          # as appuser
docker exec -it -u 0 <container> bash     # as root, for debugging
docker logs <container>
```

### Common problems

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
APT is running unprivileged. Install packages at build time in your `Dockerfile` — and remember to
`USER root` before the `RUN`, then `USER appuser` after.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Unprivileged processes cannot bind ports below 1024. Use a port ≥ 1024 inside the container and
remap it on the host (`-p 80:8080`).

**Setting `-e PUID=…` changes nothing**
Expected: `appuser` has a fixed UID of `1000` baked in at build time. Either rebuild with the IDs
you need, or adjust ownership on the host: `sudo chown -R 1000:1000 ./data`.

**Files created in a bind mount are owned by UID 1000 on the host**
That is `appuser`. Either run your host tooling under a matching UID, or `chown` the directory to
your own user after the fact.

**`Permission denied` writing to a mounted volume**
The host directory is not writable by UID `1000`. Fix it on the host:
`sudo chown -R 1000:1000 ./data`.

**`docker stop` takes ~10 seconds**
Your `CMD` is running as PID 1 and ignoring `SIGTERM`. Run it with `--init` (or `init: true` in
Compose) so a proper init forwards signals for it.

**Zombie processes accumulate**
Same cause — PID 1 is not reaping children. Use `--init`.

---

## Maintenance

- **Images are rebuilt monthly** (1st of the month, 03:00 UTC) against the current 3.24
  archive, and can be triggered manually from the Actions tab. See
  [Base image support status](#base-image-support-status) for what that does and does not cover.
- **Vulnerabilities are scanned weekly** (Mondays, 04:00 UTC) and after every build that published
  an image, with Trivy, on both architectures. Findings with no fix available are included. Full
  JSON reports are kept as build artifacts for 90 days, and every run writes a summary table to
  its workflow page. Results also go to the **Security → Code scanning** tab, which requires code
  scanning to be enabled in *Settings → Code security*; when it is not, the scan still runs and
  says so in its summary.
- **Every pull request runs hadolint and the integration tests** on amd64 and arm64. Publishing
  waits on both, and never happens from a pull request.
- **The Docker Hub description** is a separate file, `README-dockerhub.md`, synchronised by its
  own workflow when that file changes.

Source: [`Dockerfile-multi-arch`](./Dockerfile-multi-arch).
Contributions are welcome: see [`CONTRIBUTING.md`](./CONTRIBUTING.md) and the
[`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md).

---
---

# Documentation française

<p align="center">
  <a href="#docker-alpine-324---sam-tech-lab">English version ↑</a> · <b>Français</b>
</p>

## Démarrage rapide

```bash
# Lancer un shell (en tant qu'appuser, non privilégié)
docker run -it --rm ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

# Ou depuis Docker Hub
docker run -it --rm samtechlab/alpine-3.24:latest
```

Construire par-dessus :

```dockerfile
FROM ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

# L'image tourne en appuser : repasser en root pour installer, puis redescendre.
USER root
RUN apk add --no-cache votre-paquet
USER appuser
```

**Sommaire :** [Présentation](#présentation) · [Référence de l'image](#référence-de-limage) ·
[Prise en main](#prise-en-main) · [Exemple complet NGINX](#exemple-complet-nginx) ·
[Modèle de sécurité](#modèle-de-sécurité) · [Dépannage](#dépannage) ·
[Maintenance](#maintenance-1)

---

## Présentation

Une **image de base Alpine 3.24 minimale, durcie et multi-architecture**, construite
`FROM scratch` à partir du **rootfs OCI officiel de Alpine** — pour l'authenticité, la légèreté et
la reproductibilité.

Elle est conçue comme une **fondation pour vos propres images** : un système Alpine complet (sans noyau — les conteneurs
partagent celui de l'hôte), un
utilisateur non-root, un socle système durci — puis elle vous laisse travailler.

> **Architectures supportées :** `linux/amd64`, `linux/arm64`
> **Reconstructions mensuelles automatiques** intégrant les mises à jour de sécurité
> que Alpine publie pour 3.24.

### Points forts

- ✅ **Construite `FROM scratch`** depuis le rootfs OCI officiel Alpine — aucune couche de base tierce
- ✅ **Multi-arch** publiée sous un manifeste unique (`amd64`, `arm64`)
- ✅ **Exécution en utilisateur non-root** (`appuser`) de bout en bout — aucun processus privilégié
- ✅ **Durcissement système** — compte `root` verrouillé, bits SUID/SGID supprimés, bits
  world-writable retirés, `umask 027`
- ✅ **Intégrité de la chaîne d'approvisionnement** — builder Alpine figé par digest, actions CI
  figées par SHA de commit
- ✅ **Aucun cache de paquets résiduel** — apk tourne avec `--no-cache` : ni index ni paquet
  téléchargé ne subsiste dans l'image
- ✅ **Gestionnaires de services neutralisés** (`systemd`, `upstart`) pour que les paquets
  n'essaient pas de démarrer de daemons
- ✅ **Encodage et fuseau horaire définis** (`C.UTF-8`, `UTC`) — musl n'implémente pas les locales
- ✅ **SBOM, provenance SLSA et signature Cosign** joints à chaque image publiée
- ✅ **Vérifiée en continu** — hadolint, 10 tests d'intégration sur conteneur joués sur **les deux
  architectures**, scans Trivy hebdomadaires

---

## Référence de l'image

### Registres et tags

| Registre | Image | Architectures |
|---|---|---|
| Docker Hub | `samtechlab/alpine-3.24:latest` | amd64 + arm64 |
| Docker Hub | `samtechlab/alpine-3.24:YYYY.MM` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-oss/alpine-3.24:latest` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-oss/alpine-3.24:YYYY.MM` | amd64 + arm64 |

Les tags pointent vers un manifeste multi-architecture : Docker sélectionne automatiquement
l'image correspondant à la plateforme hôte. `latest` suit la reconstruction mensuelle.

**Aucun de ces tags n'est immuable.** Un tag `YYYY.MM` désigne un mois, pas un build en
particulier : tout build effectué durant ce mois le republie.

Pour une image réellement figée, **épinglez par digest** — voir
[Vérifier ce que vous exécutez](#vérifier-ce-que-vous-exécutez).

### Paquets inclus

| Catégorie | Paquets |
|---|---|
| Shell & base | `bash`, `coreutils`, `curl`, `findutils`, `jq`, `netcat-openbsd`, `tzdata` |
| Outils système | `alpine-release`, `ca-certificates`, `shadow` |

### Variables d'environnement

| Variable | Défaut | Description |
|---|---|---|
| `HOME` | `/config` | Répertoire personnel de `appuser` |
| `TZ` | `UTC` | Fuseau horaire |
| `LANG` | `C.UTF-8` | Encodage des caractères, également `LC_ALL` — musl n'implémente pas les locales |
| `TERM` | `xterm` | Type de terminal |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Chemin système standard |

> **`PUID` et `PGID` sont des arguments de build, pas des réglages d'exécution.** Ils sont figés
> à la création de `appuser` (`1000:1000`), et rien ne les applique au démarrage du conteneur :
> cette image n'a pas de système d'init. Passer `-e PUID=1001` à `docker run` **n'a aucun effet**.
> Pour d'autres identifiants :
>
> ```bash
> docker buildx build -f Dockerfile-multi-arch \
>   --build-arg PUID=1001 --build-arg PGID=1001 -t mon-3.24 .
> ```
>
> …ou alignez les permissions côté hôte (voir [Dépannage](#dépannage)).

### Arborescence

| Chemin | Rôle |
|---|---|
| `/config` | Home de `appuser`, mode `750`, appartenant à `appuser` — montez vos données persistantes ici |

### Comportement par défaut

| Réglage | Valeur |
|---|---|
| Utilisateur | `appuser` (UID `1000`, GID `1000`) |
| Shell | `SHELL ["/bin/bash", "-c"]` |
| Commande | `CMD ["/bin/bash"]` |
| Entrypoint | aucun |

---

## Prise en main

### Lancer un conteneur

```bash
docker run -it --rm ghcr.io/sam-tech-lab-oss/alpine-3.24:latest
```

Vous obtenez un shell `bash` en tant que `appuser`. Aucun processus root ne tourne dans le
conteneur.

### Construire votre propre image

L'image se termine par `USER appuser` : **toute image dérivée doit donc repasser en `root` pour
installer des paquets**, puis redescendre :

```dockerfile
FROM ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

USER root
RUN apk add --no-cache nginx
USER appuser

CMD ["nginx", "-g", "daemon off;"]
```

> **N'installez pas de paquets au démarrage du conteneur** (par ex. un `apk add` dans
> `command:`). Le conteneur tourne sans privilèges, APT échouera donc avec `Permission denied`
> sur `/var/lib/apt/lists`. Installez au build, comme ci-dessus.

### Processus et signaux

L'image ne contient pas de système d'init : votre `CMD` s'exécute directement en PID 1. C'est
suffisant pour un unique processus bien conçu, mais le PID 1 a des responsabilités particulières
sous Linux — il doit récupérer les processus fils orphelins et gérer lui-même les signaux. Un
processus qui ne fait ni l'un ni l'autre laisse des zombies, ou ignore `SIGTERM`, obligeant
`docker stop` à attendre son délai d'expiration avant de le tuer.

Si votre processus n'est pas prévu pour ce rôle, laissez Docker fournir un init minimal :

```bash
docker run --init ghcr.io/sam-tech-lab-oss/alpine-3.24:latest votre-commande
```

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-oss/alpine-3.24:latest
    init: true
```

Si vous devez superviser **plusieurs** processus dans un même conteneur, un simple `CMD` ne suffit
pas : utilisez un véritable système d'init, ou séparez-les en plusieurs conteneurs.

---

## Exemple complet NGINX

Une image dérivée complète et fonctionnelle. Le conteneur tournant sans privilèges, NGINX ne peut
ni se lier à un port inférieur à 1024, ni écrire dans ses emplacements d'exécution habituels —
d'où les ajustements ci-dessous.

`Dockerfile`
```dockerfile
FROM ghcr.io/sam-tech-lab-oss/alpine-3.24:latest

USER root
RUN apk add --no-cache nginx && \
    # Écouter sur un port non privilégié (les lignes IPv4 et IPv6 se
    # terminent toutes deux par "80 default_server;", seul ce suffixe
    # est remplacé).
    sed -i 's/80 default_server;/8080 default_server;/g' /etc/nginx/sites-enabled/default && \
    # La directive "user" ne s'applique que si le master tourne en root.
    sed -i '/^user /d' /etc/nginx/nginx.conf && \
    # /run n'est pas accessible en écriture par appuser.
    sed -i 's#pid /run/nginx.pid;#pid /tmp/nginx.pid;#' /etc/nginx/nginx.conf && \
    # Rediriger les logs vers stdout/stderr du conteneur.
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    # Les répertoires de cache et temporaires doivent être accessibles
    # en écriture par appuser.
    chown -R appuser:appuser /var/lib/nginx
USER appuser

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

`docker-compose.yml`
```yaml
services:
  web:
    build: .
    container_name: nginx-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./html:/var/www/html
    environment:
      TZ: "Europe/Paris"
```

```bash
mkdir -p html && echo '<h1>Ça marche</h1>' > html/index.html
docker compose up -d --build
curl http://localhost:8080
```

> Un montage de volume **remplace** le contenu du répertoire dans l'image. Monter un `./html`
> vide sur `/var/www/html` supprime la page par défaut de NGINX, et NGINX répond
> `403 Forbidden` faute d'`index.html` à servir. Créez d'abord un fichier, comme ci-dessus.

---

## Modèle de sécurité

Le conteneur n'a **aucun processus privilégié** : l'image se termine par `USER appuser`, donc tout
— y compris le PID 1 — s'exécute sans privilèges.

| Contrôle | Mise en œuvre |
|---|---|
| Utilisateur du conteneur | `appuser` (UID `1000`), défini via `USER` — aucun processus root |
| Compte `root` | Mot de passe verrouillé (`passwd -l`), `/root` en mode `700` |
| Shell de connexion de `appuser` | `/usr/sbin/nologin` |
| Binaires SUID/SGID | Supprimés sur toute l'image au build |
| Fichiers world-writable | Bit d'écriture retiré sur toute l'image au build |
| Umask par défaut | `027` |
| `/config` | Mode `750`, appartenant à `appuser` |
| Gestionnaires de services | `policy-rc.d` et `initctl` neutralisés |
| Chaîne d'approvisionnement | Builder Alpine figé par digest ; actions CI figées par SHA de commit |

Durcissement recommandé à l'exécution pour vos déploiements :

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-oss/alpine-3.24:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

### État du support de l'image de base

Alpine 3.24 reçoit des mises à jour de sécurité jusqu'au **1er juin 2028**, pour le dépôt
`main` comme pour le dépôt `community`.

Alpine ouvre une branche depuis edge chaque mai et chaque novembre, et maintient `main`
pendant environ deux ans. `community` n'est maintenu que jusqu'à la version stable
suivante — pour 3.24, jusqu'à la sortie de 3.25, attendue en novembre 2026. Avant de dépendre d'un paquet — livré par
cette image ou ajouté par-dessus — vérifiez de quel dépôt il provient :
`apk info -a <paquet>` en donne l'origine.

Les reconstructions mensuelles récupèrent les mises à jour de sécurité que Alpine publie
pour 3.24 : un correctif publié en amont atteint cette image à la reconstruction suivante,
ou immédiatement si vous la reconstruisez vous-même.

Les analyses de vulnérabilités remontent aussi bien les vulnérabilités sans correctif
disponible que celles qui en ont un : les rapports publiés reflètent l'exposition complète
de l'image, et non seulement ce sur quoi il est possible d'agir aujourd'hui.

### Vérifier ce que vous exécutez

Chaque image porte des labels de provenance OCI — le commit exact dont elle est issue, et sa date
de construction :

```bash
docker inspect --format '{{json .Config.Labels}}' \
  ghcr.io/sam-tech-lab-oss/alpine-3.24:latest | jq
```

Figez par digest pour garantir une reproductibilité au bit près :

```bash
docker pull ghcr.io/sam-tech-lab-oss/alpine-3.24@sha256:<digest>
```

Chaque image publiée est signée. La signature établit que l'image a été construite par le workflow
de publication de ce dépôt et n'a pas été remplacée depuis — ce que le SBOM et l'attestation de
provenance, seuls, ne montrent pas. Vérifiez-la avec
[Cosign](https://docs.sigstore.dev/cosign/system_config/installation/) :

```bash
cosign verify \
  --certificate-identity-regexp '^https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/\.github/workflows/' \
  --certificate-oidc-issuer 'https://token.actions.githubusercontent.com' \
  ghcr.io/sam-tech-lab-oss/alpine-3.24@sha256:<digest>
```

Les deux options comptent : sans elles, Cosign confirme seulement qu'*une* signature existe, pas
qu'elle est celle de ce dépôt. N'importe qui peut signer n'importe quelle image publique.

Le signalement de vulnérabilités est décrit dans [`SECURITY.md`](./SECURITY.md).

---

## Dépannage

### Obtenir un shell dans un conteneur en cours d'exécution

```bash
docker exec -it <conteneur> bash          # en tant qu'appuser
docker exec -it -u 0 <conteneur> bash     # en tant que root, pour déboguer
docker logs <conteneur>
```

### Problèmes courants

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
APT s'exécute sans privilèges. Installez les paquets au build dans votre `Dockerfile` — et
pensez à `USER root` avant le `RUN`, puis `USER appuser` après.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Un processus non privilégié ne peut pas se lier aux ports inférieurs à 1024. Utilisez un port
≥ 1024 dans le conteneur et remappez-le côté hôte (`-p 80:8080`).

**Passer `-e PUID=…` ne change rien**
C'est attendu : `appuser` a un UID fixe de `1000`, figé au build. Reconstruisez l'image avec les
identifiants voulus, ou ajustez la propriété côté hôte : `sudo chown -R 1000:1000 ./data`.

**Les fichiers créés dans un volume appartiennent à l'UID 1000 sur l'hôte**
C'est `appuser`. Faites tourner vos outils hôte sous un UID correspondant, ou faites un `chown`
du répertoire vers votre propre utilisateur ensuite.

**`Permission denied` en écriture sur un volume monté**
Le répertoire hôte n'est pas accessible en écriture par l'UID `1000`. Corrigez côté hôte :
`sudo chown -R 1000:1000 ./data`.

**`docker stop` met une dizaine de secondes**
Votre `CMD` tourne en PID 1 et ignore `SIGTERM`. Lancez-le avec `--init` (ou `init: true` en
Compose) pour qu'un véritable init relaie les signaux à sa place.

**Des processus zombies s'accumulent**
Même cause : le PID 1 ne récupère pas ses fils. Utilisez `--init`.

---

## Maintenance

- **Les images sont reconstruites chaque mois** (le 1er, à 03h00 UTC) à partir de l'archive
  3.24 courante, et peuvent être déclenchées manuellement depuis l'onglet Actions. Voir
  [État du support de l'image de base](#état-du-support-de-limage-de-base) pour ce que cela
  couvre — et ne couvre pas.
- **Les vulnérabilités sont scannées chaque semaine** (lundi, 04h00 UTC) et après chaque build
  ayant publié une image, avec Trivy, sur les deux architectures. Les vulnérabilités sans
  correctif disponible sont incluses. Les rapports JSON complets sont conservés 90 jours en
  artefacts de build, et chaque run écrit un tableau de synthèse sur sa page de workflow. Les
  résultats vont aussi dans l'onglet **Security → Code scanning**, ce qui suppose le code scanning
  activé dans *Settings → Code security* ; sinon l'analyse tourne quand même et le signale.
- **Chaque pull request exécute hadolint et les tests d'intégration** sur amd64 et arm64. La
  publication attend les deux, et n'a jamais lieu depuis une pull request.
- **La description Docker Hub** est un fichier distinct, `README-dockerhub.md`, synchronisé par
  son propre workflow quand ce fichier change.

Source : [`Dockerfile-multi-arch`](./Dockerfile-multi-arch).
Les contributions sont bienvenues : voir [`CONTRIBUTING.md`](./CONTRIBUTING.md) et le
[`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md).

---

## Support this work / Soutenir ce travail

These images are rebuilt every month, signed, scanned and documented. The work is done
in the open and given away — sponsoring is what keeps the schedule.

Ces images sont reconstruites chaque mois, signées, analysées et documentées. Ce travail
est mené au grand jour et mis à disposition — le parrainage est ce qui en maintient le
rythme.

[![Sponsor](https://img.shields.io/badge/Sponsor-GitHub-ea4aaa.svg?style=for-the-badge&logo=githubsponsors&logoColor=white)](https://github.com/sponsors/Sam-Tech-Lab-OSS)

---

## License / Licence

This project is distributed under the **Apache&nbsp;2.0** license — see [LICENSE](./LICENSE) for the terms, and [NOTICE](./NOTICE) for the attributions that must travel with any redistribution.

Ce projet est distribué sous la licence **Apache&nbsp;2.0** — voir [LICENSE](./LICENSE) pour les termes, et [NOTICE](./NOTICE) pour les attributions qui doivent accompagner toute redistribution.

---

## Copyright / Droit d'auteur

```text
Copyright (c) 2025 Sam Tech Lab
```
