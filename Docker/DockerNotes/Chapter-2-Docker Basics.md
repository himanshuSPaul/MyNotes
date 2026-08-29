


##### What Is Docker?

**Docker** is a *platform* — a collection of tools — that makes it easy to build, ship, and run containers. Before Docker existed, Linux had all the kernel features needed for containerization (namespaces since 2002, cgroups since 2008), but using them required deep expertise. Docker packaged those features behind a friendly command-line interface, a standard file format for describing containers (the **Dockerfile**), and a public sharing site (**Docker Hub**). That combination is what turned containers from a niche Linux feature into a global industry standard.

Docker as a *company* was founded in 2013. The word "Docker" today may refer to the company, the open-source engine, the CLI tool, or the ecosystem — context tells you which.

##### What Is the Docker Engine?

**Docker Engine** is the core runtime — the actual software that creates and runs containers on a Linux machine. It has three main pieces:

1. **dockerd** — the Docker daemon. A long-running background process (a "service") that listens for API requests and does the heavy lifting: pulling images, creating containers, managing networks, etc.
2. **Docker CLI (`docker`)** — the command-line tool you type into a terminal. When you run `docker run nginx`, the CLI sends an HTTP request to dockerd, which then does the work.
3. **REST API** — the protocol the CLI uses to talk to the daemon. This same API can be called by any tool, which is how graphical Docker managers, IDE plugins, and CI/CD systems talk to Docker.

##### What Is Docker Desktop?

**Docker Desktop** is a *packaged application* for Windows and macOS that bundles Docker Engine, the Docker CLI, Docker Compose, Kubernetes, a graphical management UI, and — critically on Windows — a lightweight Linux VM (using WSL2) that provides the Linux kernel needed to run Linux containers. On Linux, you do not need Docker Desktop because your kernel already is Linux; you just install Docker Engine directly.

So the relationship is:

- **Docker Desktop** (the installer you download on Windows) → contains →
- **Docker Engine** (the daemon + CLI) → which uses →
- **containerd** (the low-level container manager) → which uses →
- **runc** (the tiny program that actually calls the Linux kernel to spawn a container)

##### Docker Architecture in Detail

Docker follows a **client-server** architecture. Here is every piece and how they interact.

**The Docker Client** is the `docker` command you type. It is dumb — it just formats your command into an HTTP request and sends it to the daemon. The client can talk to a daemon on the same machine (via a Unix socket at `/var/run/docker.sock` on Linux, or a named pipe on Windows) or to a daemon on a remote machine over the network.

**The Docker Daemon (dockerd)** is the server. It manages images, containers, networks, and volumes. It pulls images from registries when asked. It creates containers by asking containerd to create them, which asks runc to create them, which asks the Linux kernel.

**A Docker Registry** is a server that stores Docker images. The default public registry is **Docker Hub** (`hub.docker.com`). Companies also run private registries: AWS ECR, Google Artifact Registry, Azure Container Registry, GitHub Container Registry, or self-hosted registries using the open-source `registry` image.

**A Docker Image** is a read-only template used to create containers. Think of it as a snapshot of a filesystem plus some metadata (which command to run, which ports are exposed, etc.). Images are built in **layers** — each instruction in a Dockerfile creates one layer, and layers are cached and shared between images to save disk space and download time.

**A Docker Container** is a running (or stopped) instance of an image. It has its own filesystem (a thin writable layer on top of the read-only image layers), its own network interface, its own process list. You can create many containers from the same image, the same way you can create many objects from the same class in programming.

##### Diagram — Docker Client-Daemon-Registry Flow

```

    YOUR MACHINE                          INTERNET
┌──────────────────────┐             ┌──────────────────┐
│  Docker Client       │             │                  │
│  (you type `docker`) │             │  Docker Hub      │
└─────────┬────────────┘             │  (Registry)      │
          │ REST API                 │                  │
          │ over socket              │  ┌────────────┐  │
          ▼                          │  │ nginx img  │  │
┌──────────────────────┐   pull      │  │ redis img  │  │
│  Docker Daemon       │◄───────────►│  │ node  img  │  │
│  (dockerd)           │   push      │  └────────────┘  │
│                      │             └──────────────────┘
│  ┌────────────────┐  │
│  │ Local Images   │  │  ← cached copies of pulled images
│  └────────────────┘  │
│  ┌────────────────┐  │
│  │ Containers     │  │  ← running / stopped instances
│  └────────────────┘  │
│  ┌────────────────┐  │
│  │ Networks       │  │  ← virtual networks between containers
│  └────────────────┘  │
│  ┌────────────────┐  │
│  │ Volumes        │  │  ← persistent storage
│  └────────────────┘  │
└──────────────────────┘
```

