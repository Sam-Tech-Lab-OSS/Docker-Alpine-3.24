# Alpine 3.24 — base image

Minimal, hardened, multi-architecture **Alpine 3.24** base image, built `FROM
scratch` from the official Alpine OCI rootfs. A complete Alpine userland, a non-root user, a
hardened baseline — then it gets out of your way.

**Full documentation, in English and French:**
<https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24>

*Version française plus bas.*

---

## Base image support

Alpine 3.24 receives security updates until **1 June 2028**, for both the `main` and the
`community` repository.

Alpine cuts a new release branch from edge every May and November and supports `main` for
about two years. `community` is supported only until the next stable release — for 3.24
that means until 3.25 ships, expected November 2026. Before relying on a package — one this
image ships or one you add on top — check which repository it comes from:
`apk info -a <package>` names its origin.

Monthly rebuilds pick up the security updates Alpine publishes for 3.24. Published
vulnerability scans report findings with no fix available alongside those that have one, so
the reports reflect the full exposure of the image.

---

## Tags

| Tag | Contents |
|---|---|
| `latest` | Tracks the monthly rebuild — amd64 + arm64 |
| `YYYY.MM` (e.g. `2026.08`) | The build from that month — amd64 + arm64 |

Tags point at a multi-architecture manifest; Docker selects the right image for the host platform.

**Neither tag is immutable.** `YYYY.MM` names the month, not one specific build: any build during
that month republishes it. For a genuinely fixed image, pin by digest:

```bash
docker pull samtechlab/alpine-3.24@sha256:<digest>
```

