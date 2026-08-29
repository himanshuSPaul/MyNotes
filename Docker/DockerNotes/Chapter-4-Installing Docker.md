
### Milestone 2: Installing Docker on Windows

---
📍 **Milestone 2 of 20** | 🟢 **Beginner** | ⏱ **Estimated Time: 1–2 hours** | **Prerequisites:** Milestone 1
---

<a id="m2-theory"></a>
#### Theory

Before you can run a single container, Docker must be installed and healthy on your Windows machine. This milestone walks you from a completely fresh Windows laptop to a working Docker Desktop install with your first container running. Do not skim — a broken install is the #1 reason beginners quit Docker.

##### The Windows-Specific Challenge

Recall from Milestone 1 that Linux containers require a Linux kernel. Windows does not have a Linux kernel — it has the Windows NT kernel. So how does Docker Desktop run Linux containers on Windows? Answer: it uses the **Windows Subsystem for Linux version 2 (WSL2)**, a Microsoft-provided lightweight Linux VM that ships with Windows 10/11 and provides a real Linux kernel to programs running on Windows. Docker Desktop installs its Linux daemon (`dockerd`) inside WSL2 and exposes it to your Windows command prompt through a socket. From your point of view, you type `docker run nginx` in PowerShell and it "just works" — under the hood, Windows PowerShell is talking to a Linux daemon inside a Linux VM.

There is also an older option — running Docker on the **Hyper-V** hypervisor — but WSL2 is faster, uses less RAM, and is the recommended default on modern Windows. This milestone assumes WSL2.

##### System Requirements

To install Docker Desktop on Windows with WSL2, you need:

- **Windows version:** Windows 10 64-bit version 22H2 (Home, Pro, Enterprise, or Education), or Windows 11 64-bit (Home, Pro, Enterprise, or Education). Older versions are not supported.
- **CPU:** 64-bit processor with **Second Level Address Translation (SLAT)**. Every Intel Core i-series CPU from 2010 onward has this, as does every AMD Ryzen and modern AMD FX chip. If your CPU is more than about 12 years old, check.
- **CPU virtualization:** must be **enabled in BIOS/UEFI**. On Intel this is called "VT-x" or "Intel Virtualization Technology"; on AMD it is called "AMD-V" or "SVM Mode". This is turned on in most business laptops by default but often off on consumer/gaming PCs.
- **RAM:** minimum 4 GB, recommended 8 GB or more. Docker Desktop uses ~2 GB just to run the WSL2 VM and services.
- **Disk:** at least 10 GB free for Docker Desktop plus space for images and containers.
- **WSL2 kernel:** Docker Desktop's installer will prompt you to install the WSL2 Linux kernel update if it is missing.

##### What Gets Installed

When you run the Docker Desktop installer on Windows, it installs:

1. **The Docker Desktop application** — a graphical management UI in your Start menu.
2. **The Docker CLI (`docker`)** — added to your Windows `PATH` so it runs from PowerShell or Command Prompt.
3. **Docker Compose (`docker compose`)** — bundled as a plugin (no separate install).
4. **A WSL2 distribution called `docker-desktop`** — a minimal Linux VM containing `dockerd` and `containerd`.
5. **A WSL2 distribution called `docker-desktop-data`** (older versions) — a separate VM for image and volume storage. In current versions, this may be a single distro.
6. **A local Kubernetes cluster (optional)** — you can turn on Kubernetes in Docker Desktop settings.
7. **The Docker Scout, Docker Extensions, and other tools** — accessible from the Docker Desktop UI.

<a id="m2-commands"></a>
#### Commands

The commands introduced in this milestone are:

- `wsl --status` — check WSL version and default distro.
- `wsl --list --verbose` — list installed WSL distributions.
- `wsl --install` — install WSL and Ubuntu (first-time setup).
- `wsl --set-default-version 2` — make WSL2 the default.
- `wsl --update` — update the WSL kernel.
- `systeminfo` — show Windows version and virtualization support (PowerShell).
- `docker version` — show client and server version details.
- `docker info` — show detailed daemon state.
- `docker system info` — alias of `docker info`.
- `docker run hello-world` — run the canonical first-container test.

<a id="m2-flags"></a>
#### Command Options/Flags

##### `docker version` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--format` | `-f` | Format output using a Go template | `docker version --format '{{.Server.Version}}'` |
| `--help` | — | Show help for the command | `docker version --help` |

