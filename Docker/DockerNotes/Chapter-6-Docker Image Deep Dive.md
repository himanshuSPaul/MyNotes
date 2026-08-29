### Milestone 4: Docker Images Deep Dive

---
📍 **Milestone 4 of 20** | 🟢 **Beginner** | ⏱ **Estimated Time: 3–4 hours** | **Prerequisites:** Milestones 1–3
---

<a id="m4-theory"></a>
#### Theory

You've been *using* images. Now you'll understand what they actually are: stacks of read-only filesystem layers, addressed by cryptographic hashes, distributed through registries, and cached with per-layer granularity. Everything about how images build, ship, and store efficiently comes from this layer model.

##### What Is an Image, Precisely?

A Docker image is a bundle of two things:

1. **A stack of read-only layers.** Each layer is a tarball of files representing a *change* to the filesystem — files added, modified, or marked as deleted. Layers stack from bottom (base) to top (most recent), and Docker's storage driver (`overlay2` on Linux) merges them into a single unified view at runtime using an **overlay filesystem**.
2. **A manifest (JSON metadata).** This describes the layer stack, the default command to run (`CMD`/`ENTRYPOINT`), exposed ports, environment variables, working directory, labels, and the target CPU architecture and OS.

When you run a container, Docker adds a *writable* layer on top of the read-only stack. Writes go into that writable layer via **copy-on-write** — reading a file traverses layers top-down until found; writing to a file copies it up into the writable layer first. When the container is deleted, the writable layer is discarded — that's why containers are "ephemeral" and why anything you want to survive belongs in a volume (Milestone 7).

##### Analogy — Layers Are Like Transparent Overlay Sheets

Imagine you're printing a color picture on a transparency machine that only prints one color per pass. You start with a blank sheet (the base), pass 1 adds cyan, pass 2 adds magenta on top, pass 3 adds yellow, pass 4 adds the black outlines. When you hold the finished stack up to the light, you see the merged image — but each sheet by itself is separate and reusable.

Docker images work the same way: each Dockerfile instruction produces one transparent sheet. If two images share the first three sheets (say, both are built from `python:3.12-slim`), Docker keeps only *one* physical copy of those sheets on disk and both images just reference them. That is why pulling a second Python image is much faster than the first — most of the layers are already cached.

##### Layered Filesystem — ASCII Diagram

```
<!-- CODE -->
Container "web-1"                Container "web-2"
    │                                 │
    ▼                                 ▼
┌──────────────────────┐         ┌──────────────────────┐
│ WRITABLE LAYER (RW)  │  ← per  │ WRITABLE LAYER (RW)  │
└──────────────────────┘  container└──────────────────────┘
┌──────────────────────┐         ┌──────────────────────┐
│ Layer 4: CMD nginx   │◄────────►│ Layer 4: CMD nginx  │  (same layer, shared)
├──────────────────────┤         ├──────────────────────┤
│ Layer 3: COPY config │◄────────►│ Layer 3: COPY config│  (same layer, shared)
├──────────────────────┤         ├──────────────────────┤
│ Layer 2: RUN apt inst│◄────────►│ Layer 2: RUN apt ins│  (same layer, shared)
├──────────────────────┤         ├──────────────────────┤
│ Layer 1: FROM debian │◄────────►│ Layer 1: FROM debian│  (same layer, shared)
└──────────────────────┘         └──────────────────────┘
   IMAGE "nginx:custom"            IMAGE "nginx:custom"
   (image is read-only,               (same image, two
    a stack of shared layers)          separate containers)
```

##### Image Caching — The Rule Nobody Forgets Twice

Docker builds and pulls layers using a **content-addressable cache** keyed on the *content hash* of each layer plus the *instruction that produced it*. When you rebuild an image and step 4 changes, steps 1–3 hit the cache and steps 4 onward rebuild. This is why the order of instructions in a Dockerfile matters enormously — you put slow, rarely changing steps early (installing OS packages) and fast, frequently changing steps late (copying application source). We'll drill this hard in Milestone 5 & 6. For now the key fact is: **layers are content-addressed and cached, and one small change invalidates every layer after it**.

##### Image Digests (SHA256) and Immutability

Every image and every layer is identified by a **digest** — a SHA256 hash of its content. You see them everywhere:

```
Digest: sha256:2140dad235c1dd3f4dc3ca35c7c8ca7b91c8f7c0a...
```

Digests are **immutable**. If two people around the world pull `nginx@sha256:2140dad...`, they are guaranteed byte-for-byte identical bits — the hash function guarantees it. This is why production deployments should pin to digests (`nginx@sha256:...`) rather than tags (`nginx:1.27`), because tags can be republished silently but digests cannot.

##### Tags — Human Names for Digests

A **tag** is a mutable label that points to a digest. `nginx:1.27` today points to digest A; next week the Nginx team publishes 1.27.2 and republishes the tag pointing to digest B. Your image is fine on disk, but a fresh pull would grab B. This is why `nginx:latest` is dangerous in production — it moves every time upstream releases.

The full "image reference" format is:

```
[REGISTRY[:PORT]/]REPOSITORY[:TAG][@DIGEST]
```

Examples:

- `nginx` → `docker.io/library/nginx:latest`
- `nginx:1.27-alpine` → same registry, specific tag
- `ghcr.io/myorg/api:v2.3.1` → GitHub Container Registry
- `myregistry.corp:5000/team/tool:2024-11-05` → private registry on port 5000
- `nginx@sha256:2140dad235c1...` → tag omitted, digest specified