Each published image is signed. The signature establishes that the image was built by this
repository's publish workflow and has not been replaced since — something the SBOM and the
provenance attestation, on their own, do not show. Verify it with
[Cosign](https://docs.sigstore.dev/cosign/system_config/installation/):

```bash
cosign verify \
  --certificate-identity-regexp '^https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/\.github/workflows/' \
  --certificate-oidc-issuer 'https://token.actions.githubusercontent.com' \
  samtechlab/alpine-3.24@sha256:<digest>
```

Both flags matter: without them Cosign only confirms that *a* signature exists, not that it is
this repository's. Anyone can sign any public image.

Also published on GHCR as `ghcr.io/sam-tech-lab-oss/alpine-3.24`.

---

## Quick start

```bash
# Shell as the unprivileged appuser
docker run -it --rm samtechlab/alpine-3.24:latest

# Check who you are
docker run --rm samtechlab/alpine-3.24:latest id
```

Build on top of it. The image ends with `USER appuser`, so switch to root to install, then switch
back:

```dockerfile
FROM samtechlab/alpine-3.24:latest

USER root
RUN apk add --no-cache your-package
USER appuser
```

---

## No init system — your CMD is PID 1

This image ships no init system: your `CMD` runs directly as PID 1. That is fine for a single
well-behaved process, but PID 1 has special duties on Linux — it must reap orphaned children and
handle signals itself. A process that does neither leaves zombies behind, or ignores `SIGTERM` so
`docker stop` waits out its timeout and kills it.

If your process is not designed for that role, let Docker supply a minimal init:

```bash
docker run --init samtechlab/alpine-3.24:latest your-command
```

```yaml
services:
  app:
    image: samtechlab/alpine-3.24:latest
    init: true
```

To supervise **several** processes in one container, a plain `CMD` is not enough: use a real init
system, or split them into separate containers. The
[s6-overlay variant](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24-S6-Overlay)
of this same 3.24 base covers that case.

---

## Key features

- Built `FROM scratch` from the official Alpine OCI rootfs — no third-party base layer
- Published as a single multi-arch manifest (`amd64`, `arm64`)
- **Runs as a non-root user (`appuser`) end to end** — there is no privileged process, PID 1 included
- **Hardening** — `root` locked, SUID/SGID stripped, world-writable bits removed, `umask 027`
- **Service managers neutralised** (`policy-rc.d`, `initctl`) so packages do not try to start daemons
- **Supply-chain integrity** — Alpine builder pinned by digest, CI actions pinned by commit SHA,
  SBOM, SLSA provenance and a Cosign signature attached to every image
- No package cache left behind — apk runs with `--no-cache`
- Character encoding and timezone set (`C.UTF-8`, `UTC`) — musl implements no locales
- Continuously verified — hadolint, 10 container integration tests on both architectures, weekly
  Trivy scans

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `HOME` | `/config` | Home directory of `appuser` |
| `TZ` | `UTC` | Timezone |
| `LANG` | `C.UTF-8` | Character encoding, also `LC_ALL` — musl implements no locales |
| `TERM` | `xterm` | Terminal type |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Standard system path |

**`PUID` and `PGID` are build-time arguments, not runtime settings.** `appuser` is created at
`1000:1000` when the image is built, and nothing applies them at container start — this image has
no init system. Passing `-e PUID=1001` to `docker run` has **no effect**. To use different IDs,
rebuild with `--build-arg PUID=1001 --build-arg PGID=1001`, or align permissions host-side.

---

## Filesystem and defaults

| Path | Purpose |
|---|---|
| `/config` | Home of `appuser`, mode `750`, owned by `appuser` — mount your persistent data here |

| Setting | Value |
|---|---|
| User | `appuser` (UID `1000`, GID `1000`) |
| Command | `CMD ["/bin/bash"]` |
| Entrypoint | none |

---

## Security model

The container has **no privileged process**: the image ends with `USER appuser`, so everything —
including PID 1 — runs unprivileged.

| Control | Implementation |
|---|---|
| Container user | `appuser` (UID `1000`), set with `USER` — no root process |
| `root` account | Password locked, `/root` mode `700` |
| Login shell for `appuser` | `/usr/sbin/nologin` |
| SUID/SGID binaries | Stripped image-wide at build time |
| World-writable files | Write bit removed image-wide at build time |
| Default umask | `027` |
| `/config` | Mode `750`, owned by `appuser` |
| Service managers | `policy-rc.d` and `initctl` neutralised |
| Supply chain | Alpine builder pinned by digest; CI actions pinned by commit SHA; SBOM, provenance and signature published |

Recommended runtime hardening:

```yaml
services:
  app:
    image: samtechlab/alpine-3.24:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

Vulnerability reporting:
[SECURITY.md](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/SECURITY.md)

---

## Troubleshooting

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
The image runs as `appuser`. Switch to `USER root` in your Dockerfile to install packages, then
back to `USER appuser`.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Unprivileged processes cannot bind ports below 1024. Use a port ≥ 1024 inside the container and
remap it on the host (`-p 80:8080`).

**`docker stop` takes ~10 seconds**
Your process is not handling `SIGTERM`, or is not reaping children as PID 1. Run it with
`--init`, or handle signals in the process itself.

**Files created in a mounted volume have the wrong owner**
`appuser` is fixed at `1000:1000`. Either `chown` the host directory to `1000:1000`, or rebuild
with `--build-arg PUID=… --build-arg PGID=…`.

**Zombie processes accumulate**
Your PID 1 is not reaping orphans. Use `--init`.

More entries in the
[full documentation](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24#troubleshooting).

---

## Support this work

These images are rebuilt every month, signed, scanned and documented. The work is done
in the open and given away — sponsoring is what keeps the schedule.

[![Sponsor](https://img.shields.io/badge/Sponsor-GitHub-ea4aaa.svg?style=for-the-badge&logo=githubsponsors&logoColor=white)](https://github.com/sponsors/Sam-Tech-Lab-OSS)

---

## License

Apache 2.0 — see
[LICENSE](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/LICENSE) and
[NOTICE](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/NOTICE).
Copyright (c) 2026 Sam Tech Lab.

---
---

# Alpine 3.24 — image de base

Image de base **Alpine 3.24** minimale, durcie et multi-architecture, construite
`FROM scratch` à partir du rootfs OCI officiel de Alpine. Un userland Alpine complet, un
utilisateur non-root, un socle durci — puis elle vous laisse travailler.

**Documentation complète, en anglais et en français :**
<https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24>

---

## Support de l'image de base

Alpine 3.24 reçoit des mises à jour de sécurité jusqu'au **1er juin 2028**, pour le dépôt
`main` comme pour le dépôt `community`.

Alpine ouvre une branche depuis edge chaque mai et chaque novembre, et maintient `main`
pendant environ deux ans. `community` n'est maintenu que jusqu'à la version stable
suivante — pour 3.24, jusqu'à la sortie de 3.25, attendue en novembre 2026. Avant de dépendre d'un paquet — livré par
cette image ou ajouté par-dessus — vérifiez de quel dépôt il provient :
`apk info -a <paquet>` en donne l'origine.

Les reconstructions mensuelles récupèrent les mises à jour de sécurité que Alpine publie pour
3.24. Les analyses de vulnérabilités publiées remontent aussi bien les vulnérabilités sans
correctif disponible que celles qui en ont un : les rapports reflètent l'exposition complète
de l'image.

---

## Tags

| Tag | Contenu |
|---|---|
| `latest` | Suit la reconstruction mensuelle — amd64 + arm64 |
| `YYYY.MM` (par ex. `2026.08`) | Le build de ce mois-là — amd64 + arm64 |

Les tags pointent vers un manifeste multi-architecture : Docker sélectionne l'image correspondant
à la plateforme hôte.

**Aucun de ces tags n'est immuable.** `YYYY.MM` désigne le mois, pas un build en particulier :
tout build de ce mois-là le republie. Pour une image réellement figée, épinglez par digest :

```bash
docker pull samtechlab/alpine-3.24@sha256:<digest>
```

Chaque image publiée est signée. La signature établit que l'image a été construite par le workflow
de publication de ce dépôt et n'a pas été remplacée depuis — ce que le SBOM et l'attestation de
provenance, seuls, ne montrent pas. Vérifiez-la avec
[Cosign](https://docs.sigstore.dev/cosign/system_config/installation/) :

```bash
cosign verify \
  --certificate-identity-regexp '^https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/\.github/workflows/' \
  --certificate-oidc-issuer 'https://token.actions.githubusercontent.com' \
  samtechlab/alpine-3.24@sha256:<digest>
```

Les deux options comptent : sans elles, Cosign confirme seulement qu'*une* signature existe, pas
qu'elle est celle de ce dépôt. N'importe qui peut signer n'importe quelle image publique.

Également publiée sur GHCR : `ghcr.io/sam-tech-lab-oss/alpine-3.24`.

---

## Démarrage rapide

```bash
# Shell en tant qu'appuser, non privilégié
docker run -it --rm samtechlab/alpine-3.24:latest

# Vérifier l'utilisateur effectif
docker run --rm samtechlab/alpine-3.24:latest id
```

Construire par-dessus. L'image se termine par `USER appuser` : repassez en root pour installer,
puis redescendez :

```dockerfile
FROM samtechlab/alpine-3.24:latest

USER root
RUN apk add --no-cache votre-paquet
USER appuser
```

---

## Pas de système d'init — votre CMD est PID 1

Cette image ne fournit aucun système d'init : votre `CMD` s'exécute directement en tant que
PID 1. C'est adapté à un processus unique et bien élevé, mais PID 1 a des devoirs particuliers
sous Linux — il doit récupérer les processus orphelins et gérer lui-même les signaux. Un processus
qui ne fait ni l'un ni l'autre laisse des zombies, ou ignore `SIGTERM` et oblige `docker stop` à
attendre son délai avant de le tuer.

Si votre processus n'est pas conçu pour ce rôle, laissez Docker fournir un init minimal :

```bash
docker run --init samtechlab/alpine-3.24:latest votre-commande
```

```yaml
services:
  app:
    image: samtechlab/alpine-3.24:latest
    init: true
```

Pour superviser **plusieurs** processus dans un même conteneur, un simple `CMD` ne suffit pas :
utilisez un vrai système d'init, ou séparez-les en conteneurs distincts. La
[variante s6-overlay](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24-S6-Overlay)
de la même base 3.24 couvre ce cas.

---

## Points forts

- Construite `FROM scratch` depuis le rootfs OCI officiel Alpine — aucune couche de base tierce
- Publiée comme un manifeste multi-architecture unique (`amd64`, `arm64`)
- **Tourne en utilisateur non-root (`appuser`) de bout en bout** — aucun processus privilégié,
  PID 1 compris
- **Durcissement** — `root` verrouillé, bits SUID/SGID supprimés, bits world-writable retirés,
  `umask 027`
- **Gestionnaires de services neutralisés** (`policy-rc.d`, `initctl`) : les paquets n'essaient
  pas de démarrer de daemons
- **Intégrité de la chaîne d'approvisionnement** — builder Alpine figé par digest, actions CI
  figées par SHA de commit, SBOM, provenance SLSA et signature Cosign joints à chaque image
- Aucun cache de paquets résiduel — apk tourne avec `--no-cache`
- Encodage et fuseau horaire définis (`C.UTF-8`, `UTC`) — musl n'implémente pas les locales
- Vérifiée en continu — hadolint, 10 tests d'intégration sur les deux architectures, scans Trivy
  hebdomadaires

---

## Variables d'environnement

| Variable | Défaut | Description |
|---|---|---|
| `HOME` | `/config` | Répertoire personnel de `appuser` |
| `TZ` | `UTC` | Fuseau horaire |
| `LANG` | `C.UTF-8` | Encodage des caractères, également `LC_ALL` — musl n'implémente pas les locales |
| `TERM` | `xterm` | Type de terminal |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Chemin système standard |

**`PUID` et `PGID` sont des arguments de build, pas des réglages d'exécution.** `appuser` est créé
en `1000:1000` à la construction de l'image, et rien ne les applique au démarrage du conteneur —
cette image n'a pas de système d'init. Passer `-e PUID=1001` à `docker run` n'a **aucun effet**.
Pour d'autres identifiants, reconstruisez avec `--build-arg PUID=1001 --build-arg PGID=1001`, ou
alignez les permissions côté hôte.

---

## Arborescence et valeurs par défaut

| Chemin | Rôle |
|---|---|
| `/config` | Home de `appuser`, mode `750`, lui appartenant — montez vos données persistantes ici |

| Réglage | Valeur |
|---|---|
| Utilisateur | `appuser` (UID `1000`, GID `1000`) |
| Commande | `CMD ["/bin/bash"]` |
| Entrypoint | aucun |

---

## Modèle de sécurité

Le conteneur n'a **aucun processus privilégié** : l'image se termine par `USER appuser`, donc tout
— y compris PID 1 — s'exécute sans privilèges.

| Contrôle | Mise en œuvre |
|---|---|
| Utilisateur du conteneur | `appuser` (UID `1000`), défini par `USER` — aucun processus root |
| Compte `root` | Mot de passe verrouillé, `/root` en mode `700` |
| Shell de connexion d'`appuser` | `/usr/sbin/nologin` |
| Binaires SUID/SGID | Supprimés sur toute l'image au build |
| Fichiers world-writable | Bit d'écriture retiré sur toute l'image au build |
| Umask par défaut | `027` |
| `/config` | Mode `750`, appartenant à `appuser` |
| Gestionnaires de services | `policy-rc.d` et `initctl` neutralisés |
| Chaîne d'approvisionnement | Builder Alpine figé par digest ; actions CI figées par SHA ; SBOM, provenance et signature publiés |

Durcissement recommandé à l'exécution :

```yaml
services:
  app:
    image: samtechlab/alpine-3.24:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

Signalement de vulnérabilité :
[SECURITY.md](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/SECURITY.md)

---

## Dépannage

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
L'image tourne en `appuser`. Passez en `USER root` dans votre Dockerfile pour installer des
paquets, puis revenez à `USER appuser`.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Un processus non privilégié ne peut pas écouter sous le port 1024. Utilisez un port ≥ 1024 dans
le conteneur et remappez-le côté hôte (`-p 80:8080`).

**`docker stop` traîne une dizaine de secondes**
Votre processus ne gère pas `SIGTERM`, ou ne récupère pas ses enfants en tant que PID 1. Lancez-le
avec `--init`, ou gérez les signaux dans le processus lui-même.

**Les fichiers créés dans un volume monté ont le mauvais propriétaire**
`appuser` est figé à `1000:1000`. Soit vous faites un `chown` du répertoire hôte vers `1000:1000`,
soit vous reconstruisez avec `--build-arg PUID=… --build-arg PGID=…`.

**Des processus zombies s'accumulent**
Votre PID 1 ne récupère pas les orphelins. Utilisez `--init`.

D'autres entrées dans la
[documentation complète](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24#dépannage).

---

## Soutenir ce travail

Ces images sont reconstruites chaque mois, signées, analysées et documentées. Ce travail
est mené au grand jour et mis à disposition — le parrainage est ce qui en maintient le
rythme.

[![Sponsor](https://img.shields.io/badge/Sponsor-GitHub-ea4aaa.svg?style=for-the-badge&logo=githubsponsors&logoColor=white)](https://github.com/sponsors/Sam-Tech-Lab-OSS)

---

## Licence

Apache 2.0 — voir
[LICENSE](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/LICENSE) et
[NOTICE](https://github.com/Sam-Tech-Lab-OSS/Docker-Alpine-3.24/blob/main/NOTICE).
Copyright (c) 2026 Sam Tech Lab.
