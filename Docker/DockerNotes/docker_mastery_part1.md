# Docker Mastery: From Zero to DevOps Hero

Welcome to **Docker Mastery: From Zero to DevOps Hero** — a single, comprehensive, self-paced learning resource that takes you from a completely empty Windows laptop with no Docker installed to a confident, production-ready DevOps practitioner. This document is designed for absolute beginners, curious developers, sysadmins pivoting to DevOps, and experienced engineers who want a single source of truth for Docker. Read it linearly if you are new, or jump to any milestone using the Table of Contents if you already know the basics. Every milestone is self-contained, so you can pause and resume at any point without losing context.

> 📖 **Navigation Tip:** If you're viewing this in VS Code, Obsidian, Typora, or GitHub, a clickable Table of Contents will appear in the left sidebar automatically. Use it to jump between milestones. If you're reading as a PDF, use the Table of Contents below or the PDF bookmark panel. For printing, page numbers will appear at the bottom-right of each page.

---

## Table of Contents

- [Docker Mastery: From Zero to DevOps Hero](#docker-mastery-from-zero-to-devops-hero)
  - [Table of Contents](#table-of-contents)
  - [Phase 1 — Foundation (🟢 Beginner)](#phase-1--foundation--beginner)
    - [Milestone 1: Understanding Containers \& Docker](#milestone-1-understanding-containers--docker)
  - [📍 **Milestone 1 of 20** | 🟢 **Beginner** | ⏱ **Estimated Time: 2–3 hours** | **Prerequisites:** None](#-milestone-1-of-20---beginner---estimated-time-23-hours--prerequisites-none)
      - [Theory](#theory)
    - [Milestone 2: Installing Docker on Windows](#milestone-2-installing-docker-on-windows)
  - [📍 **Milestone 2 of 20** | 🟢 **Beginner** | ⏱ **Estimated Time: 1–2 hours** | **Prerequisites:** Milestone 1](#-milestone-2-of-20---beginner---estimated-time-12-hours--prerequisites-milestone-1)
      - [Theory](#theory-1)
        - [The Windows-Specific Challenge](#the-windows-specific-challenge)
        - [System Requirements](#system-requirements)
        - [What Gets Installed](#what-gets-installed)
      - [Commands](#commands)
      - [Command Options/Flags](#command-optionsflags)
        - [`docker version` flags](#docker-version-flags)
        - [`docker info` flags](#docker-info-flags)
        - [`docker run` flags (introduced here; complete table in Milestone 3)](#docker-run-flags-introduced-here-complete-table-in-milestone-3)
        - [`wsl` flags (Windows PowerShell)](#wsl-flags-windows-powershell)
      - [Examples](#examples)
        - [Step 1 — Check your Windows version](#step-1--check-your-windows-version)
        - [Step 2 — Verify CPU virtualization is enabled](#step-2--verify-cpu-virtualization-is-enabled)
        - [Step 3 — Install and enable WSL2](#step-3--install-and-enable-wsl2)
        - [Step 4 — Download and install Docker Desktop](#step-4--download-and-install-docker-desktop)
        - [Step 5 — Verify installation with `docker version`](#step-5--verify-installation-with-docker-version)
        - [Step 6 — Verify installation with `docker info`](#step-6--verify-installation-with-docker-info)
        - [Step 7 — Run your first container](#step-7--run-your-first-container)
        - [Configuring Docker Desktop Settings](#configuring-docker-desktop-settings)
        - [Troubleshooting](#troubleshooting)
      - [Command Exercises](#command-exercises)
        - [Exercise 2.1 — `docker version` basic](#exercise-21--docker-version-basic)
        - [Exercise 2.2 — `docker version` with format flag](#exercise-22--docker-version-with-format-flag)
        - [Exercise 2.3 — `docker version` troubleshooting](#exercise-23--docker-version-troubleshooting)
        - [Exercise 2.4 — `docker info` basic](#exercise-24--docker-info-basic)
        - [Exercise 2.5 — `docker info` with JSON format](#exercise-25--docker-info-with-json-format)
        - [Exercise 2.6 — `docker info` filtered output](#exercise-26--docker-info-filtered-output)
        - [Exercise 2.7 — `docker system info` basic](#exercise-27--docker-system-info-basic)
        - [Exercise 2.8 — Explore Docker Desktop GUI](#exercise-28--explore-docker-desktop-gui)
      - [Hands-On Assignment](#hands-on-assignment)
      - [Mini-Project](#mini-project)
        - [🎯 Project Title](#-project-title)
        - [🎯 Objective](#-objective)
        - [📋 Requirements](#-requirements)
        - [🪜 Step-by-Step Guidance](#-step-by-step-guidance)
        - [📦 Complete Mini-Project Solution](#-complete-mini-project-solution)
        - [✅ Verification Checklist](#-verification-checklist)
        - [🌟 Bonus Challenges](#-bonus-challenges)
      - [Scenario (Real-World Use Case)](#scenario-real-world-use-case)
      - [Checkpoint Quiz](#checkpoint-quiz)

---


## Phase 1 — Foundation (🟢 Beginner)

This phase builds your mental model of what containers are, gets Docker running on your Windows machine, and teaches you the fundamental CLI commands you will use every single day for the rest of your career. Do not rush this phase — the analogies and terminology you learn here will make every advanced topic later feel intuitive.

---

### Milestone 1: Understanding Containers & Docker

---
📍 **Milestone 1 of 20** | 🟢 **Beginner** | ⏱ **Estimated Time: 2–3 hours** | **Prerequisites:** None
---

<a id="m1-theory"></a>
#### Theory

Before you type a single Docker command, you need a rock-solid mental model of what a container actually is, how it differs from a virtual machine, and what problem Docker was invented to solve. This section is intentionally theory-heavy — the hands-on commands come in Milestone 2, but if you skip this section you will be memorizing commands without understanding them.



