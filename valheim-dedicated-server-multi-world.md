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

---

World 2
```bash
nano ~/valheim-server/start_world2.sh
```

Paste this:
```bash
#!/bin/bash

./valheim_server.x86_64 \
-name "World1" \
-port 2456 \
-world "World1" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-configs/world1" \
-savedir "$HOME/valheim-configs/world2" \
-crossplay
```
Make it executable
```bash
chmod +x ~/valheim-server/start_world2.sh
```

---

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

Launch parameters for World Modifiers

Edit your server startup script for example ```start_world1.sh``` and add world modifier flags.

Difficulty presets:
```bash
-preset normal
-preset casual
-preset easy
-preset hard
-preset hardcore
-preset immersive
-preset hammer
```

```-preset``` overrides most other settings, so use it first when resetting.

Option 1: Change difficulty
```bash
./valheim_server.x86_64 \
-name "World1" \
-port 2456 \
-world "World1" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-configs/world1" \
-preset normal \
-crossplay
```
This Changes difficulty for World1
Just replace this line:

```bash
-preset normal
```

Option 2: Fully custom world modifiers

If you want direct control instead of presets:

```bash
./valheim_server.x86_64 \
-name "World1" \
-port 2456 \
-world "World1" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-configs/world1" \
-modifier combat hard \
-modifier raids less \
-modifier resources more \
-modifier portals casual \
-modifier deathpenalty easy \
-crossplay
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

Paste this:
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

---

World 2 Service
```bash
sudo nano /etc/systemd/system/valheim-world2.service
```
Paste this:
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
sudo systemctl enable --now valheim-world2
```

---

World 3 Service
```bash
sudo nano /etc/systemd/system/valheim-world3.service
```
Paste this:
```bash
[Unit]
Description=Valheim World3
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/home/user/valheim-server
ExecStart=/home/user/valheim-server/start_world3.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now valheim-world3
```

Logs:
```bash
journalctl -u valheim-world1 -f
journalctl -u valheim-world2 -f
journalctl -u valheim-world3 -f
```

# 11. Add Admins

You need the SteamID64 (numeric ID) of the player you want to make admin.

You can get it by:

-Pressing F2 in-game (while on the server) to show player list and IDs
-Or using a site like SteamID lookup (enter Steam profile URL)

Open ``` adminlist.txt ``` and add the SteamID64:

```bash
nano ~/.config/unity3d/IronGate/Valheim/adminlist.txt
```

Example:

```bash
76543211234567890
76543212345676543
```

Restart servers after changes.


# 12. Docker Setup (Recommended)

Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Log out & back in.



# 13. docker-compose.yml

Directories
```bash
mkdir -p ~/valheim-docker/world1
mkdir -p ~/valheim-docker/world2
mkdir -p ~/valheim-docker/world3
```

Create ```yml``` file
```bash
cd ~/valheim-docker
nano docker-compose.yml
```

Paste this:
```bash
services:

  world4:
    image: ghcr.io/lloesche/valheim-server
    container_name: valheim-world4
    restart: unless-stopped

    ports:
      - "2500:2456/udp"
      - "2501:2457/udp"
      - "2502:2458/udp"

    environment:
      SERVER_NAME: World4
      WORLD_NAME: World4
      SERVER_PASS: 123456

    volumes:
      - ./world4:/config


  world5:
    image: ghcr.io/lloesche/valheim-server
    container_name: valheim-world5
    restart: unless-stopped

    ports:
      - "2503:2456/udp"
      - "2504:2457/udp"
      - "2505:2458/udp"

    environment:
      SERVER_NAME: World5
      WORLD_NAME: World5
      SERVER_PASS: 123456

    volumes:
      - ./world5:/config


  world6:
    image: ghcr.io/lloesche/valheim-server
    container_name: valheim-world6
    restart: unless-stopped

    ports:
      - "2506:2456/udp"
      - "2507:2457/udp"
      - "2508:2458/udp"

    environment:
      SERVER_NAME: World6
      WORLD_NAME: World6
      SERVER_PASS: 123456

    volumes:
      - ./world6:/config
```

# 14. Start Docker Servers

Start:
```bash
docker compose up -d
```
Check:
```bash
docker ps
```

Logs:
```bash
docker logs -f valheim-world1
```

Stop:
```bash
docker compose down
```