If you don't specify a registry, Docker defaults to `docker.io` (Docker Hub). If you don't specify a tag, Docker defaults to `:latest`. If you specify both a tag and a digest, the digest wins.

##### The `latest` Tag Is Just a Convention

There is no "latest" magic in Docker — `latest` is simply the default tag assigned when no tag is given at push time. Some projects genuinely publish `latest` pointing to their most recent release; others do not. Never assume `latest` means "current" — always check what tag a project actually publishes. In production, always pin to a specific version.

##### Semantic Versioning in Image Tags

Well-maintained projects follow **semantic versioning (semver)** in their tags:

- `nginx:1.27.2` — exact patch version
- `nginx:1.27` — latest 1.27.x
- `nginx:1` — latest 1.x.x
- `nginx:latest` — latest 1.x.x for stable releases; sometimes latest mainline
- `nginx:stable`, `nginx:mainline`, `nginx:perl` — flavor tags
- `nginx:1.27-alpine`, `nginx:1.27-slim` — same version, thinner base image

Choosing `nginx:1.27` gives you patch updates automatically on re-pull, which is usually what you want in staging; `nginx:1.27.2` pins you to an exact build for production reproducibility.

##### Docker Hub — The Default Registry

**Docker Hub** (hub.docker.com) is the default public registry. It hosts millions of images in three main "spaces":

- **Official Images** — reviewed and maintained by Docker in cooperation with upstream projects. Namespace is `library/` (usually omitted). Examples: `nginx`, `python`, `postgres`, `alpine`, `debian`. Trustworthy.
- **Verified Publishers** — commercial vendors verified by Docker Inc. Examples: `bitnami/`, `cimg/` (CircleCI). Blue "Verified Publisher" badge.
- **Community Images** — anyone with a Docker Hub account. Namespace is `username/repo`. Quality ranges from excellent to abandoned. Always audit.

Docker Hub imposes **pull rate limits** on anonymous users (~100 pulls per 6 hours per IP), free logged-in users (~200 pulls per 6 hours), and higher limits for paid accounts. In corporate environments where many developers share a NAT'd egress IP, anonymous limits are frequently hit — that's a common cause of mysterious `too many requests` errors.

##### Identifying Trustworthy Images

When picking an image, look for:

1. **Official** badge (from Docker Hub), or a Verified Publisher badge.
2. **Last updated** date — abandoned images are a security risk.
3. **Pull count** — orders of magnitude matter. 10M+ pulls suggests significant use.
4. **Star count** — reasonable proxy for community trust.
5. **Documented Dockerfile** — a link to the source of the Dockerfile so you can inspect what actually goes into the image.
6. **CVE scan results** — Docker Scout, Trivy, or Snyk reports (Milestone 12 covers scanning).

For anything running in production, prefer official images or images from a Verified Publisher. For untrusted community images, always read the Dockerfile before pulling.

##### Docker Content Trust (DCT)

**Docker Content Trust** is an opt-in feature that uses cryptographic signatures to verify image integrity and publisher identity. Enable it by setting the environment variable:

```powershell
$env:DOCKER_CONTENT_TRUST = "1"
```

With DCT on, `docker pull` and `docker push` reject unsigned images. It's built on **The Update Framework (TUF)** and used mostly by regulated industries and hardened corporate environments. In practice most teams instead rely on **image digest pinning** and **Sigstore/cosign** signatures (newer standard). DCT is worth knowing but not universally deployed.

<a id="m4-commands"></a>
#### Commands

- `docker pull` — download an image (covered in Milestone 3; revisited here for tags/digests).
- `docker images` / `docker image ls` — list images (covered in Milestone 3; revisited here with filters).
- `docker image inspect` — print full JSON metadata for an image.
- `docker history` — show the layer history of an image.
- `docker tag` — create a new tag pointing to an existing image.
- `docker search` — search Docker Hub for images from the CLI.
- `docker push` — upload a tagged image to a registry.
- `docker login` / `docker logout` — authenticate to a registry.
- `docker image prune` — remove dangling/unused images.
- `docker image save` / `docker image load` — export/import images as tarballs.

<a id="m4-flags"></a>
#### Command Options/Flags

##### `docker image inspect` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--format` | `-f` | Go template for output | `docker image inspect -f '{{.Config.Cmd}}' nginx` |

##### `docker history` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--format` | — | Go template | `docker history --format '{{.CreatedBy}}' nginx` |
| `--human` | `-H` | Human-readable sizes (default true) | `docker history -H=false nginx` |
| `--no-trunc` | — | Show full commands (not truncated) | `docker history --no-trunc nginx` |
| `--quiet` | `-q` | Only show layer IDs | `docker history -q nginx` |

##### `docker tag` flags

`docker tag` has no flags — it takes exactly two arguments:

```
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

##### `docker search` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--filter` | `-f` | Filter results (is-official, stars) | `docker search -f is-official=true nginx` |
| `--format` | — | Go template | `docker search --format '{{.Name}}' nginx` |
| `--limit` | — | Max results (default 25) | `docker search --limit 5 nginx` |
| `--no-trunc` | — | Don't truncate description | `docker search --no-trunc nginx` |

