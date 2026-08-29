##### What Is Virtualization?

**Virtualization** is a technique that lets one physical computer pretend to be many separate computers. It does this using a piece of software called a **hypervisor** (examples: VMware ESXi, Microsoft Hyper-V, Oracle VirtualBox, KVM).
hypervisor sits between the real hardware and the pretend ones, deciding who gets which slice of CPU, RAM, and disk. Each pretend computer is a Virtual Machine (VM), and it contains a complete operating system — its own kernel, its own drivers, its own init system, its own system services — with your application sitting on top of all that.
Each VM contains a **complete operating system** — a full Linux or Windows install with its own kernel, its own drivers, its own init system, and every system service. On top of that OS, you install your application and its libraries. Booting a VM takes minutes because it is booting a full OS from a virtual BIOS upward, exactly like a physical computer.

Think of virtualization like a large apartment building where : 
  * The land, foundation of apartment building is your physical hardware: an i7-14700 processor with 20 cores / 28 threads, 32 GB of RAM, a 500 GB NVMe SSD.
  * The building manager is the hypervisor. They split the mains and enforce who gets what.
  * Each apartment is a VM.
  * Every apartment has its own furnace, its own water heater, its own electrical panel, and its own front door onto the street. Each apartment is fully independent, but that independence costs a lot of duplicated infrastructure.
  * The furniture and the people inside are your application and its libraries, but they arrive only after all that machinery is installed.

***What gets duplicated in the building ?***   
Walk into any appartment unit and you'll find the same machinery as the one next door: a furnace, a water heater, a breaker panel, a run of plumbing. Three units, three furnaces. Not a shared one in the basement — three separate installations, each bought, installed, and maintained on its own, each taking up a closet.

***What that maps to ?***   
Each of those private furnaces is a guest operating system. Every VM carries its own kernel, its own drivers, its own systemd, its own sshd and cron and journald. Three VMs, three complete Linux installs, running side by side and doing the same work as each other.

***What the duplication buys ?***  
The answer is real **Isolation**. Appartment Unit 2B can flood, catch fire, or lose power, and 3A doesn't notice — nothing is shared, so nothing propagates. Same with VMs: one can kernel-panic, get compromised, or run Windows while its neighbour runs Linux, and the others carry on untouched.

Now — the interesting question, and the one that splits virtualization into two types:  


***Who manages the building, and do they live in it?***

##### Type 1 — The building manager is the building

A Type 1 (bare-metal) hypervisor installs directly onto the hardware, the way an operating system normally would. There is no Windows underneath it, no Ubuntu desktop, no browser. ESXi, Hyper-V, and Xen are the examples. ESXi has its own minimal kernel (VMkernel) that talks to your NVMe and NIC drivers directly, and it does exactly one job: run VMs.

In the analogy: the building manager isn't a tenant. There's no manager's apartment, no manager's kitchen, no manager's TV. The building's infrastructure and the management office are the same thing — the boiler room is the front desk. Nothing is consumed by anyone who isn't a resident.

This is what `the hypervisor replaces the host OS` means, and it's the answer to the natural question where's the host OS in this diagram? — there isn't one. That's the point.