##### `docker info` flags

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--format` | `-f` | Format output using a Go template or `json` | `docker info --format json` |
| `--help` | — | Show help for the command | `docker info --help` |

##### `docker run` flags (introduced here; complete table in Milestone 3)

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--rm` | — | Remove container automatically when it exits | `docker run --rm hello-world` |

##### `wsl` flags (Windows PowerShell)

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `--status` | — | Show WSL configuration | `wsl --status` |
| `--list` | `-l` | List installed distributions | `wsl -l -v` |
| `--verbose` | `-v` | Verbose output (with `--list`) | `wsl --list --verbose` |
| `--install` | — | Install WSL and default Linux distro | `wsl --install` |
| `--set-default-version` | — | Set default WSL version (1 or 2) | `wsl --set-default-version 2` |
| `--update` | — | Update the WSL Linux kernel | `wsl --update` |
| `--shutdown` | — | Shut down all running WSL VMs | `wsl --shutdown` |

<a id="m2-examples"></a>
#### Examples

##### Step 1 — Check your Windows version

Open PowerShell as Administrator (right-click Start → "Terminal (Admin)" or "Windows PowerShell (Admin)") and run:

<!-- COMMAND -->
```powershell
winver
```

A dialog appears showing your Windows edition and version. You need Windows 10 22H2+ or Windows 11.

Alternatively, from PowerShell:

<!-- COMMAND -->
```powershell
systeminfo | Select-String "OS Name","OS Version"
```

<!-- OUTPUT -->
```
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22631 N/A Build 22631
```

##### Step 2 — Verify CPU virtualization is enabled

<!-- COMMAND -->
```powershell
systeminfo | Select-String "Hyper-V"
```

<!-- OUTPUT -->
```
Hyper-V Requirements:      VM Monitor Mode Extensions: Yes
                           Virtualization Enabled In Firmware: Yes
                           Second Level Address Translation: Yes
                           Data Execution Prevention Available: Yes
```

All four must be `Yes`. If "Virtualization Enabled In Firmware" is `No`, you must reboot into your BIOS/UEFI and enable VT-x (Intel) or SVM (AMD).

##### Step 3 — Install and enable WSL2

<!-- COMMAND -->
```powershell
wsl --install
```

<!-- OUTPUT -->
```
Installing: Virtual Machine Platform
Virtual Machine Platform has been installed.
Installing: Windows Subsystem for Linux
Windows Subsystem for Linux has been installed.
Installing: Ubuntu
Ubuntu has been installed.
The requested operation is successful. Changes will not be effective
until the system is rebooted.
```

Reboot when prompted. After reboot:

<!-- COMMAND -->
```powershell
wsl --set-default-version 2
wsl --update
wsl --status
```

<!-- OUTPUT -->
```
Default Distribution: Ubuntu
Default Version: 2

Windows Subsystem for Linux was last updated on 2025-10-15
The WSL kernel version is: 5.15.167.4-1
```

##### Step 4 — Download and install Docker Desktop

1. Go to **https://www.docker.com/products/docker-desktop/**
2. Click **"Download for Windows"** → **"Docker Desktop Installer.exe"** (about 700 MB).
3. Double-click the installer.
4. On the "Configuration" screen, leave **"Use WSL 2 instead of Hyper-V"** checked (this is the default).
5. Click **OK** and wait 5–10 minutes.
6. When installation completes, click **"Close and log out"** — you must sign out of Windows for group membership changes to take effect.
7. Sign back in. Docker Desktop icon (a whale with containers on its back) will appear in the system tray.
8. Docker Desktop launches; accept the license agreement; skip or complete the sign-in prompt; wait for the whale icon to stop animating.

##### Step 5 — Verify installation with `docker version`

Open a new PowerShell window and run:

<!-- COMMAND -->
```powershell
docker version
```