##### `docker push` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--all-tags` | `-a` | Push all tagged variants | `docker push -a myuser/api` |
| `--disable-content-trust` | — | Skip content trust check | `docker push --disable-content-trust ...` |
| `--quiet` | `-q` | Suppress verbose output | `docker push -q myuser/api:1.0` |

##### `docker login` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--username` | `-u` | Username | `docker login -u myuser` |
| `--password` | `-p` | Password (insecure — prefer stdin) | `docker login -p PASS` |
| `--password-stdin` | — | Read password from stdin | `echo $PAT \| docker login -u me --password-stdin` |

##### `docker image prune` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--all` | `-a` | Remove all unused (not just dangling) | `docker image prune -a` |
| `--force` | `-f` | No confirmation prompt | `docker image prune -f` |
| `--filter` | — | Filter which to prune | `docker image prune -a --filter "until=72h"` |

##### `docker image save` / `docker image load` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--output` | `-o` | Save to a file (else stdout) | `docker save -o nginx.tar nginx:alpine` |
| `--input` | `-i` | Load from a file (else stdin) | `docker load -i nginx.tar` |
| `--quiet` | `-q` | Suppress output | `docker load -q -i nginx.tar` |

##### `docker images` filter values (revisited)

| Filter | Meaning | Example |
|---|---|---|
| `dangling=true` | Untagged (`<none>:<none>`) images | `docker images -f dangling=true` |
| `label=key` or `label=key=value` | Images with specific label | `docker images -f label=env=prod` |
| `before=IMAGE` | Older than given image | `docker images -f before=nginx:alpine` |
| `since=IMAGE` | Newer than given image | `docker images -f since=alpine:3.20` |
| `reference=PATTERN` | Repository/tag pattern (globs) | `docker images -f "reference=node:*-alpine"` |

<a id="m4-examples"></a>
#### Examples

##### Example 1 — Inspect an image

<!-- COMMAND -->
```powershell
docker pull nginx:alpine
docker image inspect nginx:alpine
```

<!-- OUTPUT -->
```json
[
    {
        "Id": "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdee",
        "RepoTags": ["nginx:alpine"],
        "RepoDigests": ["nginx@sha256:2140dad235c1..."],
        "Created": "2024-10-14T21:20:42Z",
        "Config": {
            "Env": ["PATH=/usr/local/sbin:...", "NGINX_VERSION=1.27.2", ...],
            "Cmd": ["nginx", "-g", "daemon off;"],
            "ExposedPorts": {"80/tcp": {}},
            "WorkingDir": "",
            "Entrypoint": ["/docker-entrypoint.sh"],
            "StopSignal": "SIGQUIT"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 20504389,
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:c6b39d5da...", "sha256:27f5a1b7e...", ...
            ]
        }
    }
]
```

Key fields:

- `Id` — the image's own SHA256.
- `RepoTags` — all human-readable tags pointing at this image on the local machine.
- `RepoDigests` — the pull-time digest as fetched from the registry.
- `Created` — image build timestamp.
- `Config.Env` — default environment variables inside the container.
- `Config.Cmd` — the default command that runs when a container is started.
- `Config.ExposedPorts` — declared listening ports (informational; does not auto-publish).
- `Config.Entrypoint` — the wrapper script (Nginx's entrypoint copies configs into place before exec'ing the CMD).
- `Config.StopSignal` — signal sent by `docker stop` (Nginx uses `SIGQUIT` for graceful shutdown).
- `Architecture` / `Os` — target platform.
- `Size` — total on-disk size in bytes.
- `RootFS.Layers` — list of layer digests in order (bottom → top).

##### Example 2 — Extract just one field with `--format`

<!-- COMMAND -->
```powershell
docker image inspect --format '{{.Config.Cmd}}' nginx:alpine
docker image inspect --format '{{.Architecture}}/{{.Os}}' nginx:alpine
docker image inspect --format '{{.Size}}' nginx:alpine
```

<!-- OUTPUT -->
```
[nginx -g daemon off;]
amd64/linux
20504389
```

##### Example 3 — Layer history

<!-- COMMAND -->
```powershell
docker history nginx:alpine
```

<!-- OUTPUT -->
```
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
d1a364dc548d   2 weeks ago   CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      2 weeks ago   STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
<missing>      2 weeks ago   EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      2 weeks ago   ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      2 weeks ago   COPY 30-tune-worker-processes.sh                4.62kB    buildkit.dockerfile.v0
<missing>      2 weeks ago   COPY docker-entrypoint.sh                       1.62kB    buildkit.dockerfile.v0
<missing>      2 weeks ago   RUN /bin/sh -c set -x  && addgroup -g 101 ...   12.4MB    buildkit.dockerfile.v0
<missing>      2 weeks ago   ENV DYNPKG_RELEASE=1                            0B        buildkit.dockerfile.v0
...
<missing>      3 weeks ago   ADD alpine-minirootfs.tar.gz                    7.7MB     buildkit.dockerfile.v0
```

Reading top-to-bottom: the newest layer (which produced this image ID) is at the top. Each row is one instruction from the Dockerfile. `<missing>` in the IMAGE column just means those intermediate layers were not preserved with their own image IDs (normal for pulled images). Total size ≈ sum of the SIZE column.

##### Example 4 — Tag an image with a new name

<!-- COMMAND -->
```powershell
docker tag nginx:alpine myusername/nginx-custom:v1.0
docker images | Select-String nginx
```

