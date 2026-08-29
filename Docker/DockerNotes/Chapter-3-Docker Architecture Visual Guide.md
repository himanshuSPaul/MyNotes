# Docker Architecture Visual Reference Guide

## Introduction

This guide is for **future me**, three months from now, when I have written
a hundred Dockerfiles and forgotten *why* containers work the way they do.
It captures the mental model of Docker: what a container really is, how it
differs from a virtual machine, and how the pieces of Docker fit together.

The core problem Docker solves is environmental drift — the fact that
software behaves differently on different machines because of hidden
differences in OS libraries, runtime versions, and configuration.
Docker packages an application together with its entire user-space
environment into a portable unit called a container, so "works on my
machine" becomes "works on every machine".

## 1. Virtual Machine Architecture

```
┌───────────────────────────────────────────────────────┐
│  VM 1 (Ubuntu)      VM 2 (CentOS)     VM 3 (Windows)  │
│ ┌──────────┐       ┌──────────┐      ┌──────────┐     │
│ │  App     │       │   App    │      │   App    │     │
│ │ Libs     │       │   Libs   │      │   Libs   │     │
│ │ Guest OS │       │ Guest OS │      │ Guest OS │     │
│ └──────────┘       └──────────┘      └──────────┘     │
├───────────────────────────────────────────────────────┤
│                     HYPERVISOR                        │
├───────────────────────────────────────────────────────┤
│                     HOST OS                           │
├───────────────────────────────────────────────────────┤
│           HARDWARE (CPU, RAM, Disk, NIC)              │
└───────────────────────────────────────────────────────┘
```

## 2. Container Architecture

```
┌───────────────────────────────────────────────────────┐
│  Container 1        Container 2       Container 3     │
│ ┌──────────┐       ┌──────────┐      ┌──────────┐     │
│ │  App     │       │   App    │      │   App    │     │
│ │ Libs     │       │   Libs   │      │   Libs   │     │
│ └──────────┘       └──────────┘      └──────────┘     │
├───────────────────────────────────────────────────────┤
│                  DOCKER ENGINE                        │
├───────────────────────────────────────────────────────┤
│               HOST OS + LINUX KERNEL                  │
├───────────────────────────────────────────────────────┤
│           HARDWARE (CPU, RAM, Disk, NIC)              │
└───────────────────────────────────────────────────────┘
```

## 3. Docker Client-Daemon-Registry Flow

```
   [ YOU TYPE:    docker run nginx ]
              │
              ▼
   ┌────────────────────┐
   │   Docker Client    │
   └────────┬───────────┘
            │ (1) REST API call over socket
            ▼
   ┌────────────────────┐          (2) pull image
   │   Docker Daemon    │  ───────────────────────►  ┌──────────────┐
   │     (dockerd)      │                            │  Docker Hub  │
   │                    │  ◄───────────────────────  │  (Registry)  │
   │                    │       (3) image layers     └──────────────┘
   │  ┌──────────────┐  │
   │  │ Local Image  │  │  (4) create container from image
   │  │  (nginx)     │  │
   │  └──────┬───────┘  │
   │         ▼          │
   │  ┌──────────────┐  │
   │  │  Container   │  │  (5) container runs
   │  │  (nginx)     │  │
   │  └──────────────┘  │
   └────────────────────┘
```

## 4. Comparison Table

| Feature            | Container         | Virtual Machine     |
|--------------------|-------------------|---------------------|
| Isolation          | Namespaces/cgroups| Hypervisor          |
| Startup time       | ms – seconds      | Seconds – minutes   |
| Size on disk       | 5 MB – few 100 MB | 1 GB – 20 GB        |
| RAM overhead       | ~0                | Hundreds of MB – GB |
| Portability        | Very high (OCI)   | High (per hypervisor)|
| Cross-OS           | Same kernel family only | Any guest on any host |
| Density per server | 100s – 1000s      | 10s                 |
| Typical use        | Microservices     | Full OS workloads   |

## 5. When to Use Containers vs VMs — Decision Matrix

| Scenario                                              | Choose     | Why                                       |
|-------------------------------------------------------|------------|-------------------------------------------|
| Deploy 200 stateless microservices                    | Container  | Density and fast rollouts                 |
| Run legacy Windows Server 2008 application            | VM         | Container needs matching kernel; legacy   |
| Sandbox untrusted user code with maximum isolation    | VM         | Hypervisor is a smaller attack surface    |
| Ephemeral CI/CD build agents that spin up per commit  | Container  | Startup speed and image reproducibility   |
| Multi-tenant SaaS: strong customer isolation          | VM         | Kernel exploit doesn't cross the boundary |
| Local dev environment mirroring production            | Container  | Same image dev → CI → prod                |

## 6. Glossary

- **Container** — an isolated running instance of an image.
- **Image** — a read-only template used to create containers.
- **Dockerfile** — a text file with instructions to build an image.
- **Registry** — a server that stores and distributes images.
- **Docker Hub** — the default public registry.
- **Docker Engine** — the daemon + CLI on a Linux machine.
- **Docker Desktop** — Windows/Mac installer bundling Engine + a Linux VM.
- **dockerd** — the Docker daemon (background service).
- **containerd** — the OCI-compliant runtime Docker uses under the hood.
- **runc** — the low-level tool that spawns a container from an OCI bundle.
- **Namespace** — kernel feature isolating what a process can see.
- **cgroup** — kernel feature limiting what a process can use.
- **Layer** — one immutable slice of an image built by one Dockerfile step.
- **OCI** — Open Container Initiative, industry standard for images/runtimes.
- **Hypervisor** — software that runs virtual machines.
- **Virtual Machine (VM)** — a full virtualized computer with its own kernel.