<!-- OUTPUT -->
```
Client:
 Cloud integration: v1.0.35+desktop.16
 Version:           27.3.1
 API version:       1.47
 Go version:        go1.22.7
 Git commit:        ce12230
 Built:             Fri Sep 20 11:41:00 2024
 OS/Arch:           windows/amd64
 Context:           desktop-linux

Server: Docker Desktop 4.35.0 (172550)
 Engine:
  Version:          27.3.1
  API version:      1.47 (minimum version 1.24)
  Go version:       go1.22.7
  Git commit:       41ca978
  Built:            Fri Sep 20 11:41:00 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.21
  GitCommit:        472731909fa34bd7bc9c087e4c27943f9835f111
 runc:
  Version:          1.1.13
  GitCommit:        v1.1.13-0-g58aa920
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

**Explaining every line:**

- **`Client:`** — this section describes the `docker` CLI you just typed on your Windows machine.
  - `Cloud integration` — plugin for Docker Cloud features.
  - `Version` — the client version (`27.3.1`). Should match or be compatible with the server.
  - `API version` — the version of the Docker REST API the client speaks.
  - `Go version` — Docker is written in Go; this is the compiler version used to build it.
  - `Git commit` — the exact git commit hash Docker was built from (useful for bug reports).
  - `Built` — timestamp of the build.
  - `OS/Arch` — `windows/amd64` means the client binary is a Windows 64-bit executable.
  - `Context` — `desktop-linux` means the CLI is currently talking to the Linux daemon inside Docker Desktop. Other contexts might point to remote Docker hosts.
- **`Server:`** — this section describes `dockerd` running inside WSL2.
  - `Docker Desktop 4.35.0 (172550)` — Docker Desktop application version.
  - `Engine.Version` — the daemon version.
  - `Engine.API version` — API version the daemon speaks; `minimum version 1.24` means older clients back to that API version still work.
  - `Engine.OS/Arch` — `linux/amd64` — confirms the daemon runs on a Linux kernel inside WSL2.
  - `Engine.Experimental: false` — no experimental features enabled.
- **`containerd:`** — the container runtime Docker uses under the hood (industry-standard, donated to CNCF).
- **`runc:`** — the low-level tool that actually creates containers via kernel syscalls.
- **`docker-init:`** — a tiny "init" process that runs as PID 1 inside each container (only when you pass `--init`), reaping zombie processes.

If you see `Server:` details, congratulations — the daemon is running. If you see `error during connect: this error may indicate that the docker daemon is not running`, Docker Desktop hasn't finished starting up. Wait 30 seconds and try again.

##### Step 6 — Verify installation with `docker info`

<!-- COMMAND -->
```powershell
docker info
```

<!-- OUTPUT -->
```
Client:
 Version:    27.3.1
 Context:    desktop-linux
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.) v0.17.1-desktop.1
  compose: Docker Compose (Docker Inc.) v2.29.7-desktop.1
  ...

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 27.3.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 472731909fa34bd7bc9c087e4c27943f9835f111
 runc version: v1.1.13-0-g58aa920
 init version: de40ad0
 Security Options:
  seccomp
   Profile: unconfined
  cgroupns
 Kernel Version: 5.15.167.4-microsoft-standard-WSL2
 Operating System: Docker Desktop
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 7.663GiB
 Name: docker-desktop
 ID: 5a08b7c3-....
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 HTTP Proxy: http.docker.internal:3128
 HTTPS Proxy: http.docker.internal:3128
 No Proxy: hubproxy.docker.internal
 Labels:
 Experimental: false
 Insecure Registries:
  hubproxy.docker.internal:5555
  127.0.0.0/8
 Live Restore Enabled: false