<!-- OUTPUT -->
```
myusername/nginx-custom   v1.0     d1a364dc548d   2 weeks ago   20.5MB
nginx                     alpine   d1a364dc548d   2 weeks ago   20.5MB
```

Notice both tags have the *same IMAGE ID* — `docker tag` did not duplicate anything on disk. A tag is just a pointer.

##### Example 5 — Search Docker Hub

<!-- COMMAND -->
```powershell
docker search --limit 5 --filter is-official=true python
```

<!-- OUTPUT -->
```
NAME     DESCRIPTION                            STARS     OFFICIAL
python   Python is an interpreted, ...          9345      [OK]
```

##### Example 6 — Push (requires login)

<!-- COMMAND -->
```powershell
docker login
docker push myusername/nginx-custom:v1.0
```

<!-- OUTPUT -->
```
Login Succeeded

The push refers to repository [docker.io/myusername/nginx-custom]
c6b39d5da001: Mounted from library/nginx
27f5a1b7e34c: Mounted from library/nginx
...
v1.0: digest: sha256:2140dad235c1... size: 1741
```

`Mounted from library/nginx` means the registry already had those layers from the official `nginx` repo and simply linked them — no re-upload.

##### Example 7 — Enable Docker Content Trust

<!-- COMMAND -->
```powershell
$env:DOCKER_CONTENT_TRUST = "1"
docker pull alpine:3.20
```

<!-- OUTPUT -->
```
Pull (1 of 1): alpine:3.20@sha256:....
3.20@sha256:...: Pulling from library/alpine
...
Tagging alpine@sha256:... as alpine:3.20
```

When DCT is on, Docker refuses to pull unsigned images. Disable with:

<!-- COMMAND -->
```powershell
$env:DOCKER_CONTENT_TRUST = ""
```

<a id="m4-exercises"></a>
#### Command Exercises

##### Exercise 4.1 — `docker image inspect` basic

Inspect the `alpine:3.20` image and identify its size in bytes and its default `CMD`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker pull alpine:3.20
docker image inspect alpine:3.20
```

<!-- OUTPUT -->
```
[
    {
        "Id": "sha256:71fc0a10c1a1...",
        "Size": 7700000,
        "Config": {
            "Cmd": ["/bin/sh"],
            ...
        },
        ...
    }
]
```

Answers: Size ≈ 7.7 MB (the smallest general-purpose Linux distro),
CMD is `/bin/sh` — which is why `docker run -it alpine` drops you at
a shell without specifying a command.

</details>

##### Exercise 4.2 — `docker image inspect --format`

Print exactly one line containing the exposed ports of `nginx:alpine`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker image inspect --format '{{.Config.ExposedPorts}}' nginx:alpine
```

<!-- OUTPUT -->
```
map[80/tcp:{}]
```

Go template renders the map as Go syntax. To get a cleaner display:

<!-- COMMAND -->
```powershell
docker image inspect --format '{{range $p, $_ := .Config.ExposedPorts}}{{$p}} {{end}}' nginx:alpine
```

<!-- OUTPUT -->
```
80/tcp
```

</details>

##### Exercise 4.3 — `docker image inspect` combined

For every image locally cached, print `name:tag  ->  architecture  ->  size(MB)`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
foreach ($id in (docker images -q)) {
  docker image inspect --format '{{index .RepoTags 0}} -> {{.Architecture}} -> {{.Size}}' $id
}
```

<!-- OUTPUT -->
```
nginx:alpine -> amd64 -> 20504389
alpine:3.20 -> amd64 -> 7700000
python:3.12-slim -> amd64 -> 129000000
...
```

Convert bytes to MB in the shell if you want:

<!-- COMMAND -->
```powershell
foreach ($id in (docker images -q)) {
  $t = docker image inspect --format '{{index .RepoTags 0}}' $id
  $s = [int](docker image inspect --format '{{.Size}}' $id)
  "$t -> $([math]::Round($s/1MB,1)) MB"
}
```

<!-- OUTPUT -->
```
nginx:alpine -> 19.6 MB
alpine:3.20 -> 7.3 MB
python:3.12-slim -> 123 MB
```

</details>

##### Exercise 4.4 — `docker history` basic

Show the layer history of `alpine:3.20`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker history alpine:3.20
```

<!-- OUTPUT -->
```
IMAGE          CREATED       CREATED BY                    SIZE     COMMENT
71fc0a10c1a1   2 weeks ago   CMD ["/bin/sh"]               0B       buildkit.dockerfile.v0
<missing>      2 weeks ago   ADD alpine-minirootfs.tar.gz  7.7MB    buildkit.dockerfile.v0
```

Alpine is only two layers — that's why it's tiny. Compare to Ubuntu
or Debian which have many more.

</details>

##### Exercise 4.5 — `docker history --no-trunc`

Reveal the FULL Dockerfile commands (not truncated) for `nginx:alpine`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker history --no-trunc nginx:alpine
```

<!-- OUTPUT -->
```
IMAGE     CREATED       CREATED BY                                                     SIZE
sha256... 2 weeks ago   CMD ["nginx" "-g" "daemon off;"]                               0B
<missing> 2 weeks ago   RUN /bin/sh -c set -x     && addgroup -g 101 -S nginx     &&
                        adduser -S -D -H -u 101 -h /var/cache/nginx -s /sbin/nologin
                        -G nginx -g nginx nginx     && apkArch="$(cat /etc/apk/arch)"
                        ...                                                            12.4MB
