
### Milestone 3: Docker CLI Basics

---
📍 **Milestone 3 of 20** | 🟢 **Beginner** | ⏱ **Estimated Time: 3–4 hours** | **Prerequisites:** Milestones 1–2
---

<a id="m3-theory"></a>
#### Theory

Now that Docker is installed and healthy, you will learn the seven core CLI commands that make up 90% of daily Docker use: `run`, `ps`, `stop`, `start`, `restart`, `rm`, `pull`, `images`, and `rmi`. These commands cover the entire lifecycle of a container — from downloading its image to creating, inspecting, stopping, restarting, and destroying it — and every advanced topic in later milestones builds on this foundation.

##### The Container Lifecycle in One Diagram

```
<!-- CODE -->
                    ┌─────────────┐
                    │   REGISTRY  │  (Docker Hub)
                    │   (image)   │
                    └──────┬──────┘
                           │
                    docker pull
                           │
                           ▼
                    ┌─────────────┐
                    │   IMAGE     │  cached locally
                    │  (on disk)  │  `docker images`
                    └──────┬──────┘
                           │
                     docker run  (create + start)
                           │
                           ▼
                ┌──────────────────────┐
                │   CONTAINER          │
                │   (running)          │  `docker ps`
                └──────┬───────┬───────┘
                       │       │
                       │       └── docker stop ──► STOPPED
                       │                              │
                       │           docker start ◄─────┘
                       │           docker restart
                       ▼
                  docker rm
                       │
                       ▼
                    (gone)
                       │
                  docker rmi
                       │
                       ▼
              (image also gone)
```

##### The Two Modes: Interactive vs Detached

Every container runs a single "main process" (its `CMD` or `ENTRYPOINT`, defined in the image). What that process does — and how you interact with it — depends on how you launch it:

- **Interactive mode (`-i -t`, usually combined as `-it`)** attaches your terminal's keyboard and screen to the container's stdin/stdout/stderr, and gives it a pseudo-TTY (so things like colored prompts and Ctrl+C work). Use this to explore a shell, run an interactive REPL (Python, Node, mysql), or step through a script.
  - `-i` (`--interactive`) — keep stdin open, so keyboard input flows into the container.
  - `-t` (`--tty`) — allocate a pseudo-terminal, so shell prompts render properly.
  - You almost always want both together for shells. Using only `-i` works for piping input; using only `-t` is rare.
- **Detached mode (`-d`)** runs the container in the background and returns you to your prompt immediately. The container keeps running until its main process exits. Use this for long-running services like web servers and databases.

You can attach to a detached container later with `docker attach` or `docker exec -it <name> bash`, and you can watch its logs with `docker logs -f`.

##### Names and Identifiers

Every container has:

- **A container ID** — a 64-character SHA256 hex string. Docker usually shows a shortened 12-character version.
- **A name** — either auto-generated (like `boring_einstein` — a random adjective + scientist) or one you supply with `--name`.

You can refer to a container by full ID, short ID, or name in any command that takes `<container>`. Names are strongly recommended in real work because they make `docker ps`, `docker logs`, and scripts self-documenting.

##### Port Mapping

By default, container processes are only reachable from *inside* the container. To make a container's port available to the Windows host (and thus a browser), you use `-p HOST_PORT:CONTAINER_PORT`.

- `-p 8080:80` — forward host port 8080 to container port 80. Anyone hitting `http://localhost:8080` reaches the container's port 80.
- `-p 127.0.0.1:8080:80` — bind only to the loopback address on the host, so only your machine can reach it (not other machines on your LAN).
- `-p 8080:80 -p 8443:443` — multiple mappings.
- `-p 80` — publish to a random high port on the host (see the assigned port with `docker port <name>` or `docker ps`).
- `-P` (capital) — publish *all* ports the image `EXPOSE`s to random host ports.

##### Environment Variables

Applications often read configuration from environment variables (database URLs, API keys, log levels). You pass these in with:

- `-e KEY=value` — a single variable.
- `-e KEY` — pass through a variable from your shell to the container.
- `--env-file .env` — read `KEY=value` lines from a file. Cleaner for many variables and easier to keep secrets out of shell history.

##### The `--rm` Flag

Normally, a stopped container hangs around on disk so you can restart it, inspect its logs, or copy files out. During experimentation this quickly clutters your system with dozens of dead containers. Add `--rm` to `docker run` to automatically delete the container the moment its main process exits. Use `--rm` for one-off commands (`docker run --rm alpine echo hi`), and *do not* use it for services (you would lose logs when the container exits).

<a id="m3-commands"></a>
#### Commands

- `docker run` — create and start a new container from an image.
- `docker ps` — list running containers.
- `docker ps -a` — list all containers, including stopped ones.
- `docker stop` — send SIGTERM to a container's main process (then SIGKILL after a grace period).
- `docker start` — start a stopped container.
- `docker restart` — stop then start a container.
- `docker rm` — delete a stopped (or forced-stopped) container.
- `docker pull` — download an image from a registry into the local cache.
- `docker images` (alias: `docker image ls`) — list locally cached images.
- `docker rmi` (alias: `docker image rm`) — delete a locally cached image.
- `docker port` — show port mappings of a container.
- `docker logs` — print a container's captured stdout/stderr.
- `docker exec` — run a new command inside a running container.