```

**Explaining every line:**

- **`Containers`** — how many containers exist locally (running + paused + stopped).
- **`Images`** — how many images are cached locally.
- **`Server Version`** — daemon version (should match client where possible).
- **`Storage Driver: overlay2`** — the filesystem driver Docker uses to stack image layers. `overlay2` is the modern default.
- **`Backing Filesystem: extfs`** — the underlying filesystem beneath overlay2 (ext4 inside WSL2).
- **`Native Overlay Diff: true`** — kernel supports native diffs for overlay2 (faster builds).
- **`Logging Driver: json-file`** — how container stdout/stderr is captured to disk. Default is JSON files under `/var/lib/docker/containers/`.
- **`Cgroup Driver: cgroupfs`** — how Docker talks to cgroups. Alternative is `systemd`.
- **`Cgroup Version: 2`** — cgroup v2 is the modern unified hierarchy.
- **`Plugins.Volume/Network/Log`** — built-in drivers for each area.
- **`Swarm: inactive`** — Docker Swarm cluster mode is not initialized (that's Milestone 15).
- **`Runtimes`** — available container runtimes. Default is `runc`.
- **`Security Options.seccomp`** — a Linux syscall-filtering mechanism used to sandbox containers.
- **`Kernel Version`** — the Linux kernel version inside WSL2. Notice `microsoft-standard-WSL2` in the name.
- **`Operating System: Docker Desktop`** — signals this daemon is running under Docker Desktop's minimal Linux.
- **`OSType: linux`** — this daemon runs Linux containers.
- **`Architecture: x86_64`** — 64-bit Intel/AMD.
- **`CPUs`** — how many CPU cores are allocated to the WSL2 VM (default is often all cores).
- **`Total Memory`** — RAM allocated to WSL2 (adjustable in Docker Desktop settings).
- **`Docker Root Dir`** — where Docker stores images, containers, volumes (inside WSL2).
- **`Live Restore Enabled: false`** — if `true`, containers keep running when the daemon restarts.

##### Step 7 — Run your first container

<!-- COMMAND -->
```powershell
docker run hello-world
```

<!-- OUTPUT -->
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:e2fc4e5012d16e7fe466f5291c476431beaa1f9b90a5c2125b493ed28e2aba57
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**Explaining every line:**

- `Unable to find image 'hello-world:latest' locally` — Docker checked its local image cache; the image was not there.
- `latest: Pulling from library/hello-world` — Docker is downloading the `latest` tag of the `hello-world` image from the `library` (official images) namespace on Docker Hub.
- `c1ec31eb5944: Pull complete` — one layer downloaded and unpacked. `hello-world` has just one tiny layer.
- `Digest: sha256:e2fc...` — the cryptographic hash of the image. This is a content-addressable identifier — if two images have the same digest, they are bit-for-bit identical.
- `Status: Downloaded newer image for hello-world:latest` — cache updated.
- `Hello from Docker! ... To generate this message, Docker took the following steps:` — the container itself is a tiny program that prints this text.
- Steps 1–4 in the output describe *exactly* the Client → Daemon → Registry → Container flow you learned in Milestone 1. Read them and match each step to your mental model.

If you see this output, **your Docker install is 100% functional**. You have completed the hardest part of learning Docker.

##### Configuring Docker Desktop Settings

Open Docker Desktop from the system tray → click the ⚙️ (gear) icon → Settings.

- **General** — auto-start, telemetry, use containerd for image store (experimental).
- **Resources → Advanced** — set CPU cores, memory (RAM), swap, and disk image size allocated to WSL2. For most laptops, 4 CPUs and 4–6 GB RAM is a good starting point.
- **Resources → WSL Integration** — enable the toggle for each installed WSL distro (like Ubuntu) so you can also use the `docker` CLI from inside `wsl -d Ubuntu`.
- **Docker Engine** — a JSON editor for `daemon.json`. This is the daemon configuration file. Example customizations:

<!-- CODE -->
```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "insecure-registries": [],
  "registry-mirrors": []
}
```

After editing, click **Apply & Restart** — the daemon reloads.

- **Kubernetes** — enable a single-node Kubernetes cluster (adds ~1 GB RAM).
- **Software Updates** — check for new Docker Desktop releases.

##### Troubleshooting

- **"WSL 2 installation is incomplete"** — Run `wsl --update` in an admin PowerShell, reboot, and restart Docker Desktop.
- **"Virtualization is disabled"** — Reboot into BIOS/UEFI, enable VT-x (Intel) or SVM (AMD), save and exit.
- **"Hardware assisted virtualization and data execution protection must be enabled in the BIOS"** — Same as above.
- **Hyper-V conflicts** (e.g., you use VirtualBox and Docker Desktop won't start) — Older VirtualBox versions conflicted with Hyper-V. Update VirtualBox to 6.1+ and update Windows, or run only one hypervisor at a time.
- **"Docker Desktop is starting..." forever** — Restart Docker Desktop from the tray icon → Troubleshoot → Restart. If that fails, Troubleshoot → Reset to factory defaults (destroys all containers and images).
- **"docker: command not found"** in PowerShell — Docker Desktop failed to add itself to your `PATH`. Sign out and back in, or reboot.
- **`docker version` shows client but no server** — the daemon has not finished starting; wait or restart Docker Desktop.
- **"error during connect: ... The system cannot find the file specified"** — the WSL2 distro `docker-desktop` is stopped. Restart Docker Desktop, or `wsl --shutdown` then relaunch.

<a id="m2-exercises"></a>
#### Command Exercises

##### Exercise 2.1 — `docker version` basic

Run `docker version` and identify (a) your client version, (b) your server version, (c) whether they match, and (d) the OS/Arch of the server.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker version
```

<!-- OUTPUT -->
```
Client:
 Version:  27.3.1
 OS/Arch:  windows/amd64

Server: Docker Desktop 4.35.0
 Engine:
  Version:  27.3.1
  OS/Arch:  linux/amd64
```