```
  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
  │      VM 1       │ │      VM 2       │ │      VM 3       │
  │  "web"          │ │  "db"           │ │  "test"         │
  │  4 vCPU         │ │  8 vCPU         │ │  4 vCPU         │
  │  8 GB RAM       │ │  12 GB RAM      │ │  4 GB RAM       │
  │  40 GB vDisk    │ │  80 GB vDisk    │ │  30 GB vDisk    │
  ├─────────────────┤ ├─────────────────┤ ├─────────────────┤
  │ app  nginx      │ │ app  postgres   │ │ app  pytest     │
  │ libs            │ │ libs            │ │ libs            │
  │ systemd         │ │ systemd         │ │ systemd         │
  │ ██KERNEL        │ │ ██KERNEL        │ │ ██KERNEL        │
  │ vFirmware       │ │ vFirmware       │ │ vFirmware       │
  │ vCPU vRAM vDisk │ │ vCPU vRAM vDisk │ │ vCPU vRAM vDisk │
  └────────┬────────┘ └────────┬────────┘ └────────┬────────┘
           └───────────────────┼───────────────────┘
  ┌────────────────────────────┴────────────────────────────┐
  │   HYPERVISOR   (ESXi / Hyper-V)                         │
  │   ─ it IS the kernel of this machine ─                  │
  │   overhead: ~1.5 GB RAM                                 │
  │   allocated: 16 vCPU / 28T · 24 GB / 32 GB · 150/500 GB │
  │   free:      6.5 GB RAM · 350 GB SSD                    │
  └────────────────────────────┬────────────────────────────┘
  ┌────────────────────────────┴────────────────────────────┐
  │   HOST BIOS / UEFI                                      │
  └────────────────────────────┬────────────────────────────┘
  ┌────────────────────────────┴────────────────────────────┐
  │   PHYSICAL HARDWARE   i7-14700 20C/28T · 32 GB · 500 GB │
  └─────────────────────────────────────────────────────────┘
```
RAM ledger :
```

Physical RAM installed                     32.0 GB
  − hypervisor (ESXi/VMkernel) overhead   − 1.5 GB
  ─────────────────────────────────────────────────
  = available to hand out to VMs           30.5 GB   ← "about 30 GB for tenants"

  − VM1 "web"                              − 8.0 GB
  − VM2 "db"                              − 12.0 GB
  − VM3 "test"                             − 4.0 GB
  ─────────────────────────────────────────────────
  = unallocated                             6.5 GB   ← "6.5 GB to spare
  
  ```
CPU ledger
```

Physical CPU  i7-14700
  8 P-cores × 2 threads (hyper-threaded)     16 threads
  12 E-cores × 1 thread (no HT)              12 threads
  ───────────────────────────────────────────────────────
  = total hardware threads                   28 threads
  − hypervisor (VMkernel) reserves          − 2 threads
  ───────────────────────────────────────────────────────
  = schedulable for VMs                      26 threads

  − VM1 "web"                                − 4 vCPU
  − VM2 "db"                                 − 8 vCPU
  − VM3 "test"                               − 4 vCPU
  ───────────────────────────────────────────────────────
  = assigned                                  16 vCPU
  = headroom                                  10 threads

  overcommit ratio  16 vCPU / 26 threads  =  0.62 : 1   ← undercommitted
```



**The trade-off:** you can't use this machine. No browser, no IDE, no Spotify. It's a server, managed remotely. That's why Type 1 runs data centres and not laptops.

#### Type 2 — the building manager is a tenant with a day job (VirtualBox on Windows)
A Type 2 (hosted) hypervisor is an ordinary application running on your normal operating system. VirtualBox, VMware Workstation, Parallels. You double-click an icon to start it. It needs a kernel driver (VBoxDrv.sys on Windows, vboxdrv on Linux) to reach the CPU's VT-x virtualization features, which is why it demands admin rights at install and breaks after a kernel upgrade until the module is rebuilt.

In the analogy: the manager rents unit 1A and runs the building in their spare time. They have their own kitchen, their own furniture, their own life taking up space — and critically, they're a tenant, so their apartment competes for the same water pressure as everyone else's. When the manager runs a bath, the tenants notice.

That's the correct mental picture for the layering, too. VirtualBox is a sibling of Chrome, not a parent of it. To your host kernel, VirtualBox.exe is just process #4471 asking for 20 GB. Your VM's entire existence — guest kernel included — lives inside the address space of one ordinary process, scheduled alongside your browser tabs.