<a id="m3-flags"></a>
#### Command Options/Flags

##### `docker run` flags (most-used subset — full list has 100+ flags)

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--name` | — | Assign a name to the container | `docker run --name web nginx` |
| `--detach` | `-d` | Run in the background | `docker run -d nginx` |
| `--interactive` | `-i` | Keep stdin open | `docker run -i alpine` |
| `--tty` | `-t` | Allocate a pseudo-TTY | `docker run -it alpine sh` |
| `--rm` | — | Delete container on exit | `docker run --rm alpine echo hi` |
| `--publish` | `-p` | Map host port to container port | `docker run -p 8080:80 nginx` |
| `--publish-all` | `-P` | Publish all EXPOSEd ports to random host ports | `docker run -P nginx` |
| `--env` | `-e` | Set environment variable | `docker run -e LOG_LEVEL=debug nginx` |
| `--env-file` | — | Load env vars from file | `docker run --env-file .env nginx` |
| `--volume` | `-v` | Mount a volume or host path | `docker run -v mydata:/data nginx` |
| `--mount` | — | Attach a volume/bind (verbose syntax) | `docker run --mount type=volume,src=mydata,dst=/data nginx` |
| `--workdir` | `-w` | Set working directory inside container | `docker run -w /app node` |
| `--user` | `-u` | Run as UID/username | `docker run -u 1000 alpine` |
| `--network` | — | Attach to a specific network | `docker run --network host nginx` |
| `--hostname` | `-h` | Set container hostname | `docker run -h web01 nginx` |
| `--restart` | — | Restart policy (no, on-failure, always, unless-stopped) | `docker run --restart=always nginx` |
| `--memory` | `-m` | Memory limit | `docker run -m 512m nginx` |
| `--cpus` | — | CPU limit (fractional cores) | `docker run --cpus=1.5 nginx` |
| `--label` | `-l` | Set metadata label | `docker run -l env=dev nginx` |
| `--entrypoint` | — | Override image ENTRYPOINT | `docker run --entrypoint sh alpine` |
| `--pull` | — | Always/missing/never pull before run | `docker run --pull=always nginx` |
| `--init` | — | Run tiny init as PID 1 (zombie reaping) | `docker run --init node` |
| `--read-only` | — | Mount root filesystem as read-only | `docker run --read-only nginx` |
| `--tmpfs` | — | Mount a tmpfs at a path | `docker run --tmpfs /tmp nginx` |
| `--dns` | — | Custom DNS server | `docker run --dns 8.8.8.8 alpine` |
| `--add-host` | — | Add host:ip to `/etc/hosts` | `docker run --add-host db:10.0.0.1 nginx` |
| `--platform` | — | Target platform for image (linux/amd64, linux/arm64) | `docker run --platform=linux/arm64 alpine` |

##### `docker ps` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--all` | `-a` | Show all containers (including stopped) | `docker ps -a` |
| `--quiet` | `-q` | Only display container IDs | `docker ps -q` |
| `--filter` | `-f` | Filter output (see below for values) | `docker ps -f status=running` |
| `--format` | — | Go template for formatting | `docker ps --format '{{.Names}}'` |
| `--last` | `-n` | Show n last created containers | `docker ps -n 3` |
| `--latest` | `-l` | Show the latest created container | `docker ps -l` |
| `--no-trunc` | — | Don't truncate output (show full IDs) | `docker ps --no-trunc` |
| `--size` | `-s` | Show container writable-layer size | `docker ps -s` |

`docker ps` filter values include: `status=` (running/exited/paused/created/restarting), `name=`, `id=`, `label=key=value`, `ancestor=image`, `network=`, `volume=`, `publish=port`, `expose=port`, `before=name`, `since=name`.

##### `docker stop` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--time` | `-t` | Seconds to wait before killing (default 10) | `docker stop -t 30 web` |
| `--signal` | `-s` | Signal to send (default SIGTERM) | `docker stop -s SIGINT web` |

##### `docker start` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--attach` | `-a` | Attach stdout/stderr | `docker start -a web` |
| `--interactive` | `-i` | Attach stdin | `docker start -i web` |
| `--detach-keys` | — | Override sequence to detach | `docker start --detach-keys="ctrl-p,ctrl-q" web` |

##### `docker restart` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--time` | `-t` | Seconds to wait before killing before restart | `docker restart -t 5 web` |
| `--signal` | `-s` | Signal to send | `docker restart -s SIGHUP web` |

##### `docker rm` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--force` | `-f` | Force-remove a running container (sends SIGKILL) | `docker rm -f web` |
| `--volumes` | `-v` | Also remove anonymous volumes attached to it | `docker rm -v web` |
| `--link` | `-l` | Remove specified link (legacy) | `docker rm -l alias` |