Answers:
(a) Client version: 27.3.1 (yours may differ)
(b) Server version: 27.3.1
(c) They match (client and server are the same major.minor)
(d) Server OS/Arch: linux/amd64 — the daemon runs on Linux
    inside WSL2, even though you typed the command in Windows.

</details>

##### Exercise 2.2 — `docker version` with format flag

Use the `--format` flag with a Go template to display *only* the server version number, nothing else.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker version --format '{{.Server.Version}}'
```

<!-- OUTPUT -->
```
27.3.1
```

This is useful in scripts — you can capture just the version into a
variable without parsing full text.

</details>

##### Exercise 2.3 — `docker version` troubleshooting

Simulate a broken daemon: from Docker Desktop tray icon, click "Quit Docker Desktop". Now run `docker version` and observe the difference. Then restart Docker Desktop and confirm it recovers.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker version
```

<!-- OUTPUT -->
```
Client:
 Version:  27.3.1
 ...
error during connect: Get "http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.47/version":
open //./pipe/docker_engine: The system cannot find the file specified.
```

Diagnosis: the CLI (client) still works — it printed the client block —
but it cannot reach the daemon over the named pipe because Docker Desktop
is stopped. Fix:

1. Launch Docker Desktop from the Start menu.
2. Wait until the tray icon stops animating.
3. Re-run `docker version` — the Server block reappears.

</details>

##### Exercise 2.4 — `docker info` basic

Run `docker info` and answer: (a) how many images and containers exist, (b) what is the storage driver, (c) what is the total memory allocated to the daemon.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker info
```

<!-- OUTPUT -->
```
Server:
 Containers: 1
  Running: 0
  Paused: 0
  Stopped: 1
 Images: 1
 Storage Driver: overlay2
 Total Memory: 7.663GiB
```

Answers:
(a) 1 container (stopped) and 1 image (from hello-world)
(b) Storage Driver: overlay2
(c) Total Memory: 7.663 GiB (yours may differ; adjustable in
    Docker Desktop → Settings → Resources)

</details>

##### Exercise 2.5 — `docker info` with JSON format

Output `docker info` as JSON and identify the field name for the kernel version.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker info --format json
```

<!-- OUTPUT -->
```json
{
  "ID": "5a08b7c3-...",
  "Containers": 1,
  "Images": 1,
  "ServerVersion": "27.3.1",
  "KernelVersion": "5.15.167.4-microsoft-standard-WSL2",
  "OperatingSystem": "Docker Desktop",
  "OSType": "linux",
  "Architecture": "x86_64",
  ...
}
```

The field name is `KernelVersion`. In scripts you could extract it with:

<!-- COMMAND -->
```powershell
docker info --format '{{.KernelVersion}}'
```

</details>

##### Exercise 2.6 — `docker info` filtered output