##### Docker vs Virtual Machines — Detailed Comparison

| Feature | Docker Container | Virtual Machine |
|---|---|---|
| **Isolation level** | Process-level (kernel namespaces + cgroups) | Hardware-level (hypervisor virtualizes CPU/RAM/disk) |
| **Operating system** | Shares host kernel; only user-space is packaged | Full guest OS including its own kernel |
| **Startup time** | Milliseconds to a few seconds | Seconds to minutes |
| **Size on disk** | Typically 5 MB to a few hundred MB | Typically 1 GB to 20 GB per VM |
| **RAM overhead** | Near zero — only what the app uses | Hundreds of MB to GB just for the guest OS |
| **CPU overhead** | Near zero — runs as native processes | Small but measurable virtualization overhead |
| **Density (per host)** | Hundreds to thousands per server | Dozens per server |
| **Portability** | Extremely portable — any host with the same OS family and CPU architecture | Portable across hypervisors of the same family |
| **Cross-OS support** | Linux containers need Linux kernel; Windows containers need Windows kernel | Any OS can run on any hypervisor (Linux VMs on Windows host, etc.) |
| **Security isolation** | Strong but shares kernel — a kernel exploit breaks isolation | Very strong — hypervisor is a much smaller attack surface |
| **Networking** | Virtual bridge networks created by Docker | Virtual switches created by hypervisor |
| **Snapshot/backup** | Image commits, volume backups | Full VM snapshots (larger but more complete) |
| **Typical use** | Application deployment, microservices, CI/CD | Full OS workloads, legacy apps, security sandboxes |
| **Boot process** | No boot — a single process starts | Full BIOS → bootloader → kernel → init → services |
| **Update model** | Rebuild image and redeploy container | Patch the guest OS in place, or rebuild image |
| **State** | Ephemeral by default; state must be externalized to volumes | Persistent by default; state lives inside the VM |

##### The Open Container Initiative (OCI)

The **Open Container Initiative (OCI)** is a Linux Foundation project founded in 2015 to prevent container fragmentation. It defines two open standards:

1. **OCI Image Specification** — how a container image is packaged, its manifest format, its layer format. Any OCI-compliant tool can read any OCI-compliant image.
2. **OCI Runtime Specification** — how a container is launched from an image, including the JSON config that describes namespaces, cgroups, mount points, etc.

Docker donated its image format and its low-level runtime **runc** to become the reference implementations of these specs. This is critically important because it means images you build with Docker can be run by Podman, containerd, CRI-O, Kubernetes, and any other OCI-compliant runtime — you are not locked into Docker forever. The industry agrees on the format, competes on the tools.

##### Brief History of Docker and Containerization

- **1979** — Unix `chroot` introduced. First filesystem isolation primitive.
- **2000** — FreeBSD Jails. First serious multi-facet OS-level virtualization.
- **2002** — Linux namespaces introduced (mount namespace).
- **2005** — Solaris Zones and OpenVZ mature.
- **2008** — cgroups merged into Linux kernel. LXC (Linux Containers) project launches, combining namespaces + cgroups into a usable containerization system.
- **2013** — Solomon Hykes demos Docker at PyCon (originally an internal project at dotCloud, a PaaS company). Docker wraps LXC in a friendly CLI and adds the Dockerfile format and Docker Hub.
- **2014** — Docker replaces LXC with its own `libcontainer` runtime. Google, Red Hat, and others start contributing.
- **2015** — Open Container Initiative (OCI) founded. Docker donates image spec and runc.
- **2015** — Kubernetes 1.0 released by Google. Container orchestration takes off.
- **2016** — Docker Swarm mode integrated into Docker Engine. Docker for Windows and Docker for Mac released.
- **2017** — Docker donates containerd to CNCF. Container ecosystem splits into "low-level" (containerd, runc, CRI-O) and "high-level" (Docker, Podman, Kubernetes) layers.
- **2020** — Kubernetes announces it will deprecate the Docker shim in favor of any CRI-compliant runtime (usually containerd directly). Images and containers remain fully compatible.
- **2021–present** — Docker Desktop becomes a paid product for large enterprises, spurring alternatives like Rancher Desktop, Podman Desktop, and OrbStack. The container ecosystem continues to mature around OCI standards.