##### `docker pull` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--all-tags` | `-a` | Pull every tag of the repository | `docker pull -a nginx` |
| `--disable-content-trust` | — | Skip image signature verification | `docker pull --disable-content-trust nginx` |
| `--platform` | — | Set platform if image is multi-arch | `docker pull --platform=linux/arm64 nginx` |
| `--quiet` | `-q` | Suppress verbose progress output | `docker pull -q nginx` |

##### `docker images` / `docker image ls` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--all` | `-a` | Show all images (including intermediate) | `docker images -a` |
| `--digests` | — | Show image digests | `docker images --digests` |
| `--filter` | `-f` | Filter output | `docker images -f dangling=true` |
| `--format` | — | Go template | `docker images --format '{{.Repository}}:{{.Tag}}'` |
| `--no-trunc` | — | Don't truncate output | `docker images --no-trunc` |
| `--quiet` | `-q` | Only show image IDs | `docker images -q` |

`docker images` filter values include: `dangling=true|false`, `label=`, `before=image`, `since=image`, `reference=pattern`.

##### `docker rmi` / `docker image rm` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--force` | `-f` | Force-remove even if in use by stopped containers | `docker rmi -f nginx` |
| `--no-prune` | — | Don't delete untagged parent images | `docker rmi --no-prune nginx` |

<a id="m3-examples"></a>
#### Examples

##### Example 1 — Run a container in detached mode and inspect it

<!-- COMMAND -->
```powershell
docker run -d --name web -p 8080:80 nginx:alpine
```

<!-- OUTPUT -->
```
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
c6b39d5da001: Pull complete
27f5a1b7e34c: Pull complete
...
Digest: sha256:2140dad235c1...
Status: Downloaded newer image for nginx:alpine
d6f2a4e3b1c9a0f8e2d7c4b1a09e8f7d6c5b4a3d2e1f0987654321abcdef1234
```

The long hex string is the new container's full ID. Now list it:

<!-- COMMAND -->
```powershell
docker ps
```

<!-- OUTPUT -->
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
d6f2a4e3b1c9   nginx:alpine   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->80/tcp   web
```

Open `http://localhost:8080` in your browser — you see the Nginx welcome page.

##### Example 2 — Interactive shell in Alpine

<!-- COMMAND -->
```powershell
docker run -it --rm alpine sh
```

<!-- OUTPUT -->
```
/ # cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.20.3
PRETTY_NAME="Alpine Linux v3.20"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/issues"
/ # exit
```

Because of `--rm`, the container is gone the moment you type `exit`.

##### Example 3 — Environment variables

<!-- COMMAND -->
```powershell
docker run --rm -e GREETING=hello -e NAME=world alpine sh -c 'echo "$GREETING $NAME"'
```

<!-- OUTPUT -->
```
hello world
```

##### Example 4 — Stop, start, restart

<!-- COMMAND -->
```powershell
docker stop web
docker ps -a
docker start web
docker restart web
```

<!-- OUTPUT -->
```
web
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS     NAMES
d6f2a4e3b1c9   nginx:alpine   "/docker-entrypoint.…"   3 minutes ago   Exited (0) 5 seconds ago             web
web
web
```

##### Example 5 — Remove containers and images

<!-- COMMAND -->
```powershell
docker rm -f web
docker rmi nginx:alpine
```

<!-- OUTPUT -->
```
web
Untagged: nginx:alpine
Untagged: nginx@sha256:2140dad235c1...
Deleted: sha256:d1a364dc548d...
Deleted: sha256:78b5f3f45f39...
...
```

<a id="m3-exercises"></a>
#### Command Exercises

##### Exercise 3.1 — `docker run` basic

Run an Alpine container that prints `hello` and immediately exits.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker run alpine echo hello
```

<!-- OUTPUT -->
```
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
Digest: sha256:....
Status: Downloaded newer image for alpine:latest
hello
```

The image is downloaded on first use, the container runs `echo hello`,
prints the word, then exits with code 0. The stopped container remains
on disk until you `docker rm` it.

</details>

##### Exercise 3.2 — `docker run` with detached + named + port

Run Nginx detached, name the container `mysite`, and expose port 80 as host port 9090.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker run -d --name mysite -p 9090:80 nginx:alpine
```

<!-- OUTPUT -->
```
b4d2e6a7f1c30987654321fedcba9876543210abcdef1234567890abcdef1234
```

Verify with `docker ps` and open http://localhost:9090 — the Nginx
welcome page appears.

</details>

##### Exercise 3.3 — `docker run` combining flags and troubleshooting

Try to start a second container also on host port 9090 named `mysite2`. Observe the error, then successfully start it on a different host port.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker run -d --name mysite2 -p 9090:80 nginx:alpine
```

<!-- OUTPUT -->
```
docker: Error response from daemon: driver failed programming external
connectivity on endpoint mysite2: Bind for 0.0.0.0:9090 failed: port is
already allocated.
```

Diagnosis: only one process can bind host port 9090. Fix by using a
different host port; the container port can stay 80:

<!-- COMMAND -->
```powershell
docker run -d --name mysite2 -p 9091:80 nginx:alpine
docker ps
```

<!-- OUTPUT -->
```
CONTAINER ID   IMAGE          PORTS                  NAMES
b4d2e6a7f1c3   nginx:alpine   0.0.0.0:9090->80/tcp   mysite
c9e3f4d8a2b1   nginx:alpine   0.0.0.0:9091->80/tcp   mysite2
```

Both containers now run; hitting :9090 and :9091 shows the same page
from two independent Nginx containers.

</details>

##### Exercise 3.4 — `docker ps` basic

List only running containers.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker ps
```