## 7. History Timeline

- **1979** — Unix `chroot` — first filesystem isolation.
- **2000** — FreeBSD Jails — early multi-facet OS-level isolation.
- **2008** — cgroups merged into Linux kernel; LXC launches.
- **2013** — Docker publicly launched at PyCon.
- **2015** — OCI founded; Docker donates image spec and runc.
- **2015** — Kubernetes 1.0 released.
- **2017** — Docker donates containerd to CNCF.
- **2020+** — Kubernetes standardizes on containerd; ecosystem matures.
````

</details>

##### ✅ Verification Checklist

- [ ] Document is saved as `docker-architecture-guide.md` in your `docker-mastery` folder.
- [ ] Introduction paragraph explains, in your own words, the problem containers solve.
- [ ] VM architecture ASCII diagram is present with at least five labeled layers.
- [ ] Container architecture ASCII diagram is present with at least four labeled layers.
- [ ] Docker client-daemon-registry diagram shows five components with labeled arrows.
- [ ] Comparison table has at least eight rows.
- [ ] Decision matrix has at least six distinct scenarios with recommendations.
- [ ] Glossary has at least 15 terms — all listed in the requirements are included.
- [ ] History timeline has at least six milestones.
- [ ] You can explain every diagram out loud to an imaginary junior developer without looking at the notes.

##### 🌟 Bonus Challenges

1. **Compare three container runtimes.** Extend the guide with a short section comparing Docker, Podman, and containerd — what each is, how they relate, and when someone might choose one over another.
2. **Add a "Docker on Windows" architecture diagram.** Show explicitly how Docker Desktop uses WSL2 as the Linux kernel provider. Label the Windows host, WSL2 VM, dockerd inside WSL2, and containers inside dockerd.
3. **Write a 200-word "elevator pitch"** — imagine explaining containers to your non-technical grandparent in about 60 seconds. Add it to the top of the guide as a "What I would tell a non-engineer" callout.

<a id="m1-scenario"></a>
#### Scenario (Real-World Use Case)

You are a new engineer at a mid-sized fintech company. On day one, your manager tells you: "Our monolithic Java app runs on 40 VMs in AWS. We spend around $18,000 a month on those VMs. Each VM sits at 8% CPU utilization on average — most of the RAM and CPU is wasted on the guest OS. Leadership wants to know if we can cut that bill in half by migrating to containers."

Your job on day one is not to actually migrate anything — it is to explain, in one clear meeting to non-engineering executives, *why* containers would even help. You draw a whiteboard diagram of VM architecture next to container architecture, walk through the shared-kernel concept, show the density comparison table, and highlight that the same physical hardware could host ~10x more containers than VMs because containers do not carry a full guest OS each.

The executives green-light a proof-of-concept. Because you understood the fundamentals of Milestone 1, you now know what problem you are actually solving in the next 19 milestones: turning that monolith into portable, dense, fast-starting containers that can be orchestrated across a much smaller fleet of hosts.

This exact scenario — "why containers?" — is the first conversation of every containerization project in the industry. If you can have it clearly, you are already ahead of most junior engineers.

<a id="m1-quiz"></a>
#### Checkpoint Quiz

**Question 1.** Which of the following does a container share with its host?

A. Its own dedicated CPU  
B. The host's Linux kernel  
C. Its own hypervisor  
D. Its own BIOS

<details><summary>Click to reveal answer</summary>

**B. The host's Linux kernel.** Containers share the host kernel, which is exactly why they are so lightweight and why Linux containers cannot natively run on a Windows kernel.

</details>

**Question 2.** True or false: A Docker image is a running instance of a container.

<details><summary>Click to reveal answer</summary>

**False.** The relationship is the reverse: a Docker **container** is a running instance of an **image**. An image is a read-only template; a container is a live process created from that template.

</details>

**Question 3.** Which two Linux kernel features are the foundation of container isolation?

<details><summary>Click to reveal answer</summary>

**Namespaces** (which isolate what a process can *see* — e.g., its own PID list, its own network stack, its own filesystem view) and **cgroups** (which isolate what a process can *use* — e.g., how much CPU, memory, or I/O it can consume).

</details>

**Question 4.** What is the difference between Docker Engine and Docker Desktop?

<details><summary>Click to reveal answer</summary>

**Docker Engine** is the core daemon + CLI that actually runs containers on a Linux machine. **Docker Desktop** is a Windows/Mac desktop application that bundles Docker Engine together with a lightweight Linux VM (via WSL2 on Windows or a hypervisor on Mac), a graphical UI, Kubernetes, and Docker Compose. On Linux, you install Docker Engine directly; on Windows and Mac, you install Docker Desktop which contains Docker Engine.

</details>

**Question 5.** Why does the Open Container Initiative (OCI) matter to a developer who only ever uses Docker?

<details><summary>Click to reveal answer</summary>

Because OCI defines the standard image and runtime formats, an image built with Docker can be run by Podman, containerd, CRI-O, or any Kubernetes cluster — with no changes. This protects the developer from vendor lock-in: the skills, images, and Dockerfiles transfer to any OCI-compliant tool. Even if Docker (the company) disappeared tomorrow, your images would still run everywhere.

</details>

[⬆ Back to Table of Contents](#table-of-contents)

---