Use `docker info` with a Go template to print only three lines: `CPUs`, `Total Memory`, and `Kernel Version`, each on its own line.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker info --format 'CPUs: {{.NCPU}}{{"\n"}}Memory: {{.MemTotal}}{{"\n"}}Kernel: {{.KernelVersion}}'
```

<!-- OUTPUT -->
```
CPUs: 8
Memory: 8225849344
Kernel: 5.15.167.4-microsoft-standard-WSL2
```

Note: `MemTotal` is in bytes here (not the human-readable "GiB" you
see in plain `docker info` output).

</details>

##### Exercise 2.7 — `docker system info` basic

Confirm that `docker system info` is an alias for `docker info` by comparing their output.

<details><summary>💡 Solution</summary>

<!-- COMMAND -->
```powershell
docker system info | Out-File info1.txt
docker info | Out-File info2.txt
Compare-Object (Get-Content info1.txt) (Get-Content info2.txt)
```

<!-- OUTPUT -->
```
(no output — files are identical)
```

Confirmed. `docker system info` and `docker info` produce identical
output. `docker system` is the "system-management" command group, and
`info` is one of its sub-commands (others include `df`, `prune`, `events`).

</details>

##### Exercise 2.8 — Explore Docker Desktop GUI

Open Docker Desktop → Settings → Resources → Advanced. Note current CPU, Memory, and Swap. Change memory to 4 GB, click Apply & Restart, then verify with `docker info` that `Total Memory` reflects the change.

<details><summary>💡 Solution</summary>

<!-- OUTPUT -->
```
Steps:
1. Docker Desktop tray icon → Settings → Resources → Advanced
2. Note current values (e.g., Memory: 8 GB, CPUs: 8, Swap: 1 GB)
3. Drag Memory slider to 4 GB
4. Click "Apply & Restart"
5. Wait ~30 seconds for daemon to restart
6. Run:
```

<!-- COMMAND -->
```powershell
docker info --format '{{.MemTotal}}'
```

<!-- OUTPUT -->
```
4114677760
```

4114677760 bytes ≈ 3.83 GiB (WSL2 reserves a bit of overhead, so
the reported total is slightly less than the 4 GB you allocated).
Change memory back to your original value when done.

</details>

<a id="m2-handson"></a>
#### Hands-On Assignment

**Task:** From a fresh terminal, prove your Docker installation is working end-to-end.

Steps:

1. Open a new PowerShell window.
2. Run `docker version` and confirm both Client and Server sections appear.
3. Run `docker info` and identify: number of containers, number of images, storage driver, kernel version, total memory.
4. Run `docker run --rm hello-world`.
5. Run `docker run --rm -it ubuntu bash`. This drops you into an interactive shell inside an Ubuntu container. Type `cat /etc/os-release` to prove you are inside Ubuntu, then type `exit` to leave.
6. Confirm the ubuntu container was auto-removed by running `docker ps -a` — it should not appear (the `--rm` flag removed it on exit).

**Acceptance criteria:**

- [ ] `docker version` shows both Client and Server sections.
- [ ] `docker info` runs without error and reports at least 1 image.
- [ ] `hello-world` container printed the "Hello from Docker!" message.
- [ ] You successfully entered an Ubuntu container's shell and confirmed the OS.
- [ ] `docker ps -a` shows the ubuntu container is gone (thanks to `--rm`).

<a id="m2-miniproject"></a>
#### Mini-Project

##### 🎯 Project Title
**Docker Environment Health Check Script**

##### 🎯 Objective

Build a reusable PowerShell script (`docker-healthcheck.ps1`) that anyone on your team could run on a fresh Windows laptop to determine if their Docker installation is healthy. This mirrors what real DevOps engineers do — no one wants to walk 200 developers through the same 10-step manual check individually.

##### 📋 Requirements

Your script must:

1. **Check Docker is installed** — verify the `docker` command is on the PATH.
2. **Print the Docker client version.**
3. **Check the Docker daemon is reachable** — try `docker info` and detect failure.
4. **Print the Docker server (daemon) version.**
5. **Report WSL2 status** — run `wsl --status` and print result.
6. **Report resources allocated** — CPUs and memory from `docker info`.
7. **Run `hello-world`** with `--rm` and capture whether it printed the success message.
8. **Print a color-coded summary report** at the end: green ✅ for each check that passed, red ❌ for each that failed, with a final overall PASS/FAIL verdict.
9. **Exit code** — the script must exit with code `0` if everything passed and `1` if anything failed (so it can be used in CI).

##### 🪜 Step-by-Step Guidance

1. Create `docker-mastery\milestone-2\docker-healthcheck.ps1`.
2. Start each check as its own PowerShell function returning `$true` or `$false` and pushing a message to a `$results` array.
3. Use `Get-Command docker -ErrorAction SilentlyContinue` to test presence.
4. Wrap each Docker call in `try { ... } catch { ... }` so a failure doesn't crash the whole script.
5. Capture command output with `2>&1` to see both stdout and stderr.
6. For color-coded output, use `Write-Host -ForegroundColor Green` and `Write-Host -ForegroundColor Red`.
7. At the end, count failures and `exit ($failures -gt 0 ? 1 : 0)`.
8. Test the happy path (Docker running). Then quit Docker Desktop and rerun — the script should detect the daemon failure and exit 1.

##### 📦 Complete Mini-Project Solution

<details><summary>📦 Complete Mini-Project Solution</summary>

<!-- CODE -->
```powershell
# docker-healthcheck.ps1
# Docker Environment Health Check
# Usage: powershell -ExecutionPolicy Bypass -File .\docker-healthcheck.ps1

$results = @()
$failures = 0

function Add-Result {
    param([string]$Name, [bool]$Passed, [string]$Detail)
    $script:results += [PSCustomObject]@{
        Name   = $Name
        Passed = $Passed
        Detail = $Detail
    }
    if (-not $Passed) { $script:failures++ }
}

Write-Host ""
Write-Host "==== Docker Environment Health Check ====" -ForegroundColor Cyan
Write-Host ""

# 1. Is `docker` on the PATH?
$dockerCmd = Get-Command docker -ErrorAction SilentlyContinue
if ($dockerCmd) {
    Add-Result "Docker CLI installed" $true $dockerCmd.Source
} else {
    Add-Result "Docker CLI installed" $false "docker command not found on PATH"
}