...
```

You now see the exact shell command that produced each layer.
`--no-trunc` is essential when reverse-engineering an image.

</details>

##### Exercise 4.6 — `docker history` size analysis

For `nginx:alpine`, identify the SINGLE largest layer and its size.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker history nginx:alpine --format '{{.Size}}\t{{.CreatedBy}}' | Sort-Object { [regex]::Match($_, "^\S+").Value } -Descending | Select-Object -First 3
```

<!-- OUTPUT -->
```
12.4MB   RUN /bin/sh -c set -x && addgroup -g 101 ...    (the nginx install)
7.7MB    ADD alpine-minirootfs.tar.gz ...                (the alpine base)
4.62kB   COPY 30-tune-worker-processes.sh ...
```

Answer: the largest single layer is the RUN step that installs nginx
itself (~12.4 MB). This is typical — the RUN that installs packages
is almost always the biggest layer in a Dockerfile.

</details>

##### Exercise 4.7 — `docker search` basic

Search Docker Hub for images related to `postgres`, limited to 5 results.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker search --limit 5 postgres
```

<!-- OUTPUT -->
```
NAME                 DESCRIPTION                            STARS   OFFICIAL
postgres             The PostgreSQL object-relational...    13209   [OK]
bitnami/postgresql   Bitnami PostgreSQL Docker Image        278
postgrest/postgrest  REST API for any Postgres database     158
circleci/postgres    CircleCI images ...                     32
kartoza/postgis      PostGIS spatial extension              189
```

</details>

##### Exercise 4.8 — `docker search` filtered

Search for `python` images, official only.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker search --filter is-official=true python
```

<!-- OUTPUT -->
```
NAME     DESCRIPTION                            STARS   OFFICIAL
python   Python is an interpreted, ...          9345    [OK]
```

Only the official `library/python` image comes back. The `is-official`
filter is the fastest way to find trustworthy base images from the CLI.

</details>

##### Exercise 4.9 — `docker search` — troubleshooting a garbage result

Search for `nodejs` and note how many results are from user accounts (not official). What is the correct official image name?

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker search --limit 10 nodejs
```

<!-- OUTPUT -->
```
NAME                    DESCRIPTION                            STARS   OFFICIAL
adhawk/nodejs           ...                                    5
seovaia/nodejs          ...                                    3
...
```

None marked OFFICIAL — because the official image is named `node`,
not `nodejs`. Correct search:

<!-- COMMAND -->
```powershell
docker search --filter is-official=true node
```

<!-- OUTPUT -->
```
NAME   DESCRIPTION                            STARS   OFFICIAL
node   Node.js is a JavaScript-based platform 13897   [OK]
```

Lesson: image names don't always match the "friendly" project name.
Verify official image names at hub.docker.com before adopting them.

</details>

##### Exercise 4.10 — `docker tag` basic

Tag the local `alpine:3.20` image as `mylabs/alpine:2025-11-05`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker tag alpine:3.20 mylabs/alpine:2025-11-05
docker images | Select-String -Pattern "alpine|mylabs"
```

<!-- OUTPUT -->
```
mylabs/alpine     2025-11-05   71fc0a10c1a1   2 weeks ago   7.7MB
alpine            3.20         71fc0a10c1a1   2 weeks ago   7.7MB
```

Same IMAGE ID — just a new pointer.

</details>

##### Exercise 4.11 — `docker tag` for registry push

Tag `alpine:3.20` for a private registry at `myregistry.corp:5000` under `infra/alpine` with tag `latest`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker tag alpine:3.20 myregistry.corp:5000/infra/alpine:latest
docker images myregistry.corp:5000/infra/alpine
```

<!-- OUTPUT -->
```
REPOSITORY                            TAG      IMAGE ID       CREATED        SIZE
myregistry.corp:5000/infra/alpine     latest   71fc0a10c1a1   2 weeks ago    7.7MB
```

Including the registry host (with port) in the tag is how Docker knows
where to push. Without the host, it defaults to Docker Hub.

</details>

##### Exercise 4.12 — `docker tag` — undo by untagging

Remove ONLY the tag `mylabs/alpine:2025-11-05` without deleting the underlying image.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker rmi mylabs/alpine:2025-11-05
docker images | Select-String -Pattern "alpine|mylabs"
```

<!-- OUTPUT -->
```
Untagged: mylabs/alpine:2025-11-05

alpine   3.20   71fc0a10c1a1   2 weeks ago   7.7MB
```

Because `alpine:3.20` still points to the same image ID, Docker only
removes the extra tag, not the layers. This is called "untagging" —
distinct from actually deleting layers, which only happens when the
last tag/reference is removed.

</details>

##### Exercise 4.13 — `docker images` filter by reference

List all locally cached Node.js images with a specific pattern.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker pull node:20-alpine
docker pull node:20-slim
docker images -f "reference=node:*-alpine"
```

<!-- OUTPUT -->
```
REPOSITORY   TAG          IMAGE ID       CREATED       SIZE
node         20-alpine    a1b2c3d4e5f6   2 weeks ago   135MB
```

The `reference` filter accepts glob patterns matching `repo:tag`.

</details>

##### Exercise 4.14 — `docker images` filter dangling

Create a dangling image and then list dangling images.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
# Build once
mkdir tmpbuild; cd tmpbuild
"FROM alpine:3.20`nRUN echo v1 > /tag.txt" | Out-File Dockerfile -Encoding ascii
docker build -t demo:v1 .