##### ASCII Diagram — Container Architecture
```
  ┌─────────────────┐ ┌─────────────────┐      ┌─────────┐ ┌────────┐
  │      VM 1       │ │      VM 2       │      │ Chrome  │ │VS Code │
  │  "web"          │ │  "db"           │      │ ~3 GB   │ │ ~1 GB  │
  │  4 vCPU         │ │  8 vCPU         │      └────┬────┘ └───┬────┘
  │  8 GB RAM       │ │  12 GB RAM      │           │          │
  │  40 GB vDisk    │ │  80 GB vDisk    │           │          │
  ├─────────────────┤ ├─────────────────┤           │          │
  │ app  nginx      │ │ app  postgres   │           │          │
  │ libs            │ │ libs            │           │          │
  │ systemd         │ │ systemd         │           │          │
  │ ██KERNEL        │ │ ██KERNEL        │           │          │
  │ vFirmware       │ │ vFirmware       │           │          │
  │ vCPU vRAM vDisk │ │ vCPU vRAM vDisk │           │          │
  └────────┬────────┘ └────────┬────────┘           │          │
           └─────────┬─────────┘                    │          │
  ┌──────────────────┴──────────────────┐           │          │
  │  HYPERVISOR                         │           │          │
  │  VirtualBox / VMware Workstation    │           │          │
  │  (.exe + kernel driver)             │           │          │
  │  overhead: ~0.5 GB RAM              │           │          │
  │  hands out: 12 vCPU · 20 GB RAM     │           │          │
  └──────────────────┬──────────────────┘           │          │
                     └────────────┬─────────────────┴──────────┘
  ┌───────────────────────────────┴─────────────────────────────┐
  │  ★ HOST OS USERLAND ★   Explorer / GNOME, services         │
  ├─────────────────────────────────────────────────────────────┤
  │  ★ HOST KERNEL ★   Windows NT  or  Linux 6.x               │
  │  ┌───────────────────────────────────────────────────────┐  │
  │  │ + VBoxDrv.sys / vmx86  → VT-x                         │  │
  │  └───────────────────────────────────────────────────────┘  │
  │  host OS + apps: ~6 GB RAM                                  │
  │  ── RAM ledger ──   32 total                                │
  │     host OS+apps 6 · hypervisor 0.5 · VM1 8 · VM2 12        │
  │     = 26.5 used  ·  5.5 GB free  →  no room for VM3         │
  │  ── DISK ledger ──  500 total                               │
  │     host 120 · web.vdi 40 · db.vdi 80  = 240 used           │
  └───────────────────────────────┬─────────────────────────────┘
  ┌───────────────────────────────┴─────────────────────────────┐
  │  HOST BIOS / UEFI                                           │
  └───────────────────────────────┬─────────────────────────────┘
  ┌───────────────────────────────┴─────────────────────────────┐
  │  PHYSICAL HARDWARE   i7-14700 20C/28T · 32 GB · 500 GB NVMe │
  └─────────────────────────────────────────────────────────────┘


```

Read this ledger against the last one. 
```
Physical RAM installed                     32.0 GB
  − host OS (Windows) + Explorer,
    Defender, services                    − 2.0 GB
  − host apps: Chrome 3.0, VS Code 1.0    − 4.0 GB
  ─────────────────────────────────────────────────
  = left when you launch VirtualBox        26.0 GB
  − hypervisor process overhead           − 0.5 GB
  ─────────────────────────────────────────────────
  = available to hand out to VMs           25.5 GB   ← vs 30.5 GB on Type 1

  − VM1 "web"                              − 8.0 GB
  − VM2 "db"                              − 12.0 GB
  ─────────────────────────────────────────────────
  = unallocated                              5.5 GB
  − VM3 "test" wants                       − 4.0 GB
  ─────────────────────────────────────────────────
  = would leave                              1.5 GB   ← host thrashes; VM3 won't fit
```

Same hardware, same first two VMs, 5 GB less to give away. The host paid for itself before the hypervisor loaded.



Scheduler depth — how far a guest thread is from silicon
```
TYPE 1                              TYPE 2
──────────────────────────────      ──────────────────────────────
nginx wants CPU                     nginx wants CPU
  ↓ guest kernel scheduler            ↓ guest kernel scheduler
  ↓ hypervisor scheduler              ↓ hypervisor scheduler
  ↓                                   ↓ HOST KERNEL scheduler  ★
    real core                             (competes with Chrome,
                                           VS Code, Spotify)
                                        ↓
2 hops                                    real core

                                    3 hops

```

Disk I/O path — one guest write reaching the SSD


```
TYPE 1                              TYPE 2
──────────────────────────────      ──────────────────────────────
guest ext4                          guest ext4
  ↓ virtual disk controller           ↓ virtual disk controller
  ↓ VMkernel → .vmdk on datastore     ↓ VirtualBox process       ★
  ↓                                   ↓ host NTFS → web.vdi file ★
    NVMe                              ↓
                                        NVMe
1 file system

                                    2 file systems stacked
```




***Memory ledger — side by side comparison for Type1 and Type2***