<!-- OUTPUT -->
```
CONTAINER ID   IMAGE          COMMAND                  STATUS         PORTS                  NAMES
b4d2e6a7f1c3   nginx:alpine   "/docker-entrypoint.…"   Up 5 minutes   0.0.0.0:9090->80/tcp   mysite
c9e3f4d8a2b1   nginx:alpine   "/docker-entrypoint.…"   Up 2 minutes   0.0.0.0:9091->80/tcp   mysite2
```

</details>

##### Exercise 3.5 — `docker ps` with filters and format

Show ONLY the names and ports of containers whose image is `nginx:alpine`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker ps --filter ancestor=nginx:alpine --format 'table {{.Names}}\t{{.Ports}}'
```

<!-- OUTPUT -->
```
NAMES     PORTS
mysite    0.0.0.0:9090->80/tcp
mysite2   0.0.0.0:9091->80/tcp
```

The `ancestor` filter matches containers created from a specific image.

</details>

##### Exercise 3.6 — `docker ps -a` and quiet mode

Get only the IDs of all containers (running AND stopped) so you can pass them to another command.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker ps -a -q
```

<!-- OUTPUT -->
```
b4d2e6a7f1c3
c9e3f4d8a2b1
5e6f7a8b9c01
```

This is the idiomatic way to script bulk operations. For example:

<!-- COMMAND -->
```powershell
docker rm -f $(docker ps -a -q)
```

removes every container on the host. Use with caution.

</details>

##### Exercise 3.7 — `docker stop` default and custom grace period

Stop `mysite` with the default 10-second SIGTERM grace period.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker stop mysite
```

<!-- OUTPUT -->
```
mysite
```

Docker sends SIGTERM to the main process. Nginx's master handles it and
exits gracefully within a fraction of a second, so `stop` returns almost
instantly. If a process ignores SIGTERM for 10 seconds, Docker sends SIGKILL.

</details>

##### Exercise 3.8 — `docker stop` with custom timeout

Stop `mysite2` with only a 2-second grace period.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker stop -t 2 mysite2
```

<!-- OUTPUT -->
```
mysite2
```

Useful when a container is misbehaving and you don't want to wait 10 seconds.

</details>

##### Exercise 3.9 — `docker stop` on already-stopped container

Try to stop a container that's already stopped. Observe.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker stop mysite
docker stop mysite
```

<!-- OUTPUT -->
```
mysite
mysite
```

Stopping a stopped container is a no-op — it succeeds silently and prints
the name. This is idempotent behavior, safe for scripts.

</details>

##### Exercise 3.10 — `docker start` basic

Start `mysite` again after stopping it.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker start mysite
docker ps
```

<!-- OUTPUT -->
```
mysite
CONTAINER ID   IMAGE          STATUS          NAMES
b4d2e6a7f1c3   nginx:alpine   Up 2 seconds    mysite
```

`docker start` restarts an existing stopped container. All its
configuration (name, ports, env vars, volumes) is preserved.

</details>

##### Exercise 3.11 — `docker start` attached

Create and immediately stop a small container, then use `docker start -a` to run it and see its output.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker create --name counter alpine sh -c 'for i in 1 2 3; do echo tick $i; sleep 1; done'
docker start -a counter
```

<!-- OUTPUT -->
```
counter
tick 1
tick 2
tick 3
```

`-a` attaches your terminal so you see the container's output as it prints.
Without `-a`, `docker start` returns immediately and the output goes only
to the logs (accessible via `docker logs counter`).

</details>

##### Exercise 3.12 — `docker rm` basic

Remove the stopped `counter` container.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker rm counter
```

<!-- OUTPUT -->
```
counter
```

Only stopped containers can be removed by default. Verify with
`docker ps -a` — `counter` no longer appears.

</details>

##### Exercise 3.13 — `docker rm -f` (force)

Try to remove a RUNNING container without `-f`, observe the error, then force-remove it.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker rm mysite
```

<!-- OUTPUT -->
```
Error response from daemon: cannot remove container "/mysite": container is
running: stop the container before removing or force remove
```

<!-- COMMAND -->
```powershell
docker rm -f mysite
```

<!-- OUTPUT -->
```
mysite
```

`-f` sends SIGKILL to the container then removes it. Skip the graceful
shutdown — useful when you don't care about clean exit.

</details>

##### Exercise 3.14 — `docker rm` bulk

Create three throwaway alpine containers that exit immediately, then remove them all in one command.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker run --name t1 alpine echo done
docker run --name t2 alpine echo done
docker run --name t3 alpine echo done
docker rm t1 t2 t3
```