# 2. Docker client version
try {
    $clientVer = docker version --format '{{.Client.Version}}' 2>&1
    Add-Result "Docker client version" $true $clientVer
} catch {
    Add-Result "Docker client version" $false $_.Exception.Message
}

# 3. Docker daemon reachable?
try {
    $info = docker info --format json 2>&1 | ConvertFrom-Json
    Add-Result "Docker daemon reachable" $true "OK"
} catch {
    Add-Result "Docker daemon reachable" $false "Cannot connect to daemon (is Docker Desktop running?)"
    $info = $null
}

# 4. Docker server version
if ($info) {
    Add-Result "Docker server version" $true $info.ServerVersion
} else {
    Add-Result "Docker server version" $false "unavailable"
}

# 5. WSL2 status
try {
    $wsl = (wsl --status 2>&1) -join " "
    if ($wsl -match "Default Version:\s*2") {
        Add-Result "WSL2 status" $true "default version 2"
    } else {
        Add-Result "WSL2 status" $false $wsl
    }
} catch {
    Add-Result "WSL2 status" $false $_.Exception.Message
}

# 6. Resources allocated
if ($info) {
    $cpus  = $info.NCPU
    $memGB = [math]::Round($info.MemTotal / 1GB, 2)
    Add-Result "Allocated resources" $true "CPUs=$cpus  Memory=${memGB}GiB"
} else {
    Add-Result "Allocated resources" $false "unavailable"
}

# 7. Run hello-world
try {
    $hello = docker run --rm hello-world 2>&1
    if ($hello -match "Hello from Docker!") {
        Add-Result "hello-world container" $true "runs successfully"
    } else {
        Add-Result "hello-world container" $false ($hello -join " ")
    }
} catch {
    Add-Result "hello-world container" $false $_.Exception.Message
}

# 8. Print report
Write-Host ""
Write-Host "==== Summary ====" -ForegroundColor Cyan
foreach ($r in $results) {
    if ($r.Passed) {
        Write-Host ("[PASS] {0,-30} : {1}" -f $r.Name, $r.Detail) -ForegroundColor Green
    } else {
        Write-Host ("[FAIL] {0,-30} : {1}" -f $r.Name, $r.Detail) -ForegroundColor Red
    }
}
Write-Host ""

# 9. Exit code
if ($failures -eq 0) {
    Write-Host "OVERALL: PASS  Docker environment is healthy." -ForegroundColor Green
    exit 0
} else {
    Write-Host "OVERALL: FAIL  $failures check(s) failed." -ForegroundColor Red
    exit 1
}
```

**Sample run — healthy environment:**

<!-- COMMAND -->
```powershell
powershell -ExecutionPolicy Bypass -File .\docker-healthcheck.ps1
```

<!-- OUTPUT -->
```
==== Docker Environment Health Check ====

==== Summary ====
[PASS] Docker CLI installed          : C:\Program Files\Docker\Docker\resources\bin\docker.exe
[PASS] Docker client version         : 27.3.1
[PASS] Docker daemon reachable       : OK
[PASS] Docker server version         : 27.3.1
[PASS] WSL2 status                   : default version 2
[PASS] Allocated resources           : CPUs=8  Memory=7.66GiB
[PASS] hello-world container         : runs successfully

OVERALL: PASS  Docker environment is healthy.
```

**Sample run — Docker Desktop quit:**

<!-- OUTPUT -->
```
==== Summary ====
[PASS] Docker CLI installed          : C:\Program Files\Docker\Docker\resources\bin\docker.exe
[PASS] Docker client version         : 27.3.1
[FAIL] Docker daemon reachable       : Cannot connect to daemon (is Docker Desktop running?)
[FAIL] Docker server version         : unavailable
[PASS] WSL2 status                   : default version 2
[FAIL] Allocated resources           : unavailable
[FAIL] hello-world container         : Cannot connect to the Docker daemon