```
                                    TYPE 1        TYPE 2
  ──────────────────────────────────────────────────────
  Physical RAM installed             32.0          32.0
  − host OS + host apps            −  0.0       −  6.0
  − hypervisor overhead            −  1.5       −  0.5
  ──────────────────────────────────────────────────────
  = allocatable to VMs               30.5          25.5

  − VM1 "web"                      −  8.0       −  8.0
  − VM2 "db"                       − 12.0       − 12.0
  − VM3 "test"                     −  4.0          n/a
  ──────────────────────────────────────────────────────
  = unallocated                       6.5           5.5
    VMs running                       3             2   ← VM3 needs 4.0,
                                                          would leave 1.5
```
``Of the allocated RAM, roughly 1 GB per VM is guest kernel + systemd — 3 GB (Type 1) or 2 GB (Type 2) spent running operating systems, not applications.
``



### What Is Containerization?

***Containerization*** is a lighter-weight isolation technique. Instead of virtualizing an entire computer, containers share the **host operating system's kernel** and virtualize only the parts *above* the kernel — the filesystem, the network stack, the process list, and the user list. This is done using Linux kernel features called **namespaces** (which isolate what a process can *see*) and **cgroups** (which limit what a process can *use*, like CPU and memory).

Because containers share the host kernel, they start in milliseconds, use a fraction of the RAM of a VM, and have almost zero performance overhead. But because they share the kernel, a Linux container needs a Linux kernel — you cannot run a Windows container on a Linux kernel or vice versa. (On Windows, Docker Desktop uses a lightweight Linux VM called WSL2 to provide the Linux kernel for Linux containers.)

Think of containerization like a single-family house divided into apartments that share the main furnace, water heater, and electrical panel of the house. Each apartment has its own private rooms, its own front door, and its own kitchen — but the underlying infrastructure is shared, saving enormous amounts of duplicated equipment.

##### ASCII Diagram — Virtual Machine Architecture

```

┌───────────────────────────────────────────────────────────────┐
│  VM 1                    VM 2                    VM 3         │
│ ┌──────────┐            ┌──────────┐            ┌──────────┐  │
│ │  App A   │            │  App B   │            │  App C   │  │
│ ├──────────┤            ├──────────┤            ├──────────┤  │
│ │Libraries │            │Libraries │            │Libraries │  │
│ ├──────────┤            ├──────────┤            ├──────────┤  │
│ │  Guest   │            │  Guest   │            │  Guest   │  │
│ │    OS    │            │    OS    │            │    OS    │  │
│ │ (Ubuntu) │            │ (CentOS) │            │(Windows) │  │
│ │ ~2 GB    │            │ ~2 GB    │            │ ~10 GB   │  │
│ └──────────┘            └──────────┘            └──────────┘  │
├───────────────────────────────────────────────────────────────┤
│                    HYPERVISOR (Type 1 or 2)                   │
│                (VMware, Hyper-V, VirtualBox, KVM)             │
├───────────────────────────────────────────────────────────────┤
│                     HOST OPERATING SYSTEM                     │
│                    (only for Type 2 hypervisor)               │
├───────────────────────────────────────────────────────────────┤
│           PHYSICAL HARDWARE (CPU, RAM, Disk, NIC)             │
└───────────────────────────────────────────────────────────────┘
```

##### ASCII Diagram — Container Architecture

```
<!-- CODE -->
┌───────────────────────────────────────────────────────────────┐
│ Container 1            Container 2            Container 3     │
│ ┌──────────┐          ┌──────────┐           ┌──────────┐     │
│ │  App A   │          │  App B   │           │  App C   │     │
│ ├──────────┤          ├──────────┤           ├──────────┤     │
│ │Libraries │          │Libraries │           │Libraries │     │
│ │ ~50 MB   │          │ ~100 MB  │           │ ~30 MB   │     │
│ └──────────┘          └──────────┘           └──────────┘     │
├───────────────────────────────────────────────────────────────┤
│                DOCKER ENGINE (containerd + runc)              │
├───────────────────────────────────────────────────────────────┤
│                    HOST OPERATING SYSTEM                      │
│                    (Linux kernel — SHARED)                    │
├───────────────────────────────────────────────────────────────┤
│           PHYSICAL HARDWARE (CPU, RAM, Disk, NIC)             │
└───────────────────────────────────────────────────────────────┘
```

Notice how the container stack is dramatically thinner. No guest OS per container, no hypervisor. That is where all the speed and efficiency come from.