# Change and rebuild — the previous demo:v1 becomes dangling
"FROM alpine:3.20`nRUN echo v2 > /tag.txt" | Out-File Dockerfile -Encoding ascii
docker build -t demo:v1 .

docker images -f dangling=true
```

<!-- OUTPUT -->
```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
<none>       <none>    abc123def456   30 seconds ago   7.7MB
```

The dangling image is the "orphaned" `<none>:<none>` version that no
longer has a tag pointing at it (because the second build stole the
`demo:v1` tag). Clean up:

<!-- COMMAND -->
```powershell
docker image prune -f
```

</details>

##### Exercise 4.15 — `docker images` filter since/before

List all images pulled AFTER `alpine:3.20`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker images -f since=alpine:3.20
```

<!-- OUTPUT -->
```
REPOSITORY   TAG          IMAGE ID       CREATED           SIZE
node         20-alpine    a1b2c3d4e5f6   30 seconds ago    135MB
demo         v1           efabcd012345   3 minutes ago     7.7MB
```

Useful for identifying "what have I pulled today" or clearing recent
builds.

</details>

<a id="m4-handson"></a>
#### Hands-On Assignment

**Task:** Pull three variants of Node.js 20, inspect them, and produce a written comparison.

Steps:

1. Pull all three variants:
   <!-- COMMAND -->
   ```powershell
   docker pull node:20
   docker pull node:20-slim
   docker pull node:20-alpine
   ```

2. List them with size:
   <!-- COMMAND -->
   ```powershell
   docker images node --format 'table {{.Tag}}\t{{.Size}}'
   ```
   <!-- OUTPUT -->
   ```
   TAG          SIZE
   20           1.11GB
   20-slim      241MB
   20-alpine    135MB
   ```

3. Inspect each and record the base OS. Hint: check the layer history for the base image ADD/tar.gz layer, or read the label `org.opencontainers.image.base.name` if present:
   <!-- COMMAND -->
   ```powershell
   docker image inspect --format '{{index .Config.Labels "org.opencontainers.image.base.name"}}' node:20
   docker image inspect --format '{{index .Config.Labels "org.opencontainers.image.base.name"}}' node:20-slim
   docker image inspect --format '{{index .Config.Labels "org.opencontainers.image.base.name"}}' node:20-alpine
   ```

4. Count layers each:
   <!-- COMMAND -->
   ```powershell
   docker history node:20 -q | Measure-Object | ForEach-Object Count
   docker history node:20-slim -q | Measure-Object | ForEach-Object Count
   docker history node:20-alpine -q | Measure-Object | ForEach-Object Count
   ```

5. Create a comparison table in a `nodejs-comparison.md` file:

   | Tag | Base | Size | Layers |
   |---|---|---|---|
   | node:20 | debian:bookworm | 1.11 GB | 8 |
   | node:20-slim | debian:bookworm-slim | 241 MB | 5 |
   | node:20-alpine | alpine:3.20 | 135 MB | 4 |

**Acceptance criteria:**

- [ ] All three images pulled and visible in `docker images`.
- [ ] Comparison table saved to `nodejs-comparison.md`.
- [ ] Base OS identified for each variant.
- [ ] Layer counts recorded.
- [ ] You can state, in one sentence, which variant you'd pick for production and why.

<a id="m4-miniproject"></a>
#### Mini-Project

##### 🎯 Project Title
**Docker Image Investigator Report**

##### 🎯 Objective

Choose a category of application (web servers), pull five popular images in that category, forensically analyze each using `inspect`, `history`, and `search` metadata, then produce a written recommendation for which to use in a hypothetical production deployment. This is the exact workflow a platform engineer performs when choosing base images for their team.

##### 📋 Requirements

