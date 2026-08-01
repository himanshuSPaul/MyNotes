# Docker Hands-On Exercise Workbook (with Answers)

Companion to the Docker Command Guide. Do these IN ORDER — each level builds
on the last. Type every command yourself; don't copy-paste, that's where the
learning happens.

**How to use this:**
1. Read the task.
2. Try it yourself FIRST (peek at the guide if stuck — that's allowed!).
3. Only then read the **Answer** below the task. Answers appear right after
   each exercise, so cover them with your hand or scroll carefully.
4. ✅ checkpoints tell you how to verify you did it right.

Start at Level 0 even if Docker is already installed — the verification
exercises teach you the client/daemon architecture. All exercises use small
public images.

---

## LEVEL 0 — Installation & Setup

### Exercise 0.1 — Install Docker
Install Docker on your machine:
- **Windows / Mac**: download and install Docker Desktop from
  https://www.docker.com/products/docker-desktop/ and launch it
  (wait for the whale icon to stop animating).
- **Linux (Ubuntu/Debian)**: install Docker Engine.

Then verify the client is installed.

**Answer:**

Linux install:
```bash
curl -fsSL https://get.docker.com | sh
sudo systemctl start docker
sudo systemctl enable docker    # start automatically on boot
```
Verify (all platforms):
```bash
docker --version
```
✅ Prints something like `Docker version 27.x.x`.

### Exercise 0.2 — Is the daemon actually running?
`docker --version` only proves the CLI exists. Prove the Docker daemon
(server) is running and reachable too.

**Answer:**

```bash
docker version        # no dashes — shows CLIENT and SERVER sections
docker info           # daemon-wide info: containers, images, storage driver
```
✅ `docker version` shows a **Server** section without errors. If instead you
get "Cannot connect to the Docker daemon":
- Linux: `sudo systemctl start docker`
- Windows/Mac: Docker Desktop isn't running — start it.

### Exercise 0.3 — (Linux only) Escape sudo
Right now every command probably needs `sudo`. Fix it so your user can run
docker directly, and prove it works.

**Answer:**

```bash
sudo usermod -aG docker $USER
newgrp docker            # or fully log out and back in
docker info              # NO sudo!
```
✅ `docker info` works without sudo and without "permission denied ... 
docker.sock". Windows/Mac users: skip — Docker Desktop handles this.

### Exercise 0.4 — End-to-end smoke test
Run the official test container and read its output carefully — it
describes the exact 4 steps Docker just performed. Can you name them?

**Answer:**

```bash
docker run hello-world
```
The message explains: (1) CLI contacted the daemon, (2) daemon pulled the
hello-world image from Docker Hub, (3) daemon created a container from that
image, (4) daemon streamed its output to your terminal.
✅ You saw "Hello from Docker!" — installation fully verified. You also just
learned the client → daemon → registry architecture without noticing.

### Exercise 0.5 — Explore help
Without using the internet: find out (a) what `docker run -P` (capital P)
does, and (b) three subcommands of `docker image`.

**Answer:**

```bash
docker run --help | grep -A1 '\-P'
docker image --help
```
(a) `-P` publishes ALL exposed ports to random host ports.
(b) e.g. `ls`, `pull`, `rm`, `prune`, `inspect`, `history`.
✅ You now know how to answer your own questions — `--help` works offline
on every command and subcommand.

### Exercise 0.6 — Stop and revive the daemon (Linux; Desktop users: quit/relaunch the app)
Stop the Docker daemon fully, prove it's stopped, then bring it back.

**Answer:**

```bash
sudo systemctl stop docker.socket docker
docker info                    # "Cannot connect to the Docker daemon..."
sudo systemctl start docker
docker info                    # works again
```
✅ You saw both states. Note: stopping the daemon stops all running
containers too — check `docker ps` before doing this for real.

---

## LEVEL 1 — First Containers

### Exercise 1.1 — Hello
Run the hello-world container.

**Answer:**

```bash
docker run hello-world
```
✅ You see "Hello from Docker!"

### Exercise 1.2 — Pull without running
Download the `alpine` image (a 5MB Linux) WITHOUT running it, then list your
images to confirm it's there.

**Answer:**

```bash
docker pull alpine
docker images
```
✅ `alpine` appears in the list with a size around 5–8MB.

### Exercise 1.3 — Your first web server
Run nginx in the background, named `web1`, with your machine's port 8080
connected to the container's port 80. Then open http://localhost:8080.

**Answer:**

```bash
docker run -d --name web1 -p 8080:80 nginx
```
✅ Browser shows "Welcome to nginx!". `docker ps` shows web1 running.

### Exercise 1.4 — Two at once
Run a SECOND nginx called `web2` on port 8081. Prove both are running with
one command.

**Answer:**

```bash
docker run -d --name web2 -p 8081:80 nginx
docker ps
```
✅ Both web1 and web2 listed, different ports. Same image, two containers —
that's the whole point of images vs containers.

### Exercise 1.5 — Lifecycle
Stop web2, confirm it's stopped but still exists, start it again, then
stop AND delete it permanently.

**Answer:**

```bash
docker stop web2
docker ps            # web2 gone from here...
docker ps -a         # ...but still here (status: Exited)
docker start web2
docker rm -f web2    # force remove even though running
docker ps -a         # web2 fully gone
```
✅ `docker ps -a` no longer shows web2.

### Exercise 1.6 — Cleanup reflex
Stop web1, then remove ALL stopped containers with one command.

**Answer:**

```bash
docker stop web1
docker container prune       # answer y
```
✅ `docker ps -a` is empty (or has no stopped containers).

### Exercise 1.7 — Search from the terminal
Without opening a browser, find official images related to `redis`.

**Answer:**

```bash
docker search redis
```
✅ The one marked OFFICIAL is the one you'd normally use.

### Exercise 1.8 — stop vs kill
Run nginx as `victim`. Time `docker stop` on it. Run it again and time
`docker kill`. Why the difference?

**Answer:**

```bash
docker run -d --name victim nginx
time docker stop victim        # ~0.3s — nginx handles SIGTERM promptly
docker rm victim
docker run -d --name victim alpine sleep 3600
time docker stop victim        # ~10s! sleep ignores SIGTERM, Docker waits,
                               # then SIGKILLs after the grace period
docker rm victim
docker run -d --name victim alpine sleep 3600
time docker kill victim        # instant — SIGKILL immediately
docker rm victim
```
✅ stop = polite (SIGTERM, wait, then kill); kill = immediate. The 10-second
wait happens when the process ignores SIGTERM.

### Exercise 1.9 — Restart policies
Run nginx as `phoenix` with `--restart unless-stopped`. Kill it with
`docker kill`. Check `docker ps` a few seconds later. Surprised? Then stop
it properly with `docker stop` — does it come back now?

**Answer:**

```bash
docker run -d --name phoenix --restart unless-stopped nginx
docker kill phoenix
sleep 3 && docker ps           # it's BACK — Docker auto-restarted it
docker stop phoenix
sleep 3 && docker ps           # stays down — "unless-stopped" respects
                               # an explicit stop
docker rm phoenix
```
✅ Crashes trigger restart; a deliberate `stop` doesn't. This is how you keep
services alive across crashes and reboots.

### Exercise 1.10 — Remove an image (and hit the classic error)
Try to `docker rmi nginx` while a stopped nginx container still exists.
Read the error, fix the cause properly (no -f), and remove the image.
Then pull it back — you'll need it later.

**Answer:**

```bash
docker rmi nginx               # Error: image is being used by stopped container...
docker ps -a                   # find the container(s) using it
docker rm <container>          # remove them
docker rmi nginx               # now works
docker pull nginx
```
✅ Images can't be removed while containers (even stopped ones) reference
them. `-f` exists but understanding the dependency is better.

---

## LEVEL 2 — Inside Containers

### Exercise 2.1 — Interactive Linux
Start an alpine container and get a shell inside it. Inside: create a file
`/tmp/hello.txt` containing "i was here", print it, then exit.

**Answer:**

```bash
docker run -it --name box alpine sh
# inside the container:
echo "i was here" > /tmp/hello.txt
cat /tmp/hello.txt
exit
```
✅ You saw the shell prompt change (you were "inside"), and the file printed.

### Exercise 2.2 — The disappearing file
Run a NEW alpine container interactively and check: does /tmp/hello.txt exist?
Why or why not?

**Answer:**

```bash
docker run -it --rm alpine sh
cat /tmp/hello.txt      # No such file!
exit
```
✅ File doesn't exist. Each `docker run` creates a FRESH container from the
image. Your file lives only in the stopped `box` container's writable layer.
(Bonus: `docker start box && docker exec box cat /tmp/hello.txt` — it's
still there in THAT container.)

### Exercise 2.3 — Exec into a running container
Start nginx detached as `web`, then WITHOUT stopping it, open a shell inside,
and find the file containing the welcome page (hint: /usr/share/nginx/html).

**Answer:**

```bash
docker run -d --name web -p 8080:80 nginx
docker exec -it web bash
ls /usr/share/nginx/html      # index.html
cat /usr/share/nginx/html/index.html
exit
```
✅ You saw the HTML of the welcome page. The container kept running the
whole time (`docker ps` confirms).

### Exercise 2.4 — Change a live website
Using `docker exec` (one-liner, no interactive shell), overwrite index.html
with "MY FIRST DOCKER SITE". Refresh the browser.

**Answer:**

```bash
docker exec web sh -c 'echo "MY FIRST DOCKER SITE" > /usr/share/nginx/html/index.html'
```
✅ http://localhost:8080 now shows your text.

### Exercise 2.5 — Logs
Refresh the page a few times, then view web's logs. Then follow them live
(-f) while refreshing again. Ctrl+C to stop following.

**Answer:**

```bash
docker logs web
docker logs -f web     # refresh browser, watch requests appear, Ctrl+C
```
✅ You see GET / requests with status 200.

### Exercise 2.6 — Copy files both ways
Copy index.html from the container to your current directory. Edit it on
your machine (add another line), copy it back, refresh browser.

**Answer:**

```bash
docker cp web:/usr/share/nginx/html/index.html .
echo "edited on my machine" >> index.html
docker cp index.html web:/usr/share/nginx/html/index.html
```
✅ Browser shows both lines.

### Exercise 2.7 — Environment variables
Run alpine with an env var `NAME=<your name>` and have it print
"hello $NAME" and exit. The container should delete itself afterwards.

**Answer:**

```bash
docker run --rm -e NAME=Ravi alpine sh -c 'echo "hello $NAME"'
docker ps -a     # container is gone (--rm)
```
✅ Prints your name; no leftover container.

### Exercise 2.8 — attach vs exec (feel the danger)
Run `docker run -d --name ticker alpine sh -c 'while true; do date; sleep 1; done'`.
(a) Attach to it and watch the output. Detach WITHOUT killing it
(Ctrl+P then Ctrl+Q) and verify it's still running.
(b) Attach again — this time press Ctrl+C. What happened to the container?

**Answer:**

```bash
docker run -d --name ticker alpine sh -c 'while true; do date; sleep 1; done'
docker attach ticker      # dates stream by
# Ctrl+P, Ctrl+Q
docker ps                 # still running ✔
docker attach ticker
# Ctrl+C
docker ps -a              # Exited! Ctrl+C went to the MAIN process
docker rm ticker
```
✅ attach = your keyboard talks to the main process (Ctrl+C kills it);
exec = a separate shell, always safe to exit.

### Exercise 2.9 — The shell-form signal bug (live!)
Build two images from these Dockerfiles and time `docker stop` on each:
```dockerfile
# Dockerfile.shellform          # Dockerfile.execform
FROM alpine                     FROM alpine
CMD sleep 300                   CMD ["sleep", "300"]
```
Predict which stops fast. Then test. (Trick question — check the answer!)

**Answer:**

```bash
docker build -f Dockerfile.shellform -t shellform .
docker build -f Dockerfile.execform -t execform .
docker run -d --name s1 shellform
time docker stop s1            # ~10s: SIGTERM went to /bin/sh, not sleep
docker run -d --name s2 execform
time docker stop s2            # also ~10s here — because sleep itself
                               # ignores SIGTERM! But with a REAL app
                               # (nginx, python) exec form stops instantly
                               # while shell form still hangs 10s.
docker rm s1 s2
```
✅ The lesson stands: exec form delivers signals to YOUR process; shell form
wraps it in /bin/sh which swallows them. Real apps that handle SIGTERM only
shut down cleanly with exec form. Retest with `CMD nginx -g "daemon off;"`
vs `CMD ["nginx", "-g", "daemon off;"]` to see a dramatic difference.

### Exercise 2.10 — Monitoring commands
With nginx running as `web`: (a) show its live CPU/memory once (no stream),
(b) list the processes running inside it without exec-ing in,
(c) show its port mappings.

**Answer:**

```bash
docker run -d --name web -p 8080:80 nginx    # if not already running
docker stats web --no-stream
docker top web
docker port web
```
✅ You can monitor a container entirely from outside.

### Exercise 2.11 — Env files
Create a file `myvars.env` containing `APP=shop` and `MODE=dev` (one per
line). Run alpine loading that file and print both variables.

**Answer:**

```bash
printf "APP=shop\nMODE=dev\n" > myvars.env
docker run --rm --env-file myvars.env alpine sh -c 'echo "$APP in $MODE"'
```
✅ Prints "shop in dev". Env files keep secrets/config out of your shell
history and Dockerfiles.

---

## LEVEL 3 — Data That Survives

### Exercise 3.1 — Prove data loss
Run postgres named `db1` with password `secret`, create nothing, just remove
it, then answer: what happened to any data it wrote?

**Answer:**

```bash
docker run -d --name db1 -e POSTGRES_PASSWORD=secret postgres:16
docker rm -f db1
```
✅ All its data died with it — it lived in the container's writable layer.
This is the problem volumes solve.

### Exercise 3.2 — Named volume survives container death
Create a volume `pgdata`. Run postgres `db1` using it. Create a database
named `school` inside. Destroy the container. Start a NEW container `db2`
with the SAME volume and verify `school` still exists.

**Answer:**

```bash
docker volume create pgdata
docker run -d --name db1 -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data postgres:16
sleep 5    # let postgres boot
docker exec -it db1 psql -U postgres -c "CREATE DATABASE school;"
docker rm -f db1

docker run -d --name db2 -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data postgres:16
sleep 5
docker exec -it db2 psql -U postgres -c "\l"
```
✅ `school` appears in the database list of db2. Data outlived the container.
Clean up: `docker rm -f db2` (keep the volume if you like).

### Exercise 3.3 — Bind mount = live editing
Make a folder `mysite` with an index.html saying "bind mounts are magic".
Run nginx serving THAT folder. Then edit the file on your machine and refresh —
no docker cp needed.

**Answer:**

```bash
mkdir mysite
echo "bind mounts are magic" > mysite/index.html
docker run -d --name site -p 8080:80 \
  -v "$(pwd)/mysite:/usr/share/nginx/html" nginx
# now edit mysite/index.html in any editor, refresh browser
```
✅ Every save shows instantly in the browser. This is how dev workflows work.
Clean up: `docker rm -f site`

### Exercise 3.4 — Read-only mount
Repeat 3.3 but mount read-only. Then exec in and TRY to modify the file
from inside. What happens?

**Answer:**

```bash
docker run -d --name site-ro -p 8081:80 \
  -v "$(pwd)/mysite:/usr/share/nginx/html:ro" nginx
docker exec site-ro sh -c 'echo hacked > /usr/share/nginx/html/index.html'
```
✅ "Read-only file system" error. The container cannot alter your files.
Clean up: `docker rm -f site-ro`

### Exercise 3.5 — Where does a volume actually live?
Inspect the `pgdata` volume and find its Mountpoint on your disk.
(Linux: you can even `sudo ls` it. Docker Desktop: it's inside the VM.)

**Answer:**

```bash
docker volume inspect pgdata
# "Mountpoint": "/var/lib/docker/volumes/pgdata/_data"
sudo ls /var/lib/docker/volumes/pgdata/_data     # Linux only
```
✅ Volumes are just directories Docker manages for you.

### Exercise 3.6 — Same thing, --mount syntax
Re-run the nginx bind-mount from 3.3, but using `--mount` instead of `-v`.

**Answer:**

```bash
docker run -d --name site2 -p 8082:80 \
  --mount type=bind,source="$(pwd)/mysite",target=/usr/share/nginx/html nginx
```
✅ Identical result, more explicit syntax — and it FAILS LOUDLY if the source
folder doesn't exist (unlike -v, which silently creates it). Try it with a
typo in source to see. Clean up: `docker rm -f site2`

### Exercise 3.7 — tmpfs: RAM-only files
Run alpine with a tmpfs at /scratch. Write a file there, verify it exists,
then check: does the file appear anywhere on your host disk? What happens
to it when the container stops?

**Answer:**

```bash
docker run -it --rm --tmpfs /scratch alpine sh
echo secret > /scratch/temp.txt
cat /scratch/temp.txt
df -h /scratch          # filesystem type: tmpfs (RAM)
exit
```
✅ File lived in RAM only, never touched disk, vanished on exit — perfect
for sensitive temp data.

### Exercise 3.8 — Volume housekeeping
Create two throwaway volumes `junk1`, `junk2`, use neither, then remove all
unused volumes in one command. Confirm `pgdata` survived IF a container
still references it (otherwise it's gone too — check first!).

**Answer:**

```bash
docker volume create junk1
docker volume create junk2
docker volume ls
docker volume prune          # removes ALL volumes not used by any container
docker volume ls
```
✅ junk1/junk2 gone. Warning learned: `volume prune` deletes any unreferenced
volume — including ones with data you care about. Name your important volumes
and keep a container or use `docker volume rm` selectively.

---

## LEVEL 4 — Build Your Own Images

### Exercise 4.1 — Simplest possible image
Create a Dockerfile that starts FROM alpine and CMD prints "built by me".
Build it as `greet:v1` and run it.

**Answer:**

Dockerfile:
```dockerfile
FROM alpine
CMD ["echo", "built by me"]
```
```bash
docker build -t greet:v1 .
docker run --rm greet:v1
```
✅ Prints "built by me".

### Exercise 4.2 — A real Python app
Create `app.py`:
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
class H(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200); self.end_headers()
        self.wfile.write(b"Hello from my container!\n")
HTTPServer(("0.0.0.0", 5000), H).serve_forever()
```
Write a Dockerfile (python:3.12-slim base, workdir /app, copy the file,
run it), build as `pyapp:v1`, run it on port 5000, and curl it.

**Answer:**

Dockerfile:
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```
```bash
docker build -t pyapp:v1 .
docker run -d --name pyapp -p 5000:5000 pyapp:v1
curl http://localhost:5000
```
✅ "Hello from my container!"

### Exercise 4.3 — Watch the cache work
Run the build from 4.2 again unchanged — note the speed and "CACHED" lines.
Now change the text in app.py and rebuild. Which steps re-ran and why?

**Answer:**

```bash
docker build -t pyapp:v1 .          # everything CACHED, ~instant
# edit app.py, change the message
docker build -t pyapp:v2 .          # FROM/WORKDIR cached; COPY app.py
                                    # and everything AFTER it re-ran
```
✅ Only steps from the changed COPY onward rebuilt. That's layer caching —
and why instruction order matters.

### Exercise 4.4 — .dockerignore
In the same folder create a fake junk file: `dd if=/dev/zero of=big.junk bs=1M count=50`
(or just create any large file). Rebuild and note the "transferring context" size.
Now add a `.dockerignore` containing `big.junk` and rebuild. Compare.

**Answer:**

```bash
echo "big.junk" > .dockerignore
docker build -t pyapp:v2 .
```
✅ Build context dropped from ~50MB to a few kB. Junk never reached the build.

### Exercise 4.5 — CMD vs ENTRYPOINT
Build image `pinger` from this Dockerfile:
```dockerfile
FROM alpine
ENTRYPOINT ["ping", "-c", "3"]
CMD ["localhost"]
```
Run it three ways: with nothing, with `google.com`, and overriding the
entrypoint to get a shell. Explain each result.

**Answer:**

```bash
docker build -t pinger .
docker run --rm pinger                      # pings localhost (CMD = default arg)
docker run --rm pinger google.com           # pings google (your arg REPLACES CMD)
docker run --rm -it --entrypoint sh pinger  # shell (ENTRYPOINT itself overridden)
```
✅ ENTRYPOINT = fixed program, CMD = default arguments.

### Exercise 4.6 — Tag and inspect layers
Tag `pyapp:v2` additionally as `pyapp:latest`. List images (note: same
IMAGE ID, two tags — no extra disk). View its layer history.

**Answer:**

```bash
docker tag pyapp:v2 pyapp:latest
docker images | grep pyapp
docker history pyapp:v2
```
✅ Both tags share one IMAGE ID; history shows one layer per Dockerfile step.

### Exercise 4.7 — ADD's special power
Create `files.tar` containing a text file (`tar cf files.tar index.html`).
Build two images: one using `COPY files.tar /data/`, one using
`ADD files.tar /data/`. Run each and list /data. What's different?

**Answer:**

```bash
tar cf files.tar index.html
# Dockerfile.copy: FROM alpine / COPY files.tar /data/ / CMD ["ls","/data"]
# Dockerfile.add:  FROM alpine / ADD files.tar /data/  / CMD ["ls","/data"]
docker build -f Dockerfile.copy -t copytest . && docker run --rm copytest
# -> files.tar
docker build -f Dockerfile.add -t addtest . && docker run --rm addtest
# -> index.html   (auto-extracted!)
```
✅ ADD silently extracted the tar. Useful occasionally, surprising usually —
hence "use COPY unless you specifically want extraction".

### Exercise 4.8 — Healthcheck
Add a HEALTHCHECK to your pyapp Dockerfile that curls localhost:5000
(python:3.12-slim has no curl — use python's urllib instead). Rebuild, run,
and watch `docker ps` go from (health: starting) to (healthy). Then query
just the health status via inspect.

**Answer:**

Add before CMD:
```dockerfile
HEALTHCHECK --interval=5s --timeout=3s \
  CMD python -c "import urllib.request;urllib.request.urlopen('http://localhost:5000')" || exit 1
```
```bash
docker build -t pyapp:v3 .
docker run -d --name pyapp3 -p 5001:5000 pyapp:v3
docker ps                     # (health: starting) -> (healthy)
docker inspect -f '{{.State.Health.Status}}' pyapp3
docker rm -f pyapp3
```
✅ Status reads "healthy". Orchestrators use exactly this signal to decide
whether to route traffic or restart your container.

### Exercise 4.9 — Tags lie, digests don't
Find the digest of your local nginx image. Then pull nginx BY DIGEST and
confirm docker recognizes it's the same content.

**Answer:**

```bash
docker images --digests | grep nginx
docker inspect -f '{{index .RepoDigests 0}}' nginx
docker pull nginx@sha256:<paste-digest-here>
# "Image is up to date" — same content, verified by fingerprint
```
✅ You pinned exact content. This is what "pin by digest" means in production.

### Exercise 4.10 — Peek inside a multi-arch manifest
Inspect the manifest of `alpine` and answer: how many CPU architectures
does the single `alpine:latest` tag serve?

**Answer:**

```bash
docker manifest inspect alpine | grep architecture
```
✅ Typically 7+ (amd64, arm64, arm/v6, arm/v7, 386, ppc64le, s390x...).
One tag, many images — Docker picked yours automatically at pull time.

### Exercise 4.11 — The layer-deletion trap
Build this image and check its size. Then "fix" it and compare:
```dockerfile
FROM alpine
RUN dd if=/dev/zero of=/big.file bs=1M count=100
RUN rm /big.file
```

**Answer:**

```bash
docker build -t trap .
docker images trap            # ~110MB — deletion in a LATER layer didn't help!
docker history trap           # see the 100MB layer still there
```
Fix — delete in the SAME layer:
```dockerfile
FROM alpine
RUN dd if=/dev/zero of=/big.file bs=1M count=100 && rm /big.file
```
```bash
docker build -t fixed .
docker images fixed           # ~8MB
```
✅ Layers are append-only; later deletion only hides files, doesn't remove
them. This is copy-on-write made visible — and why real Dockerfiles chain
`install && cleanup` in one RUN.

### Exercise 4.12 — Move an image without a registry
Save your `greet:v1` image to a tar file. Delete the image. Restore it from
the tar. Run it to prove it works.

**Answer:**

```bash
docker save -o greet.tar greet:v1
docker rmi greet:v1
docker load -i greet.tar
docker run --rm greet:v1
```
✅ "built by me" again. This is how images travel to air-gapped machines.

### Exercise 4.13 — Snapshot a container with commit
Run alpine interactively, install curl inside (`apk add curl`), exit.
Commit that container as image `alpine-curl` and verify a NEW container
from it already has curl. Then reflect: why is a Dockerfile still better?

**Answer:**

```bash
docker run -it --name custom alpine sh
apk add curl
exit
docker commit custom alpine-curl
docker run --rm alpine-curl curl --version
docker rm custom
```
✅ curl is baked in. But the image is now a mystery box — nobody can see HOW
it was made or rebuild it. Dockerfiles are reproducible recipes; commit is
for quick experiments and forensics only.

---

## LEVEL 5 — Networking

### Exercise 5.1 — Name resolution fails on default bridge
Run two alpine containers `a1` and `a2` (default network, keep them alive
with `sleep 3600`). From a1, try to ping a2 BY NAME. What happens?

**Answer:**

```bash
docker run -d --name a1 alpine sleep 3600
docker run -d --name a2 alpine sleep 3600
docker exec a1 ping -c 2 a2
```
✅ "bad address 'a2'" — the DEFAULT bridge has no name resolution.

### Exercise 5.2 — Custom network fixes it
Create network `mynet`, connect both containers to it, ping by name again.

**Answer:**

```bash
docker network create mynet
docker network connect mynet a1
docker network connect mynet a2
docker exec a1 ping -c 2 a2
```
✅ Ping succeeds. Custom networks give automatic DNS by container name.
Clean up: `docker rm -f a1 a2`

### Exercise 5.3 — Real two-tier app by hand
On `mynet`, run postgres as `db` (password secret). Then run
`alpine` interactively on the same network and verify you can reach
the database host: `nc -zv db 5432` (install with `apk add netcat-openbsd`
or use `ping db`).

**Answer:**

```bash
docker run -d --name db --network mynet -e POSTGRES_PASSWORD=secret postgres:16
docker run -it --rm --network mynet alpine sh
# inside:
ping -c 2 db
exit
```
✅ The "app" container found the database by the hostname `db` — exactly how
multi-container apps connect. Clean up: `docker rm -f db`

### Exercise 5.4 — Inspect the wiring
Use inspect to find: (a) the IP address of a running container, and
(b) which containers are attached to mynet.

**Answer:**

```bash
docker run -d --name web --network mynet nginx
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web
docker network inspect mynet
```
✅ You can read IPs and membership. Clean up: `docker rm -f web && docker network rm mynet`

### Exercise 5.5 — Reach your host machine
Start a simple server ON YOUR MACHINE (not in Docker):
`python3 -m http.server 9000` (leave it running). From inside a container,
fetch it. Remember: the container's localhost is NOT your machine.

**Answer:**

```bash
# separate terminal on host: python3 -m http.server 9000

# Docker Desktop (Win/Mac):
docker run --rm alpine wget -qO- http://host.docker.internal:9000

# Linux:
docker run --rm --add-host host.docker.internal:host-gateway \
  alpine wget -qO- http://host.docker.internal:9000
```
✅ You got the directory listing HTML. This is THE pattern for a container
talking to a database/app running directly on your laptop.

### Exercise 5.6 — network none and host
(a) Run alpine with NO network and prove it can't reach anything.
(b) (Linux only) Run nginx with host networking — no -p flag — and explain
why http://localhost:80 works anyway.

**Answer:**

```bash
docker run --rm --network none alpine ping -c1 8.8.8.8
# "Network unreachable" — total isolation

docker run -d --name hostnet --network host nginx     # Linux
curl localhost:80        # works WITHOUT -p: the container shares the
                         # host's network stack directly, no mapping needed
docker rm -f hostnet
```
✅ none = airgap; host = no isolation at all (fast, but no port mapping and
containers can conflict over ports).

### Exercise 5.7 — Borrow another container's network
Run nginx as `web` (no published ports). Then run an alpine container that
SHARES web's network namespace and curl nginx via localhost from inside it.

**Answer:**

```bash
docker run -d --name web nginx
docker run --rm --network container:web alpine \
  wget -qO- http://localhost:80 | head -3
docker rm -f web
```
✅ From the second container, nginx was on ITS OWN localhost — they share one
network stack. This is exactly how debug sidecars and Kubernetes pods work.

---

## LEVEL 6 — Docker Compose

### Exercise 6.1 — Composify the two-tier app
In a new folder, write a `docker-compose.yml` with:
- service `web`: image nginx, port 8080:80
- service `db`: image postgres:16, env POSTGRES_PASSWORD=secret,
  named volume `dbdata` for its data
Bring it up detached, list what's running, then check web's logs.

**Answer:**

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - dbdata:/var/lib/postgresql/data

volumes:
  dbdata:
```
```bash
docker compose up -d
docker compose ps
docker compose logs web
```
✅ Both services up; Compose auto-created a network — web can reach `db`
by name (try: `docker compose exec web ping -c1 db` — install ping first
or just trust exercise 5).

### Exercise 6.2 — Exec, restart, and one-off runs
(a) Open a psql shell in the db service. (b) Restart only web.
(c) Run a ONE-OFF temporary container of the db service image that just
prints its postgres version and disappears.

**Answer:**

```bash
docker compose exec db psql -U postgres      # \q to quit
docker compose restart web
docker compose run --rm db postgres --version
```
✅ Three different tools: exec (into running), restart (service-targeted),
run (fresh throwaway container).

### Exercise 6.3 — Build within compose
Add your Level-4 `pyapp` as a third service built from its Dockerfile
(copy app.py + Dockerfile into an `app/` subfolder):
```yaml
  app:
    build: ./app
    ports:
      - "5000:5000"
```
Rebuild and restart everything with one command, then curl port 5000.

**Answer:**

```bash
docker compose up -d --build
curl http://localhost:5000
```
✅ "Hello from my container!" — Compose built AND ran your image.

### Exercise 6.4 — Teardown levels
(a) Stop everything but keep containers. (b) Bring back up. (c) Tear down
containers+network but KEEP the db volume. (d) Verify the volume survived.

**Answer:**

```bash
docker compose stop
docker compose start
docker compose down          # no -v flag!
docker volume ls             # dbdata still listed
```
✅ `down` removed containers and the network; the named volume survived.
(`down -v` would have deleted it too.)

### Exercise 6.5 — env_file and variable substitution
(a) Move the postgres password out of the YAML into a `.env`-style file
loaded with `env_file:`. (b) Also parametrize the web port using `${WEB_PORT}`
substitution from a `.env` file. Verify with `docker compose config`.

**Answer:**

`db.env`:
```
POSTGRES_PASSWORD=secret
```
`.env` (auto-read by compose for substitution):
```
WEB_PORT=8080
```
compose file changes:
```yaml
  web:
    image: nginx
    ports:
      - "${WEB_PORT}:80"
  db:
    image: postgres:16
    env_file:
      - db.env
```
```bash
docker compose config      # shows resolved port 8080 and the env var
docker compose up -d
```
✅ Secrets/config now live outside the YAML. `.env` = substitution INTO the
yaml; `env_file:` = variables INTO the container. Different things!

### Exercise 6.6 — Profiles
Add a `pgadmin` (or just `busybox` with `sleep 3600`) service under a
profile called `tools`. Verify normal `up` skips it, and `--profile tools up`
includes it.

**Answer:**

```yaml
  toolbox:
    image: busybox
    command: sleep 3600
    profiles: ["tools"]
```
```bash
docker compose up -d && docker compose ps        # no toolbox
docker compose --profile tools up -d
docker compose ps                                # toolbox running
docker compose --profile tools down
```
✅ Profiles keep optional/debug services out of the default startup.

### Exercise 6.7 — Scale
Remove the host-port binding from web (scaling needs it — why?), then run
3 replicas of web. Verify with ps.

**Answer:**

```yaml
  web:
    image: nginx        # ports: removed — 3 containers can't share host 8080!
```
```bash
docker compose up -d --scale web=3
docker compose ps       # web-1, web-2, web-3
```
✅ Three identical containers. (Fixed host ports conflict; real setups put a
load balancer in front or use a port RANGE like "8080-8082:80".)

### Exercise 6.8 — Dev/prod override files
Create `docker-compose.prod.yml` that changes only WEB_PORT... actually,
that only changes web's port to 80 and adds `restart: always` to both
services. Run dev (default) vs prod (explicit -f files) and diff their
`config` output.

**Answer:**

`docker-compose.prod.yml`:
```yaml
services:
  web:
    ports: !override
      - "80:80"
    restart: always
  db:
    restart: always
```
```bash
docker compose config > dev.txt
docker compose -f docker-compose.yml -f docker-compose.prod.yml config > prod.txt
diff dev.txt prod.txt
```
✅ Same base file, different environments layered on top — no duplication.
(A file named `docker-compose.override.yml` would merge automatically.)

### Exercise 6.9 — Compose watch (needs Compose v2.22+)
Add a `develop.watch` sync rule for your app service (sync ./app to /app),
run `docker compose watch`, edit app.py's message, and confirm the change
lands in the container.

**Answer:**

```yaml
  app:
    build: ./app
    develop:
      watch:
        - action: sync+restart
          path: ./app
          target: /app
```
```bash
docker compose watch
# edit app/app.py message, save, wait a moment
curl localhost:5000     # new message!
```
✅ Live dev loop with no manual rebuilds. (`docker compose version` if watch
isn't recognized — you may need to update.)

---

## LEVEL 7 — Ops Skills

### Exercise 7.1 — Disk audit and cleanup
Check how much disk Docker is using, then reclaim space from stopped
containers, unused networks and dangling images.

**Answer:**

```bash
docker system df
docker system prune          # answer y
docker system df             # compare
```
✅ RECLAIMABLE shrank.

### Exercise 7.2 — Memory limit in action
Run nginx limited to 64MB RAM and 0.5 CPU. Verify the limits via stats.

**Answer:**

```bash
docker run -d --name tiny --memory 64m --cpus 0.5 nginx
docker stats tiny --no-stream
```
✅ MEM LIMIT column shows 64MiB. Clean up: `docker rm -f tiny`

### Exercise 7.3 — Debug a crashing container
Run: `docker run -d --name crasher alpine sh -c 'echo starting; sleep 2; echo dying; exit 1'`
It will exit. Using only docker commands, find (a) its exit code and
(b) its last words.

**Answer:**

```bash
docker ps -a                 # STATUS: Exited (1) ...
docker inspect -f '{{.State.ExitCode}}' crasher    # 1
docker logs crasher          # starting / dying
docker rm crasher
```
✅ Exit code 1, logs show both echoes. This is your standard crash-triage flow.

### Exercise 7.4 — Formatted ps
Print running containers as a table showing ONLY name, image and status.

**Answer:**

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```
✅ Clean three-column output.

### Exercise 7.5 — The nuclear drill (do it safely)
Stop ALL running containers and remove ALL containers using command
substitution one-liners. (Make sure nothing important is running!)

**Answer:**

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```
✅ `docker ps -a` is empty. You now hold the keys — use responsibly.

### Exercise 7.6 — pause, rename, diff, wait
With nginx running as `web`:
(a) pause it and try loading the page (it should hang), then unpause;
(b) rename it to `frontend`;
(c) modify a file inside it, then use one command to list what changed
    vs the original image;
(d) in one terminal run `docker wait frontend`, in another `docker stop frontend`
    — what did wait print?

**Answer:**

```bash
docker run -d --name web -p 8080:80 nginx
docker pause web           # browser: page hangs (processes frozen)
docker unpause web
docker rename web frontend
docker exec frontend sh -c 'echo x > /usr/share/nginx/html/new.html'
docker diff frontend       # A /usr/share/nginx/html/new.html (+ C dirs)
docker wait frontend       # (blocks...)
# other terminal: docker stop frontend
# wait prints: 0   <- the exit code
docker rm frontend
```
✅ Four small tools: freeze, rename, audit changes, and block-until-exit
(handy in scripts).

### Exercise 7.7 — Update a running container
Run nginx with 64MB memory and NO restart policy. Without recreating it,
raise memory to 256MB and set restart to unless-stopped. Verify both.

**Answer:**

```bash
docker run -d --name live nginx
docker update --memory 256m --memory-swap 256m --restart unless-stopped live
docker inspect -f 'mem={{.HostConfig.Memory}} restart={{.HostConfig.RestartPolicy.Name}}' live
docker rm -f live
```
✅ Changed live, zero downtime.

### Exercise 7.8 — Watch events
In one terminal: `docker events --filter event=start --filter event=die`.
In another, run and stop a couple of containers. Watch the stream. Then
stop the stream and re-query only the last 5 minutes.

**Answer:**

```bash
docker events --filter event=start --filter event=die
# other terminal: docker run --rm alpine echo hi ; docker run --rm alpine echo bye
# Ctrl+C the stream
docker events --since 5m --until 0m
```
✅ Every start/die appeared in real time. This is the raw feed monitoring
tools consume.

### Exercise 7.9 — Log rotation
Run a deliberately chatty container WITH rotation limits:
alpine looping `yes hello` is too brutal — use a 1-line-per-second logger.
Verify the log file config took effect via inspect.

**Answer:**

```bash
docker run -d --name chatty \
  --log-opt max-size=10k --log-opt max-file=2 \
  alpine sh -c 'while true; do date; sleep 0.1; done'
docker inspect -f '{{.HostConfig.LogConfig}}' chatty
# {json-file map[max-file:2 max-size:10k]}
sleep 20 && docker logs chatty | wc -l    # capped — old lines rotated away
docker rm -f chatty
```
✅ Logs capped at ~2×10k instead of growing forever. Set this globally in
daemon.json on any real machine.

---

## LEVEL 8 — Expert Exercises

### Exercise 8.1 — Multi-stage build shrinkage
Build a tiny Go-style demo without Go — use the classic pattern with a
"builder" that has tools and a final stage that doesn't. Compare sizes:
```dockerfile
# Dockerfile.fat — everything in one stage
FROM python:3.12
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```
vs a multi-stage that ships from python:3.12-slim while "building" in full
python:3.12. Measure both.

**Answer:**

```dockerfile
# Dockerfile.slim
FROM python:3.12 AS builder
WORKDIR /app
COPY app.py .
# (imagine compiling assets / installing build deps here)

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/app.py .
CMD ["python", "app.py"]
```
```bash
docker build -f Dockerfile.fat -t app:fat .
docker build -f Dockerfile.slim -t app:slim .
docker images | grep app
# fat ~1GB, slim ~130MB
```
✅ ~85% smaller by shipping only the final stage. Build tools never reach
production.

### Exercise 8.2 — BuildKit cache mounts
Create `requirements.txt` with `flask` and `requests`. Build twice with a
normal `RUN pip install`, then switch to a cache mount and build twice again
(touch app.py between builds to bust the layer). Compare the SECOND build
times.

**Answer:**

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip pip install -r requirements.txt
```
```bash
docker build -t cached .            # first: downloads packages
touch app.py
docker build -t cached .            # second: pip layer re-runs BUT hits the
                                    # persistent pip cache — much faster
```
✅ Even when the layer cache busts, package downloads are reused. Huge on
big dependency trees.

### Exercise 8.3 — Build secrets done right
Prove the danger first: build an image with `ARG TOKEN` /
`RUN echo "using $TOKEN"` passing `--build-arg TOKEN=supersecret`, then find
the secret with `docker history`. Now do it properly with a secret mount and
confirm history shows nothing.

**Answer:**

```bash
# BAD
docker build --build-arg TOKEN=supersecret -t leaky .
docker history leaky      # supersecret VISIBLE in the layer commands!

# GOOD
echo supersecret > token.txt
```
```dockerfile
RUN --mount=type=secret,id=tok cat /run/secrets/tok > /dev/null && echo "used secret"
```
```bash
docker build --secret id=tok,src=token.txt -t sealed .
docker history sealed     # no trace of the secret
```
✅ ARGs and ENVs live in image history forever; secret mounts exist only
during that RUN.

### Exercise 8.4 — Buildx multi-platform (dry run)
List your buildx builders and build your greet image for BOTH amd64 and
arm64 (without pushing — just prove it builds).

**Answer:**

```bash
docker buildx ls
docker buildx create --use --name multi 2>/dev/null || docker buildx use multi
docker buildx build --platform linux/amd64,linux/arm64 -t greet:multi .
```
✅ Both platforms compiled (emulated via QEMU). Add `--push` with a registry
tag to publish a real multi-arch image.

### Exercise 8.5 — Contexts
Create a context pointing at your OWN docker socket under a different name,
switch to it, run `docker ps`, and switch back. (Simulates remote hosts
without needing a server.)

**Answer:**

```bash
docker context create local2 --docker "host=unix:///var/run/docker.sock"
docker context use local2
docker ps                    # same daemon, via the new context
docker context use default
docker context rm local2
```
✅ Same commands, different target — with `host=ssh://user@server` this
becomes full remote management.

### Exercise 8.6 — Troubleshooting drills (break things on purpose)
Cause each error, then fix it:
(a) a port conflict, (b) a name conflict, (c) a container that exits
immediately with a bad command, (d) exec into a container that has no bash.

**Answer:**

```bash
# (a)
docker run -d --name p1 -p 8080:80 nginx
docker run -d --name p2 -p 8080:80 nginx     # "port is already allocated"
docker run -d --name p2 -p 8081:80 nginx     # fix: different host port

# (b)
docker run -d --name p1 nginx                # "name /p1 already in use"
docker rm -f p1                              # fix, then rerun

# (c)
docker run -d --name oops alpine wrongcmd    # exits instantly
docker ps -a                                 # Exited (127) = command not found
docker logs oops                             # "exec: wrongcmd: not found"

# (d)
docker run -d --name mini alpine sleep 300
docker exec -it mini bash                    # FAILS — no bash in alpine
docker exec -it mini sh                      # fix: use sh
docker rm -f p1 p2 oops mini
```
✅ You've now SEEN the four most common beginner errors and their fixes —
they'll never slow you down again.

---

## CAPSTONE PROJECTS

### Capstone A — Full app, from scratch, no peeking
Build a Flask + Postgres visit counter, all containerized:
1. `app/app.py` — Flask app that increments and shows a counter stored in Postgres
2. `app/requirements.txt` — flask, psycopg2-binary
3. `app/Dockerfile` — slim base, cached dependency layer, exec-form CMD,
   non-root user, .dockerignore
4. `docker-compose.yml` — app (built) + db (postgres:16) + named volume,
   env vars for the DB connection, depends_on
5. Prove persistence: refresh counter to 5, `docker compose down` (no -v),
   `up -d` again — counter continues from 5.

**Answer (one working version):**

`app/app.py`:
```python
import os, time
import psycopg2
from flask import Flask

app = Flask(__name__)

def conn():
    return psycopg2.connect(
        host=os.environ["DB_HOST"],
        user="postgres",
        password=os.environ["DB_PASS"],
    )

# wait for db, create table
for _ in range(10):
    try:
        c = conn(); break
    except Exception:
        time.sleep(2)
with c, c.cursor() as cur:
    cur.execute("CREATE TABLE IF NOT EXISTS visits (n int); ")
    cur.execute("INSERT INTO visits SELECT 0 WHERE NOT EXISTS (SELECT 1 FROM visits);")

@app.route("/")
def home():
    with conn() as c, c.cursor() as cur:
        cur.execute("UPDATE visits SET n = n + 1 RETURNING n;")
        (n,) = cur.fetchone()
    return f"Visit number {n}\n"

app.run(host="0.0.0.0", port=5000)
```

`app/requirements.txt`:
```
flask
psycopg2-binary
```

`app/Dockerfile`:
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd -m appuser
USER appuser
EXPOSE 5000
CMD ["python", "app.py"]
```

`app/.dockerignore`:
```
__pycache__
*.pyc
```

`docker-compose.yml`:
```yaml
services:
  app:
    build: ./app
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_PASS=secret
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - dbdata:/var/lib/postgresql/data

volumes:
  dbdata:
```

Run and verify:
```bash
docker compose up -d --build
curl localhost:5000    # Visit number 1 ... repeat to 5
docker compose down
docker compose up -d
curl localhost:5000    # Visit number 6 — persistence proven!
```
✅ If the counter survived `down`+`up`, you've used: Dockerfile layering,
non-root user, exec CMD, compose networking (DB_HOST=db!), env vars,
named volumes, depends_on. That's ~80% of daily Docker.

### Capstone B — Ship it
Take Capstone A's app image and:
1. Tag it properly (`yourname/visits:1.0`) and push to Docker Hub
   (free account) — then delete it locally and pull it back.
2. Add a HEALTHCHECK to the Dockerfile and verify `docker ps` shows (healthy).
3. Run `docker scout quickview` on it and read the results.
4. Rebuild with `--no-cache` and time the difference vs a cached build.

**Answer:**

```bash
# 1
docker login
docker tag <project>-app yourname/visits:1.0
docker push yourname/visits:1.0
docker rmi yourname/visits:1.0
docker pull yourname/visits:1.0

# 2 — add to Dockerfile before CMD:
# HEALTHCHECK --interval=10s --timeout=3s CMD python -c "import urllib.request;urllib.request.urlopen('http://localhost:5000')"
docker compose up -d --build
docker ps        # ... (healthy)

# 3
docker scout quickview yourname/visits:1.0

# 4
time docker build -t visits:cached app/
time docker build --no-cache -t visits:nocache app/
```
✅ You've completed the full local → registry → verified cycle.

---

## SELF-TEST — Can You Answer These Cold?

1. Difference between an image and a container?
2. What does `-d`, `-it`, `-p 8080:80`, `--rm` each do?
3. Why did `/tmp/hello.txt` vanish in exercise 2.2?
4. When would you use a bind mount vs a named volume?
5. Why does Dockerfile instruction order affect build speed?
6. CMD vs ENTRYPOINT in one sentence each?
7. Why couldn't a1 ping a2 by name on the default bridge?
8. What's the difference between `compose stop`, `compose down`,
   and `compose down -v`?
9. A container exits immediately after `run -d` — first two commands you type?
10. Why should production never use the `:latest` tag?

**Answers:**

1. Image = read-only template; container = a running (or stopped) instance
   of it with its own writable layer.
2. `-d` background; `-it` interactive terminal; `-p` map host:container port;
   `--rm` auto-delete on exit.
3. Each `docker run` makes a NEW container from the clean image; the file
   lived only in the old container's writable layer.
4. Bind mount for live development with files you edit on your machine;
   named volume for data Docker should manage (databases, persistence).
5. Each instruction is a cached layer; the first changed instruction
   invalidates everything after it.
6. ENTRYPOINT is the fixed program; CMD is the default (replaceable)
   command/arguments.
7. The default bridge network has no DNS between containers; custom
   networks do.
8. stop = pause containers; down = remove containers+network;
   down -v = also delete volumes (data!).
9. `docker logs <name>` then `docker ps -a` (exit code).
10. `latest` is a movable tag — deployments become unreproducible and can
    silently change; pin versions or digests.

If you got 8+ of these without looking — congratulations, you're no longer
"way below beginner". You're a Docker user.
