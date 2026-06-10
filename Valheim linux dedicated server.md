
# Valheim Dedicated Server 

A complete guide to setup a dedicated Valheim server on LinuxMint/Ubuntu

Usefull tools

```bash
sudo apt update && sudo apt install -y btop mc openssh-server rsync vlc tree
```


---

# Table of Contents

- [1. Install Dependencies](#1-install-dependencies)
- [2. Install SteamCMD](#2-install-steamcmd)
- [3. Install Valheim Dedicated Server](#3-install-valheim-dedicated-server)
- [4. Fix Steam Runtime](#4-fix-steam-runtime)
- [5. Generate your world](#5-generate-your-world)
- [6. Open firewall ports](#6-open-firewall-ports)
- [7. Create Startup Script](#7-native-startup-script)
- [8. World Modifiers](#8-world-modifiers)
- [9. Fine-tuned Modifiers (advanced)](#9-fine-tuned-modifiers)
- [10. systemd Services](#10-systemd-services-auto-start)
- [11. Admin Setup](#11-add-admins)
- [12. Docker Setup](#12-docker-setup-recommended)
- [13. docker-compose](#13-docker-composeyml)
- [14. Start Docker](#14-start-docker-servers)
---

# 1. Install Dependencies

Open a terminal and install the basics:

```bash
sudo apt update
sudo apt install curl wget tar lib32gcc-s1
```

# 2. Install SteamCMD

This is what you’ll use to download the server:

```bash
mkdir ~/steamcmd && cd ~/steamcmd
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
```

# 3. Install Valheim Dedicated Server

Run SteamCMD:

```bash
./steamcmd.sh
```

Inside SteamCMD:
```bash
force_install_dir /home/$USER/valheim-server/
login anonymous
app_update 896660 validate
quit
```


# 4. Fix Steam Runtime

```bash
mkdir -p ~/.steam/sdk64
cp ~/steamcmd/linux64/steamclient.so ~/.steam/sdk64/
```


# 5. Generate your world

Generate your world locally in the game client using:

```bash
Name: TestWorld
Seed: 42069lolxd
```

or any other combination of Name/Seed you want to use.

```bash
World  Seed
World1	Valheim
World2	tyeDWcN7cp
World3	ym6RwFNELS
World4  UDiMgesqtu
World5  p7iWJbr2V4
World6  cAD0nzXZpv
```

Then copy the files ```TestWorld.db``` and ```TestWorld.fwl``` from your game client to:

```bash
~/.config/unity3d/IronGate/Valheim/worlds_local/
```

# 6. Open firewall ports

Default ports (important)

A single Valheim server uses:

```UDP 2456–2458```

Allow them:

```bash
sudo ufw allow 2456:2458/udp
```



# 7. Create Startup Script

Go to your server folder:

```bash
cd ~/valheim-server
nano start_server.sh
```

Paste this:
```bash
#!/bin/bash

export templatedir=$PWD

./valheim_server.x86_64 \
-name "TestWorld" \
-port 2456 \
-world "TestWorld" \
-password "123456" \
-public 1 \
-savedir "$HOME/.config/unity3d/IronGate/Valheim" \
-crossplay \
```

Make it executable

```bash
chmod +x start_server.sh
```

# 8. World Modifiers

Edit your server startup script for example ```start_world1.sh``` and add world modifier flags.

Option 1: Change difficulty

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

Just replace this line:

```bash
-preset normal
```

Example:
```bash
./valheim_server.x86_64 \
-name "TestWorld" \
-port 2456 \
-world "TestWorld" \
-password "123456" \
-public 1 \
-savedir "$HOME/.config/unity3d/IronGate/Valheim" \
-preset hammer \
-crossplay
```

This Changes difficulty for World1

# 9. Fine-tuned Modifiers (advanced)

You can override specific systems using ```-modifier:```:

```bash
-modifier combat hard
-modifier raids less
-modifier resources more
-modifier portals casual
-modifier deathpenalty easy
```

Common values:

-Combat: ```veryeasy / easy / hard / veryhard```

-Raids: ```none / less / more / muchmore```

-Resources: ```less / more / muchmore```

-Portals: ```casual / hard / veryhard```

-Death penalty:```casual / easy / hard / hardcore```



```bash
./valheim_server.x86_64 \
-name "World1" \
-port 2456 \
-world "World1" \
-password "123456" \
-public 1 \
-savedir "$HOME/valheim-server/world1" \
-modifier combat hard \
-modifier raids less \
-modifier resources more \
-modifier portals casual \
-modifier deathpenalty easy \
-crossplay
```

#  Open Firewall Ports

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
Description=Valheim Server: WorldX
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/home/user/valheim-server
ExecStart=/home/user/valheim-server/start_worldx.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
replace ```user``` with your ```username``` but do not change the ```WantedBy=multi-user.target``` line.


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
World Modifiers for Docker

If you’re using a popular image like ```lloesche/valheim-server```, you configure everything via ```environment:```.

Example ```docker-compose.yml```

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
      VALHEIM_PRESET: normal

    volumes:
      - ./world4:/config
```
To change world modifiers (recommended way) Just edit:

```bash
- VALHEIM_PRESET=normal
```

Options:
```bash
casual
normal
easy
hard
hardcore
immersive
hammer
```

Custom modifiers (if your image supports it)

Some images allow fine-tuning:

```bash
environment:
  - VALHEIM_MODIFIER_COMBAT=hard
  - VALHEIM_MODIFIER_RAIDS=less
  - VALHEIM_MODIFIER_RESOURCES=more
  - VALHEIM_MODIFIER_PORTALS=casual
  - VALHEIM_MODIFIER_DEATHPENALTY=easy
```

Not all Docker images support this. If ignored, your image likely only supports ```VALHEIM_PRESET```.

Important Docker folder structure (THIS MATTERS)

Because you set:
```bash
WORLD_NAME=World4
```
Your mounted volume:
```bash
$HOME/valheim-server/world1:/config
```
Inside container, world files must be:
```bash
/config/worlds_local/World4.db
/config/worlds_local/World4.fwl
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