1. **Pick the category:** web servers.
2. **Pull five official/community images:** `nginx:alpine`, `httpd:alpine`, `caddy:2-alpine`, `traefik:v3.1`, `lighttpd`.
3. **For each image, record:** size, layer count, base OS (from labels or last ADD layer), exposed ports, entrypoint, cmd, and creation date.
4. **Build a comparison table** with columns: Image, Size, Layers, Base, Exposed Ports, Entrypoint, CMD, Created.
5. **Identify the largest single layer** in each image using `docker history`.
6. **Tag one image** with a custom tag `mycompany/webserver:2025-11-eval` and verify with `docker images`.
7. **(Optional)** Push the tagged image to your personal Docker Hub account (skip if you don't have one — no penalty).
8. **Write a recommendation:** given a hypothetical requirement — "we need a reverse proxy in front of 20 internal microservices, minimizing image size and CVE surface, with built-in HTTPS/Let's Encrypt" — which image would you pick and why?

##### 🪜 Step-by-Step Guidance

1. Create folder `docker-mastery\milestone-4\`, then file `image-investigator.md`.
2. In the file, write a `## Setup` section listing the five `docker pull` commands and their outputs.
3. Write a `## Per-Image Analysis` section. For each image, subsection with:
   - `### <image>`
   - Output of `docker image inspect --format '<template>'` for size, cmd, entrypoint, exposed ports, created, architecture.
   - Output of `docker history <image>` — full table.
   - A one-sentence summary of the largest layer.
4. Write a `## Comparison Table` section with all five images side-by-side.
5. Write a `## Tagging Demo` section — the `docker tag` command and its verification via `docker images`.
6. Write a `## Recommendation` section (200–400 words) with your pick, justification, and trade-offs.
7. (Optional) Write a `## Push Log` section if you pushed to Docker Hub.

##### 📦 Complete Mini-Project Solution

<details><summary>📦 Complete Mini-Project Solution</summary>

<!-- CODE -->
````markdown
# Docker Image Investigator Report — Web Servers

## Setup — Pull five images

```powershell
docker pull nginx:alpine
docker pull httpd:alpine
docker pull caddy:2-alpine
docker pull traefik:v3.1
docker pull lighttpd
```

## Per-Image Analysis

### nginx:alpine

```powershell
docker image inspect --format 'Size:{{.Size}} | Cmd:{{.Config.Cmd}} | Entrypoint:{{.Config.Entrypoint}} | Ports:{{.Config.ExposedPorts}} | Created:{{.Created}}' nginx:alpine
docker history nginx:alpine
```

- Size: ~20.5 MB
- Base: alpine:3.20
- Exposed: 80/tcp
- Entrypoint: [/docker-entrypoint.sh]
- Cmd: [nginx -g daemon off;]
- Layers: 9
- Largest layer: 12.4 MB (RUN installing nginx package)

### httpd:alpine

- Size: ~62 MB
- Base: alpine:3.20
- Exposed: 80/tcp
- Entrypoint: [httpd-foreground]
- Cmd: []
- Layers: 8
- Largest layer: ~44 MB (RUN compile-and-install httpd)

### caddy:2-alpine

- Size: ~52 MB
- Base: alpine:3.20
- Exposed: 80/tcp, 443/tcp, 443/udp, 2019/tcp
- Entrypoint: [caddy]
- Cmd: [run --config /etc/caddy/Caddyfile --adapter caddyfile]
- Layers: 6
- Largest layer: ~44 MB (RUN adding caddy binary)

### traefik:v3.1

- Size: ~172 MB
- Base: alpine (no explicit label)
- Exposed: 80/tcp
- Entrypoint: [/entrypoint.sh]
- Cmd: [traefik]
- Layers: 7
- Largest layer: ~165 MB (COPY traefik binary + plugins)

### lighttpd

- Size: ~14 MB
- Base: alpine:3.20
- Exposed: 80/tcp
- Entrypoint: []
- Cmd: [lighttpd -D -f /etc/lighttpd/lighttpd.conf]
- Layers: 5
- Largest layer: ~6 MB (RUN install lighttpd)

## Comparison Table

| Image             | Size    | Layers | Base    | Ports              | Entrypoint            | CMD                              |
|-------------------|---------|--------|---------|--------------------|-----------------------|----------------------------------|
| nginx:alpine      | 20.5 MB | 9      | alpine  | 80                 | /docker-entrypoint.sh | nginx -g daemon off              |
| httpd:alpine      | 62 MB   | 8      | alpine  | 80                 | httpd-foreground      | (none)                           |
| caddy:2-alpine    | 52 MB   | 6      | alpine  | 80, 443, 443/udp   | caddy                 | run --config /etc/caddy/Caddyfile|
| traefik:v3.1      | 172 MB  | 7      | alpine  | 80                 | /entrypoint.sh        | traefik                          |
| lighttpd          | 14 MB   | 5      | alpine  | 80                 | (none)                | lighttpd -D -f /etc/lighttpd/... |

## Tagging Demo

```powershell
docker tag caddy:2-alpine mycompany/webserver:2025-11-eval
docker images | Select-String mycompany
```

Output:
```
mycompany/webserver   2025-11-eval   0d1e2f3a4b5c   2 weeks ago   52MB
```

## Recommendation

For the stated requirement — a reverse proxy in front of 20 internal
microservices, minimizing image size and CVE surface, with built-in
HTTPS/Let's Encrypt — my pick is **Caddy (`caddy:2-alpine`)**.

Rationale:

1. **Built-in HTTPS via ACME.** Caddy handles Let's Encrypt certificate
   provisioning and renewal automatically with zero configuration. Nginx
   and Traefik can do it but require plugin/config work; Apache and
   Lighttpd require external tooling like certbot.
2. **Small footprint.** At ~52 MB Caddy is 3x smaller than Traefik and
   about 2.5x larger than nginx:alpine — a reasonable trade-off given
   the HTTPS features it includes out of the box.
3. **Modern config (Caddyfile).** Much easier to read and maintain for
   20 microservices than Nginx conf blocks.
4. **Single static Go binary.** Fewer runtime dependencies → smaller
   attack surface than Apache's module ecosystem.

Trade-offs I would flag to my team:

- Caddy has a smaller operator community than Nginx or Traefik. If the
  team already has deep Nginx expertise, Nginx + certbot may be a lower
  learning-curve choice.
- If we later need active service discovery (Docker Swarm / K8s labels),
  Traefik is superior — I would revisit then.
- If image size is *strictly* the top priority, `nginx:alpine` (20 MB)
  or `lighttpd` (14 MB) win, but you lose auto-HTTPS.

Second choice: `nginx:alpine` for its long-term stability and huge
community. Third choice: `traefik:v3.1` if service discovery becomes
a requirement.
````

</details>

##### ✅ Verification Checklist

- [ ] `image-investigator.md` exists at `docker-mastery\milestone-4\`.
- [ ] All five images (`nginx:alpine`, `httpd:alpine`, `caddy:2-alpine`, `traefik:v3.1`, `lighttpd`) are pulled — verified via `docker images`.
- [ ] For each image, the report records: size, base OS, exposed ports, entrypoint, cmd, created date, layer count.
- [ ] `docker image inspect --format` was used at least once per image.
- [ ] `docker history` was inspected for each image and the largest layer is called out.
- [ ] A comparison table with all five images appears in the report.
- [ ] `docker tag` was used to create `mycompany/webserver:2025-11-eval` and verified.
- [ ] A written recommendation paragraph (200+ words) explains a preferred choice with rationale and trade-offs.
- [ ] Report includes at least three distinct considerations (size, security, features, ease of use) in the recommendation.

##### 🌟 Bonus Challenges

1. **Multi-arch check.** Use `docker manifest inspect` (experimental) on each of the five to list which platforms each image publishes. Add a "Multi-arch support" column to your comparison table.
2. **CVE preview.** If you have Docker Scout enabled, run `docker scout cves nginx:alpine` and record the number of high/critical CVEs for each image. Add a "CVE count" column and let that influence your recommendation.
3. **Push to Docker Hub.** Sign up for a free Docker Hub account, `docker login`, and push the `mycompany/webserver:2025-11-eval` tag to your namespace. Add the "Push Log" section to your report showing the layer-mount optimization at work.

<a id="m4-scenario"></a>
#### Scenario (Real-World Use Case)

You've joined a platform team responsible for approving base images used by 200 developers across 40 microservices. Marketing announces a new mobile push notification service, and its lead engineer files a ticket: "We want to use `randomuser/node-alpine-fast:latest` as our base — it's smaller and starts faster."

Before approving, you run through Milestone 4 muscle memory: `docker pull randomuser/node-alpine-fast:latest`, then `docker image inspect` reveals the image was built two years ago. `docker history --no-trunc` shows a suspicious `curl | bash` step downloading a script from a shortener URL. `docker search randomuser/node-alpine-fast` shows 3 stars and no verified publisher badge. You reject the image, point the engineer at the official `node:20-alpine`, and explain the risk in three bullet points that reference the exact `docker history` output.

Your team writes a policy: "All base images must be Official or Verified Publisher, updated within the last 90 days, with no external downloads in RUN steps." You put a Slack bot on it that automates the check. Every skill from Milestone 4 — `inspect`, `history`, `search`, understanding tags vs digests, official vs community — was in play in that 20-minute review.

<a id="m4-quiz"></a>
#### Checkpoint Quiz

**Question 1.** What is the difference between a tag and a digest?

<details><summary>Click to reveal answer</summary>

A **tag** is a mutable, human-readable label (like `nginx:1.27`) that a publisher can repoint to a different image at any time. A **digest** is an immutable SHA256 hash (like `nginx@sha256:2140dad235c1...`) computed from the image's exact contents. Two pulls of the same digest, anywhere on Earth, are guaranteed byte-identical. Production deployments should pin to digests for reproducibility; tags are fine for development where you want automatic updates.

</details>

**Question 2.** If two different images both start with `FROM python:3.12-slim`, how many copies of the `python:3.12-slim` layers does Docker store on your disk?

<details><summary>Click to reveal answer</summary>

**One copy.** Docker stores each layer once (keyed by content hash) and multiple images reference the same layer blob. That is the entire point of the layered filesystem — massive disk savings across related images. This is also why pulling many images from the same base is fast: only the differing top layers need to download.

</details>

**Question 3.** Predict: what does `docker tag foo:1.0 foo:2.0` do to disk usage?

<details><summary>Click to reveal answer</summary>

**Nothing.** `docker tag` only creates a new name pointing to an existing image ID — no layers are copied, no bytes hit the disk. This is why tagging is instant and free. Contrast this with `docker build` or `docker commit`, which produce new layer content and do consume disk.

</details>

**Question 4.** Your CI pipeline pulls `nginx:1.27` on every build. Six weeks later, `nginx:1.27` is silently republished by the Nginx team with a security patch (`1.27.2`). What happens on your next CI build?

<details><summary>Click to reveal answer</summary>

Docker sees the tag `nginx:1.27` now points to a new digest. On pull, Docker downloads the new layers and updates your local cache. Your build starts using the patched image — invisibly and automatically. This is often what you want (auto-patching), but it means the same tag can yield different builds over time. To lock in exact reproducibility, pin to `nginx@sha256:...` instead of `nginx:1.27`.

</details>

**Question 5.** You run `docker history someimage` and see one row `<missing>` with a 2 GB size. What does `<missing>` mean, and why is the size that big?

<details><summary>Click to reveal answer</summary>

`<missing>` in the `IMAGE` column means that intermediate layer was **not stored with its own tagged image ID** — normal for images you pulled (rather than built locally). It does *not* mean the layer is broken. The 2 GB size means whatever Dockerfile instruction produced that layer wrote 2 GB of content — often a `RUN apt-get install ... -y` for something heavy like TeX Live or CUDA, or a `COPY` of a giant asset. Big layers are red flags for image bloat and a signal to investigate multi-stage builds (Milestone 6).

</details>
