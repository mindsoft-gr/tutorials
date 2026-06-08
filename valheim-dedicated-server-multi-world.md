# Valheim Dedicated Server (Multi-World: Native + Docker)

A complete guide to running **multiple Valheim worlds on one Linux server**, using either:

- Native Linux (systemd-based setup)
- Docker (recommended for scalability and isolation)

# Overview

With this buide you will be able to:

- Host multiple independent Valheim worlds
- Assign each world its own ports
- Run worlds via systemd or Docker or both
- Scale easily from 1 → 10+ worlds

---

# Table of Contents

- [1. Install Dependencies](#1-install-dependencies)
- [2. Install SteamCMD](#2-install-steamcmd)
- [3. Install Valheim Dedicated Server](#3-install-valheim-dedicated-server)
- [4. Fix Steam Runtime](#4-fix-steam-runtime)
- [5. Multi-World Directory Structure](#5-multi-world-directory-structure)
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

# 5. Multi-World Directory Structure

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

# 6. Create Worlds

(Recommended Method)

Create worlds in-game:
```bash
World  Seed
World1	Valheim
World2	tyeDWcN7cp
World3	ym6RwFNELS
```

Copy:

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

# 7. Native Startup Scripts

World 1
```bash
nano ~/valheim-server/start_world1.sh
```

Paste this:
```bash
#!/bin/bash

export templatedir=$PWD

./valheim_server.x86_64 \
-name "World1" \
-port 2456 \
-world "World1" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-configs/world1" \
-crossplay
```

Make it executable
```bash
chmod +x ~/valheim-server/start_world1.sh
```

World 2
```bash
nano ~/valheim-server/start_world2.sh
```

Paste this:
```bash
#!/bin/bash

./valheim_server.x86_64 \
-name "World2" \
-port 2459 \
-world "World2" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-configs/world2" \
-crossplay
```
Make it executable
```bash
chmod +x ~/valheim-server/start_world2.sh
```

World 3
```bash
nano ~/valheim-server/start_world3.sh
```

Paste this:
```bash
#!/bin/bash

./valheim_server.x86_64 \
-name "World3" \
-port 2462 \
-world "World3" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-configs/world3" \
-crossplay
```
Make it executable
```bash
chmod +x ~/valheim-server/start_world3.sh
```

# 8. Open Firewall Ports

```bash
sudo ufw allow 2456:2458/udp
sudo ufw allow 2459:2461/udp
sudo ufw allow 2462:2464/udp
```

# 9. Run Servers (Native)

```bash
./start_world1.sh
./start_world2.sh
./start_world3.sh
```

Connect:

```bash
server-IP:2456
server-IP:2459
server-IP:2462
```

# 10. systemd Services (Auto Start)

World 1 Service
```bash
sudo nano /etc/systemd/system/valheim-world1.service
```

```bash
[Unit]
Description=Valheim World1
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/home/user/valheim-server
ExecStart=/home/user/valheim-server/start_world1.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now valheim-world1
```

Logs:
```bash
journalctl -u valheim-world1 -f
```

World 2 Service
```bash
sudo nano /etc/systemd/system/valheim-world2.service
```

```bash
[Unit]
Description=Valheim World2
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/home/user/valheim-server
ExecStart=/home/user/valheim-server/start_world2.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now valheim-world1
```

Logs:
```bash
journalctl -u valheim-world1 -f
```

11. Add Admins
nano ~/.config/unity3d/IronGate/Valheim/adminlist.txt

Example:

76561198012345678
76561198087654321

Restart server after changes.

12. Docker Setup (Recommended)
Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

Log out & back in.

Directory
mkdir -p ~/valheim-docker/world1
mkdir -p ~/valheim-docker/world2
mkdir -p ~/valheim-docker/world3
13. docker-compose.yml
services:

  world1:
    image: ghcr.io/lloesche/valheim-server
    container_name: valheim-world1
    restart: unless-stopped

    ports:
      - "2456:2456/udp"
      - "2457:2457/udp"
      - "2458:2458/udp"

    environment:
      SERVER_NAME: World1
      WORLD_NAME: World1
      SERVER_PASS: StrongPassword1

    volumes:
      - ./world1:/config

  world2:
    image: ghcr.io/lloesche/valheim-server
    container_name: valheim-world2
    restart: unless-stopped

    ports:
      - "2459:2456/udp"
      - "2460:2457/udp"
      - "2461:2458/udp"

    environment:
      SERVER_NAME: World2
      WORLD_NAME: World2
      SERVER_PASS: StrongPassword2

    volumes:
      - ./world2:/config

  world3:
    image: ghcr.io/lloesche/valheim-server
    container_name: valheim-world3
    restart: unless-stopped

    ports:
      - "2462:2456/udp"
      - "2463:2457/udp"
      - "2464:2458/udp"

    environment:
      SERVER_NAME: World3
      WORLD_NAME: World3
      SERVER_PASS: StrongPassword3

    volumes:
      - ./world3:/config
14. Start Docker Servers
docker compose up -d

Logs:

docker logs -f valheim-world1

Stop:

docker compose down