<!-- OUTPUT -->
```
done
done
done
t1
t2
t3
```

`docker rm` accepts multiple names/IDs in one call. In scripts you often
combine with `docker ps -aq`:

<!-- COMMAND -->
```powershell
docker rm $(docker ps -aq --filter status=exited)
```

removes every stopped container.

</details>

##### Exercise 3.15 — `docker pull` basic

Pull the `busybox` image explicitly (before running any container).

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker pull busybox
```

<!-- OUTPUT -->
```
Using default tag: latest
latest: Pulling from library/busybox
c3f5a5c92cb0: Pull complete
Digest: sha256:34b191d63fbc93e25e275bccccdaf3b3d4f4d2f95ab0d0d5c1b96d5d78e26ea1
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
```

Because no tag was given, Docker defaulted to `:latest`.

</details>

##### Exercise 3.16 — `docker pull` specific tag

Pull `python:3.12-slim` explicitly.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker pull python:3.12-slim
```

<!-- OUTPUT -->
```
3.12-slim: Pulling from library/python
c1ec31eb5944: Already exists
5a1c1e8dfd7b: Pull complete
...
Digest: sha256:abc123...
Status: Downloaded newer image for python:3.12-slim
docker.io/library/python:3.12-slim
```

`Already exists` on the first layer means Docker recognized that layer
from a previous pull (perhaps `debian` or `nginx:alpine`) — layers are
shared and only downloaded once.

</details>

##### Exercise 3.17 — `docker pull --platform`

Pull the ARM64 variant of alpine on your x86 laptop (useful for testing multi-arch builds).

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker pull --platform=linux/arm64 alpine:3.20
```

<!-- OUTPUT -->
```
3.20: Pulling from library/alpine
71fc0a10c1a1: Pull complete
Digest: sha256:....
Status: Downloaded newer image for alpine:3.20
docker.io/library/alpine:3.20
```

You now have the arm64 variant cached. Trying to run it directly on
Windows x86_64 would use QEMU emulation via Docker Desktop's rosetta
integration — much slower, but works.

</details>

##### Exercise 3.18 — `docker images` basic

List all images on your machine.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker images
```

<!-- OUTPUT -->
```
REPOSITORY     TAG          IMAGE ID       CREATED        SIZE
nginx          alpine       d1a364dc548d   3 days ago     20.5MB
python         3.12-slim    5a3c1e8f9a10   1 week ago     129MB
alpine         latest       f8c20f8bbcb6   2 weeks ago    7.7MB
alpine         3.20         71fc0a10c1a1   2 weeks ago    7.7MB
busybox        latest       034ac2d9bcbb   4 weeks ago    4.5MB
hello-world    latest       d2c94e258dcb   6 months ago   13.3kB
```

</details>

##### Exercise 3.19 — `docker images` filtered and formatted

Show ONLY the repository:tag and size of images larger than nothing, sorted by size descending — using `--format`.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker images --format 'table {{.Repository}}:{{.Tag}}\t{{.Size}}'
```

<!-- OUTPUT -->
```
REPOSITORY:TAG        SIZE
nginx:alpine          20.5MB
python:3.12-slim      129MB
alpine:latest         7.7MB
alpine:3.20           7.7MB
busybox:latest        4.5MB
hello-world:latest    13.3kB
```

For true sorting you would need to pipe to PowerShell `Sort-Object`,
but this format is much cleaner for scripts.

</details>

##### Exercise 3.20 — `docker images -q` for scripting

Get just the image IDs so you could feed them to another command.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker images -q
```

<!-- OUTPUT -->
```
d1a364dc548d
5a3c1e8f9a10
f8c20f8bbcb6
71fc0a10c1a1
034ac2d9bcbb
d2c94e258dcb
```

Combine with `docker rmi $(docker images -q)` for a nuclear cleanup —
but be careful, that removes everything.

</details>

##### Exercise 3.21 — `docker rmi` basic

Remove the `busybox:latest` image.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker rmi busybox:latest
```

<!-- OUTPUT -->
```
Untagged: busybox:latest
Untagged: busybox@sha256:34b191d63fbc93e25e275bccccdaf3b3d4f4d2f95ab0d0d5c1b96d5d78e26ea1
Deleted: sha256:034ac2d9bcbb...
Deleted: sha256:1c1c1cbcbb...
```

`Untagged` means the tag `busybox:latest` and its digest reference were
removed. `Deleted` means the actual layer blobs were reclaimed from
disk because no other image referenced them.

</details>

##### Exercise 3.22 — `docker rmi` in-use error

Try to remove an image that a stopped container was created from. Observe the error, then force it.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker run --name web nginx:alpine
docker stop web
docker rmi nginx:alpine
```

<!-- OUTPUT -->
```
Error response from daemon: conflict: unable to remove repository reference
"nginx:alpine" (must force) - container 8f1e2b3c4d5e is using its referenced
image d1a364dc548d
```

Fix: force-remove or first delete the container.