OVERALL: FAIL  4 check(s) failed.
```

</details>

##### ✅ Verification Checklist

- [ ] Script file `docker-healthcheck.ps1` exists at `docker-mastery\milestone-2\`.
- [ ] Script runs to completion without a PowerShell parse error.
- [ ] Each of the 7 checks appears in the summary as either [PASS] or [FAIL].
- [ ] "Docker CLI installed" check passes on your machine.
- [ ] "Docker client version" prints a version like `27.x.x`.
- [ ] "Docker daemon reachable" passes when Docker Desktop is running.
- [ ] "Docker server version" prints the daemon version.
- [ ] "WSL2 status" confirms default version 2.
- [ ] "Allocated resources" prints CPU count and memory in GiB.
- [ ] "hello-world container" prints "runs successfully".
- [ ] Final line says `OVERALL: PASS  Docker environment is healthy.` in green.
- [ ] Exit code is `0` when healthy — check with `echo $LASTEXITCODE`.
- [ ] Stopping Docker Desktop and rerunning produces `OVERALL: FAIL` and exit code `1`.

##### 🌟 Bonus Challenges

1. **Add a disk-space check.** Query `docker system df --format json` and warn if the reclaimable space is over 10 GB (suggest `docker system prune`).
2. **Add a network check.** Try `docker run --rm alpine ping -c 1 8.8.8.8` and confirm containers can reach the internet.
3. **Convert the script to output HTML or JSON.** Add a `-Format` parameter that accepts `Text`, `JSON`, or `HTML` and writes the report accordingly — useful for feeding into an internal dashboard.

<a id="m2-scenario"></a>
#### Scenario (Real-World Use Case)

You are the new lead engineer on a team of 25 developers scattered across three continents. Every Monday, roughly two developers report to Slack that "Docker broke on my machine over the weekend" — sometimes because Windows Update replaced the WSL2 kernel, sometimes because a corporate anti-virus quarantined `dockerd`, sometimes because they filled the disk. Each incident costs you 15–45 minutes of hand-holding, and you have burned an entire day per month on Docker troubleshooting.

You take the health-check script from this milestone, expand it with three more checks (disk, network, corporate registry connectivity), publish it to the team's internal wiki, and add a rule to your Slack triage bot: "If you say 'Docker is broken', the bot replies with a link to the script and asks you to paste the output." Within two weeks, the average Docker-related interruption drops from 30 minutes to 3 minutes because 90% of issues are self-diagnosed by the script's `[FAIL]` lines.

That is what installation and verification skills look like at production scale: not typing `docker version` once, but automating the health checks so an entire team can self-serve.

<a id="m2-quiz"></a>
#### Checkpoint Quiz

**Question 1.** Why does Docker Desktop for Windows install a WSL2 distribution called `docker-desktop`?

<details><summary>Click to reveal answer</summary>

Because Linux containers require a Linux kernel, and Windows does not natively provide one. `docker-desktop` is a minimal Linux VM (delivered as a WSL2 distro) that provides the Linux kernel and runs the Docker daemon (`dockerd`). Your Windows-side `docker` CLI talks to that daemon over a socket, giving you the illusion of Linux containers running on Windows.

</details>

**Question 2.** In `docker version` output, what does the `Context: desktop-linux` line under the Client section mean?

<details><summary>Click to reveal answer</summary>

A Docker **context** is a named endpoint the CLI talks to. `desktop-linux` is the default context created by Docker Desktop and points to the Linux daemon inside WSL2. You can list contexts with `docker context ls` and switch with `docker context use <name>` — useful when managing multiple remote Docker hosts.

</details>

**Question 3.** True or false: Running `docker run hello-world` after a fresh install must download an image before running the container.

<details><summary>Click to reveal answer</summary>

**True.** Because your local image cache is empty on a fresh install, Docker prints `Unable to find image 'hello-world:latest' locally` and pulls it from Docker Hub. On a second run, the image is cached and only the container is created — much faster and no download output.

</details>

**Question 4.** Your colleague reports that after a Windows Update, `docker version` shows only the Client block and errors out on the Server. What is the most likely fix?

<details><summary>Click to reveal answer</summary>

The daemon inside WSL2 is not reachable. Most likely causes: (a) Docker Desktop is not running — launch it from the Start menu; (b) the WSL2 kernel was updated and needs the WSL runtime restarted — run `wsl --shutdown` in PowerShell then relaunch Docker Desktop; (c) in rare cases, run `wsl --update` to install the latest WSL kernel package.

</details>

**Question 5.** You want scripts to work whether the user has Docker Desktop or a remote Docker daemon. Which single field in `docker info --format json` tells you what OS the daemon is running on?

<details><summary>Click to reveal answer</summary>

The **`OSType`** field. It will be `linux` for Linux daemons (including Docker Desktop's WSL2 daemon) and `windows` for a Windows-container daemon. You would use it like `docker info --format '{{.OSType}}'` and branch your script accordingly.

</details>
