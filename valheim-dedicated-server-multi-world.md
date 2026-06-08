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
- [5. Create Multi-World Structure](#5-multi-world-directory-structure)
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
```

# 2. Install SteamCMD

```bash
mkdir -p ~/steamcmd
cd ~/steamcmd

wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
```

# 3. Install Valheim Dedicated Server

```bash
./steamcmd.sh
```

Inside SteamCMD:
```bash
force_install_dir /home/$USER/valheim-server
login anonymous
app_update 896660 validate
quit
```

# 4. Fix Steam Runtime
```bash
mkdir -p ~/.steam/sdk64
cp ~/steamcmd/linux64/steamclient.so ~/.steam/sdk64/
```

# 5. Create Multi-World Structure
```bash
mkdir -p ~/valheim-configs/world1
mkdir -p ~/valheim-configs/world2
mkdir -p ~/valheim-configs/world3
```
```bash
valheim-configs/
├── world1/
├── world2/
└── world3/
```

#7. Create Worlds (Recommended Method)

Create worlds in-game:
```bash
World	Seed
World1	42069lolxd
World2	vikings123
World3	hardcore999
```
Copy files:
```bash
WorldName.db
WorldName.fwl
```
Place into:
```bash
~/.config/unity3d/IronGate/Valheim/worlds_local/
```
Then move into:
```bash
~/valheim-configs/worldX/worlds_local/
```

#8. Native Startup Scripts

World 1
```bash
nano ~/valheim-server/start_world1.sh
```
```bash
#!/bin/bash

export templatedir=$PWD

./valheim_server.x86_64 \
-name "World1" \
-port 2456 \
-world "World1" \
-password "StrongPassword1" \
-public 1 \
-savedir "$HOME/valheim-configs/world1" \
-crossplay
```
```bash
chmod +x ~/valheim-server/start_world1.sh
```

World 2
nano ~/valheim-server/start_world2.sh
#!/bin/bash

./valheim_server.x86_64 \
-name "World2" \
-port 2459 \
-world "World2" \
-password "StrongPassword2" \
-public 1 \
-savedir "$HOME/valheim-configs/world2" \
-crossplay
World 3
nano ~/valheim-server/start_world3.sh
#!/bin/bash

./valheim_server.x86_64 \
-name "World3" \
-port 2462 \
-world "World3" \
-password "StrongPassword3" \
-public 1 \
-savedir "$HOME/valheim-configs/world3" \
-crossplay

# 9. Firewall Setup
sudo ufw allow 2456:2458/udp
sudo ufw allow 2459:2461/udp
sudo ufw allow 2462:2464/udp
sudo ufw enable

# 10. Run Servers (Manual)
./start_world1.sh
./start_world2.sh
./start_world3.sh

Connect:

IP:2456
IP:2459
IP:2462

# 11. systemd Auto Start
World 1 Service
sudo nano /etc/systemd/system/valheim-world1.service
[Unit]
Description=Valheim World1
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=/home/$USER/valheim-server
ExecStart=/home/$USER/valheim-server/start_world1.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

Enable:

sudo systemctl daemon-reload
sudo systemctl enable --now valheim-world1

Logs:

journalctl -u valheim-world1 -f
# 12. Add Admins
nano ~/.config/unity3d/IronGate/Valheim/adminlist.txt

Example:

76561198012345678
76561198087654321