<!-- COMMAND -->
```powershell
docker rmi -f nginx:alpine
```

<!-- OUTPUT -->
```
Untagged: nginx:alpine
Deleted: sha256:d1a364dc548d...
```

Note: `-f` removes only the image *tag*. The container `web` (stopped)
still exists and its writable layer still consumes disk. `docker ps -a`
will show the container as `<none>:<none>`. Clean up with `docker rm web`.

</details>

##### Exercise 3.23 — `docker rmi` dangling images

Find and remove all "dangling" images (images with no tag, usually left by rebuilds).

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker images -f dangling=true -q
docker rmi $(docker images -f dangling=true -q)
```

<!-- OUTPUT -->
```
8f7c6b5a4d3e
2e1f0d9c8b7a

Deleted: sha256:8f7c6b5a4d3e...
Deleted: sha256:2e1f0d9c8b7a...
```

Or, more idiomatically:

<!-- COMMAND -->
```powershell
docker image prune -f
```

<!-- OUTPUT -->
```
Total reclaimed space: 156.2MB
```

Both approaches accomplish the same thing.

</details>

<a id="m3-handson"></a>
#### Hands-On Assignment

**Task:** Run Nginx in a container, browse to it, view its logs, stop it, and clean up.

Steps and expected outputs:

1. Pull nginx image:
   <!-- COMMAND -->
   ```powershell
   docker pull nginx:alpine
   ```
   <!-- OUTPUT -->
   ```
   alpine: Pulling from library/nginx
   ... Pull complete ...
   Status: Downloaded newer image for nginx:alpine
   ```

2. Run detached with port mapping and a name:
   <!-- COMMAND -->
   ```powershell
   docker run -d --name my-nginx -p 8080:80 nginx:alpine
   ```
   <!-- OUTPUT -->
   ```
   a1b2c3d4e5f6789...
   ```

3. Verify running:
   <!-- COMMAND -->
   ```powershell
   docker ps
   ```
   <!-- OUTPUT -->
   ```
   CONTAINER ID   IMAGE          PORTS                  NAMES
   a1b2c3d4e5f6   nginx:alpine   0.0.0.0:8080->80/tcp   my-nginx
   ```

4. Open **http://localhost:8080** in browser → see "Welcome to nginx!" page.

5. View recent logs (Nginx logs the incoming request):
   <!-- COMMAND -->
   ```powershell
   docker logs my-nginx
   ```
   <!-- OUTPUT -->
   ```
   /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
   ...
   2025/xx/xx 12:34:56 [notice] 1#1: start worker processes
   172.17.0.1 - - [xx/xxx/2025:12:35:01 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 ..."
   ```

6. Stop the container:
   <!-- COMMAND -->
   ```powershell
   docker stop my-nginx
   ```

7. Remove it:
   <!-- COMMAND -->
   ```powershell
   docker rm my-nginx
   ```

8. Confirm gone:
   <!-- COMMAND -->
   ```powershell
   docker ps -a
   ```

**Acceptance criteria:**

- [ ] `nginx:alpine` image is pulled.
- [ ] Container starts detached with name `my-nginx` and port mapping `8080:80`.
- [ ] Browser at `http://localhost:8080` shows Nginx welcome page.
- [ ] `docker logs my-nginx` shows at least one GET request from your browser.
- [ ] Container stops without force.
- [ ] Container is removed and does not appear in `docker ps -a`.

<a id="m3-miniproject"></a>
#### Mini-Project

##### 🎯 Project Title
**Personal Container Playground Manager**

##### 🎯 Objective

Practice the complete container lifecycle by running, managing, and cleaning up three real-world containers — a web server (Nginx), a second web server (Apache/httpd), and a data store (Redis) — using every CLI command from this milestone. You will document every command and its output, producing a lab notebook that proves you can operate Docker end-to-end.

##### 📋 Requirements

1. **Pull three images:** `nginx:alpine`, `httpd:alpine`, `redis:alpine` — using `docker pull`.
2. **Run all three as detached named containers** with these specific mappings:
   - Nginx: name `pg-nginx`, port `8081:80`, env `SITE_NAME=playground`.
   - Apache: name `pg-httpd`, port `8082:80`, env `SERVER_ROLE=api-mock`.
   - Redis: name `pg-redis`, port `6379:6379`.
3. **Verify all three are running** with a single `docker ps` and a filtered `docker ps` that uses `--format`.
4. **Browse** to `http://localhost:8081` (Nginx) and `http://localhost:8082` (Apache) and confirm both welcome pages appear.
5. **Enter Redis interactively** with `docker exec -it pg-redis redis-cli` and run `SET foo bar` then `GET foo`.
6. **Stop only `pg-httpd`**, verify with `docker ps` (should show only 2 running) and `docker ps -a` (should show 3, one exited).
7. **Restart `pg-httpd`** and verify all three are again running.
8. **Remove all three containers** (force where required).
9. **Remove all three images.**
10. **Verify a clean state:** `docker ps -a` shows no playground containers; `docker images` shows none of the three.

Document every command and expected output in a Markdown file called `playground-lab.md`.

##### 🪜 Step-by-Step Guidance

1. Create `docker-mastery\milestone-3\playground-lab.md` and open it in VS Code.
2. Write a "## Setup" section — run each `docker pull` command and paste its output.
3. Write a "## Run" section — three `docker run -d ...` commands with expected container IDs.
4. Write a "## Verify" section — outputs of `docker ps` and a formatted variant.
5. Write a "## Browser Test" section — screenshots or a note that both welcome pages appeared.
6. Write a "## Redis Test" section — the `docker exec` command, the SET/GET output, and the `exit` step.
7. Write a "## Stop and Restart" section — `docker stop`, verification, `docker start` / `restart`, verification.
8. Write a "## Cleanup" section — `docker rm -f`, `docker rmi`, and the final proof of a clean state.
9. Reflect briefly at the end — three things you learned that you didn't know at the start.

##### 📦 Complete Mini-Project Solution

<details><summary>📦 Complete Mini-Project Solution</summary>

<!-- CODE -->
````markdown
# Personal Container Playground Manager — Lab

## Setup — Pull images

```powershell
docker pull nginx:alpine
docker pull httpd:alpine
docker pull redis:alpine
```

Output (abridged):
```
alpine: Pulling from library/nginx
Status: Downloaded newer image for nginx:alpine

alpine: Pulling from library/httpd
Status: Downloaded newer image for httpd:alpine

alpine: Pulling from library/redis
Status: Downloaded newer image for redis:alpine
```

## Run — Three detached, named containers

```powershell
docker run -d --name pg-nginx -p 8081:80 -e SITE_NAME=playground nginx:alpine
docker run -d --name pg-httpd -p 8082:80 -e SERVER_ROLE=api-mock httpd:alpine
docker run -d --name pg-redis -p 6379:6379 redis:alpine
```

Output:
```
a1b2c3d4e5f67890...
b2c3d4e5f6789012...
c3d4e5f678901234...
```

## Verify — All three running

```powershell
docker ps
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}'
```

Output:
```
CONTAINER ID   IMAGE          PORTS                    NAMES
a1b2c3d4e5f6   nginx:alpine   0.0.0.0:8081->80/tcp     pg-nginx
b2c3d4e5f678   httpd:alpine   0.0.0.0:8082->80/tcp     pg-httpd
c3d4e5f67890   redis:alpine   0.0.0.0:6379->6379/tcp   pg-redis

NAMES       IMAGE          PORTS                    STATUS
pg-nginx    nginx:alpine   0.0.0.0:8081->80/tcp     Up 30 seconds
pg-httpd    httpd:alpine   0.0.0.0:8082->80/tcp     Up 25 seconds
pg-redis    redis:alpine   0.0.0.0:6379->6379/tcp   Up 20 seconds
```

## Browser Test

- http://localhost:8081 → "Welcome to nginx!" ✅
- http://localhost:8082 → "It works!" ✅

## Redis Test — Interactive exec

```powershell
docker exec -it pg-redis redis-cli
```

Inside the Redis shell:
```
127.0.0.1:6379> SET foo bar
OK
127.0.0.1:6379> GET foo
"bar"
127.0.0.1:6379> exit
```

## Stop and Restart

```powershell
docker stop pg-httpd
docker ps                # only 2 running
docker ps -a             # 3 exist, 1 exited
docker start pg-httpd
docker restart pg-nginx  # restart just to demonstrate
docker ps                # all 3 running again
```

Output:
```
pg-httpd
(2 rows shown in docker ps)
(3 rows shown in docker ps -a; pg-httpd status = Exited (0))
pg-httpd
pg-nginx
(3 rows shown again in docker ps)
```

## Cleanup — Remove containers and images

```powershell
docker rm -f pg-nginx pg-httpd pg-redis
docker rmi nginx:alpine httpd:alpine redis:alpine
docker ps -a
docker images
```

Output:
```
pg-nginx
pg-httpd
pg-redis
Untagged: nginx:alpine
Deleted: sha256:...
Untagged: httpd:alpine
Deleted: sha256:...
Untagged: redis:alpine
Deleted: sha256:...
(docker ps -a: no playground containers)
(docker images: no playground images)
```

## Reflections

1. `-p HOST:CONTAINER` uses the *host* port on the left. I had them
   backward at first and got confusing 404s.
2. `docker exec -it <name> <cmd>` runs a *new* process in a running
   container, unlike `docker run` which creates a new container.
3. `docker rm -f` is faster than `stop`+`rm` but skips graceful
   shutdown — fine for playground, dangerous in production for stateful
   services like Redis (data-loss risk).
````

</details>

##### ✅ Verification Checklist

- [ ] `playground-lab.md` exists at `docker-mastery\milestone-3\`.
- [ ] `docker pull` was used for all three images.
- [ ] All three containers were started with `docker run -d`, `--name`, `-p`, and (for two of them) `-e`.
- [ ] `docker ps` showed three running containers with the correct names, ports, and images.
- [ ] `docker ps --format` was used at least once with a Go template.
- [ ] Nginx welcome page opened at `localhost:8081`.
- [ ] Apache welcome page opened at `localhost:8082`.
- [ ] `docker exec -it pg-redis redis-cli` was used and `SET`/`GET` succeeded.
- [ ] `docker stop pg-httpd` succeeded and `docker ps -a` showed one exited container.
- [ ] `docker start` (or `restart`) brought `pg-httpd` back up.
- [ ] `docker rm -f` removed all three containers.
- [ ] `docker rmi` removed all three images.
- [ ] Final `docker ps -a` and `docker images` confirmed a clean state.
- [ ] At least three personal reflections were recorded.

##### 🌟 Bonus Challenges

1. **Add a Postgres container** — pull `postgres:16-alpine`, run with `POSTGRES_PASSWORD=devsecret`, exec into it and run `psql -U postgres -c "SELECT version();"`.
2. **Restart policies** — recreate all three containers with `--restart=unless-stopped`. Quit Docker Desktop and relaunch it; verify the containers come back up automatically.
3. **Resource limits** — recreate the Nginx container with `--memory=64m --cpus=0.25`. Prove the limits are in place with `docker stats pg-nginx` and observe CPU/memory usage against the limits.

<a id="m3-scenario"></a>
#### Scenario (Real-World Use Case)

You are on-call for a small startup. At 2 AM Slack pings you: "The staging API is down." You SSH into the staging VM, run `docker ps`, and see the `api` container is missing. You run `docker ps -a --filter name=api` and see it exited 12 minutes ago with exit code 137 (SIGKILL — likely OOM-killed). You check `docker logs --tail 50 api` and confirm a `fatal: out of memory` line just before exit.

You bring it back with `docker start api` and it stays up for 4 minutes before dying again — same cause. Rather than wake up the backend team, you restart it with a higher memory limit: `docker rm -f api` then `docker run -d --name api --memory=1g --restart=unless-stopped -p 8080:80 ourcompany/api:staging`. It stays up. In the morning you file a ticket with the log excerpt so the team can find the memory leak, then go back to sleep.

Every command you used in that incident — `ps`, `ps -a --filter`, `logs`, `start`, `rm -f`, `run` with flags — came from Milestone 3. The rest of Docker mastery is layered on top of this fluency. Practice these commands until they are muscle memory.

<a id="m3-quiz"></a>
#### Checkpoint Quiz

**Question 1.** What is the difference between `-i`, `-t`, and `-it`?

<details><summary>Click to reveal answer</summary>

`-i` (`--interactive`) keeps stdin open so keyboard input is forwarded to the container. `-t` (`--tty`) allocates a pseudo-terminal so the container thinks it's talking to a real terminal (colors, prompts, Ctrl+C behave). `-it` combines both and is the standard combination for shells. Use `-i` alone for piping input to a container (`echo hi | docker run -i alpine cat`), and `-it` together for anything you'd type into interactively.

</details>

**Question 2.** In `-p 8080:80`, which side is the host and which is the container?

<details><summary>Click to reveal answer</summary>

**HOST_PORT:CONTAINER_PORT** — the LEFT number (`8080`) is the port on your Windows host, the RIGHT number (`80`) is the port inside the container. Your browser talks to `localhost:8080`, Docker forwards that traffic to port 80 inside the container. Getting these backward is one of the most common beginner mistakes.

</details>

**Question 3.** What is the difference between `docker stop` and `docker rm -f`?

<details><summary>Click to reveal answer</summary>

`docker stop` sends SIGTERM (a polite "please shut down") to the container's main process, waits up to 10 seconds by default, then sends SIGKILL if it hasn't exited — but the container remains on disk with its logs and configuration intact and can be restarted with `docker start`. `docker rm -f` sends SIGKILL immediately (no graceful shutdown) *and* deletes the container. Use `stop` for services you might restart; use `rm -f` when you're truly done with the container.

</details>

**Question 4.** Why does `docker rmi nginx:alpine` sometimes fail with "conflict"?

<details><summary>Click to reveal answer</summary>

Because a container (running OR stopped) was created from that image. Docker tracks the reference — deleting the image would corrupt the container's layer chain. Either `docker rm` the containers first, or use `docker rmi -f` to force-untag the image (leaving the container's copy of the layers intact until the container itself is deleted). Use `docker ps -a --filter ancestor=nginx:alpine` to find offending containers.

</details>

**Question 5.** Predict the output of these two commands run back-to-back:

```powershell
docker run --rm -d --name x alpine sleep 30
docker ps -a
```

<details><summary>Click to reveal answer</summary>

The first command launches `alpine sleep 30` detached with the name `x` and `--rm`. It prints the container ID and returns immediately. The container will run in the background for 30 seconds. The second command lists containers, and `x` will appear with STATUS `Up X seconds`. If you wait 30 seconds and rerun `docker ps -a`, the container `x` is *gone* — because `--rm` deleted it automatically when `sleep` exited. Without `--rm`, it would appear as `Exited (0)`.

</details>

[⬆ Back to Table of Contents](#table-of-contents)

---