##### Key Terminology Recap

Before moving on, make sure you can define each of these in your own words:

- **Container** — an isolated, running instance of an image.
- **Image** — a read-only template used to create containers.
- **Dockerfile** — a text file with instructions to build an image.
- **Registry** — a server that stores images (e.g., Docker Hub).
- **Docker Engine** — the daemon + CLI on a Linux machine.
- **Docker Desktop** — the installer for Windows/macOS that bundles the engine plus a Linux VM.
- **dockerd** — the Docker daemon process.
- **containerd** — the industry-standard container runtime that Docker uses under the hood.
- **runc** — the low-level tool that actually creates a container from an OCI runtime bundle.
- **Namespace** — Linux kernel feature that isolates what a process can see.
- **cgroup** — Linux kernel feature that limits what a process can use.
- **Layer** — one immutable slice of an image, produced by one Dockerfile instruction.
- **OCI** — the Open Container Initiative, which standardizes image and runtime formats.
- **Hypervisor** — software that runs virtual machines.
- **Virtual Machine (VM)** — a fully virtualized computer with its own guest OS.

<a id="m1-commands"></a>
#### Commands

Milestone 1 is theory-only. You will not run commands until Docker is installed in Milestone 2. However, to prepare you for Milestone 2, here are the *conceptual* commands you will meet next — no need to memorize, just recognize:

- `docker version` — shows client and server versions.
- `docker info` — shows detailed daemon state (containers, images, storage driver, etc.).
- `docker run hello-world` — the traditional "am I working" test.

<a id="m1-flags"></a>
#### Command Options/Flags

Because no commands are executed in this milestone, there are no flags to detail here. Milestone 2 will introduce the first real flag tables.

<a id="m1-examples"></a>
#### Examples

The following are **conceptual examples** — do not run them yet. They illustrate what Docker commands look like so the syntax is not a surprise in Milestone 2.

Example 1 — Checking whether Docker is installed:

<!-- COMMAND -->
```powershell
docker version
```

Example 2 — Running your first container:

<!-- COMMAND -->
```powershell
docker run hello-world
```

Example 3 — Listing running containers:

<!-- COMMAND -->
```powershell
docker ps
```

<a id="m1-exercises"></a>
#### Command Exercises

Because this milestone is theory-only, these exercises test *conceptual understanding* rather than command execution. Answer each in your head or on paper, then reveal the solution.

##### Exercise 1.1 — Matching: Component to Role

Match each Docker component (left) to its role (right):

| Component | Role |
|---|---|
| A. dockerd | 1. A snapshot filesystem used to create containers |
| B. Docker CLI | 2. A public server that stores images |
| C. Docker Hub | 3. A background service that manages containers |
| D. Image | 4. A running instance of an image |
| E. Container | 5. The command-line tool you type into |

<summary>💡 Solution</summary>

<!-- OUTPUT -->
```
A → 3  (dockerd is the background daemon)
B → 5  (Docker CLI is the command-line tool)
C → 2  (Docker Hub is the public registry)
D → 1  (An Image is the read-only template / snapshot)
E → 4  (A Container is a running instance of an image)
```



##### Exercise 1.2 — Predict the Output: VM vs Container

You are told that starting a virtual machine takes 60 seconds and uses 2 GB of RAM just for its guest OS. Starting an equivalent container of nginx takes about 0.5 seconds and uses about 10 MB of RAM. If a physical server has 64 GB of RAM, roughly how many idle nginx VMs versus how many idle nginx containers could you run on it (ignoring the host OS overhead)?

<summary>💡 Solution</summary>

<!-- OUTPUT -->
```
VMs   : 64 GB / 2 GB   =  ~32 VMs
Containers: 64 GB / 10 MB = ~6,400 containers

This ~200x density difference is exactly why the industry
moved from VMs to containers for stateless workloads.
```



##### Exercise 1.3 — Troubleshooting Thought Experiment

A junior developer says, "I built a Docker image for my Windows desktop application on my Linux server, but when I try to run it on a Windows machine with Docker Desktop, it fails." Based on what you learned about the shared kernel model, explain in one sentence why this fails, and then in one sentence what needs to change.

