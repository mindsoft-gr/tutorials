# Valheim Dedicated Server (Multi-World: Native + Docker)

A complete guide to running **multiple Valheim worlds on one Linux server**, using either:

- Native Linux (systemd-based setup)
- Docker (recommended for scalability and isolation)

---

# Table of Contents

- [Overview](#-valheim-dedicated-server-multi-world-native--docker)
- [1. Install Dependencies](#1-install-dependencies)
- [2. Install SteamCMD](#2-install-steamcmd)
- [3. Install Valheim Dedicated Server](#3-install-valheim-dedicated-server)
- [4. Fix Steam Runtime](#4-fix-steam-runtime-library)
- [5. Multi-World Structure](#5-multi-world-directory-structure)
- [6. Create Worlds](#6-create-worlds-recommended-method)
- [7. Native Startup Scripts](#7-native-startup-scripts)
- [8. Firewall Setup](#8-open-firewall-ports)
- [9. Run Servers (Native)](#9-run-servers-native)
- [10. systemd Services](#10-systemd-services-auto-start)
- [11. Admin Setup](#11-add-admins)
- [12. Docker Setup](#12-docker-setup-recommended)
- [13. docker-compose](#13-docker-composeyml)
- [14. Start Docker](#14-start-docker-servers)
- [15. Production Notes](#15-recommended-production-setup)

---

# . Overview

With this buide you will be able to:

- Host multiple independent Valheim worlds
- Assign each world its own ports
- Run worlds via systemd or Docker or both
- Scale easily from 1 → 10+ worlds

---

# 1. Install Dependencies

Open a terminal and install the basics:

```bash
sudo apt update && sudo apt install curl wget tar lib32gcc-s1 -y

# 2. Install SteamCMD