<summary>💡 Solution</summary>

<!-- OUTPUT -->
```
WHY IT FAILS:
Linux containers share the host's Linux kernel, so a Linux-based
image cannot execute Windows-native code — and vice versa.

WHAT NEEDS TO CHANGE:
Either package the app as a Windows container (built on a Windows
base image, run on Windows Server or Docker Desktop in Windows
container mode), OR rewrite the app as a Linux-compatible program
so it can run in a Linux container.
```



<a id="m1-handson"></a>
#### Hands-On Assignment

**Task:** Produce a one-page "cheat sheet" (in Markdown or on paper) that answers the following five questions in your own words. Do not copy from this document — restating in your own words is what cements understanding.

1. What problem does Docker solve, in one sentence?
2. Explain the difference between a **Virtual Machine** and a **Container** using an analogy that is *not* apartments/houses (invent your own).
3. Draw (ASCII or on paper) the flow from typing `docker run nginx` on your keyboard to nginx actually running. Label at least: Client, Daemon, Registry, Image, Container.
4. Define these five terms in one sentence each: **Image**, **Container**, **Registry**, **Dockerfile**, **Layer**.
5. Name three real-world scenarios where you would pick a Virtual Machine over a container.



<a id="m1-miniproject"></a>
#### Mini-Project

##### 🎯 Project Title
**Docker Architecture Visual Reference Guide**

##### 🎯 Objective

You will create a single Markdown document (`docker-architecture-guide.md`) that will serve as your personal reference for the rest of this course. Because Milestone 1 is pure theory, the deliverable is a document — but a *good* one, because you will look back at it constantly as you learn commands in later milestones and want a refresher on the mental model.

##### 📋 Requirements

Your Markdown document must contain, in this order:

1. **A title and one-paragraph introduction** stating who the guide is for (you, in three months, when you have forgotten some of this) and what problem containers solve.
2. **An ASCII diagram of Virtual Machine architecture** with at least five labeled layers (Hardware, Hypervisor, Guest OS, Libraries, App).
3. **An ASCII diagram of Container architecture** with at least four labeled layers (Hardware, Host OS/Kernel, Docker Engine, Container = App + Libraries).
4. **A labeled Docker architecture diagram** showing the flow: Docker Client → Docker Daemon → Registry (Docker Hub) → Local Images → Running Containers. Every arrow must be labeled with what flows through it (e.g., "REST API request", "pull image", "create container").
5. **A comparison table** with at least eight rows comparing VMs and Containers on: isolation, startup time, size, RAM overhead, portability, cross-OS support, density, and typical use.
6. **A decision matrix** — a table titled "When to Use Containers vs VMs" with at least six scenarios and a recommendation column ("Container", "VM", or "Either — explain").
7. **A glossary** of at least 15 terms with one-sentence definitions. Include at minimum: Container, Image, Dockerfile, Registry, Docker Hub, Docker Engine, Docker Desktop, dockerd, containerd, runc, namespace, cgroup, layer, OCI, hypervisor, virtual machine.
8. **A "history timeline"** listing at least six milestones in the evolution of containers, from chroot (1979) to today.

##### 🪜 Step-by-Step Guidance

1. Create a new folder called `docker-mastery` on your Desktop. Inside it, create a file called `docker-architecture-guide.md`.
2. Open the file in VS Code or any Markdown editor with preview support.
3. Write the introduction paragraph first — this forces you to articulate the "why" before the "what".
4. Sketch the two architecture diagrams on paper first, then convert them to ASCII using box-drawing characters (`┌ ┐ └ ┘ ─ │ ├ ┤`). Keep them simple.
5. Build the client-daemon-registry diagram next. Label every arrow.
6. Draft the comparison table. Try to write each cell without looking back at this milestone — check afterward.
7. For the decision matrix, brainstorm six *distinct* scenarios (e.g., "run a legacy Windows Server 2008 app", "run 500 microservices on one cluster", "sandbox untrusted user code with maximum isolation"). Assign each a recommendation and a one-sentence justification.
8. Write the glossary last, when the terms are freshest.
9. End with the timeline.
10. Save the file. Commit to reviewing it at the end of every subsequent milestone to reinforce the mental model.

##### 📦 Complete Mini-Project Solution

<details><summary>📦 Complete Mini-Project Solution</summary>

<!-- CODE -->
````markdown