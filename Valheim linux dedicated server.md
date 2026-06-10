
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
- [10. Run Server](#10-run-server)
- [11. systemd Service (Auto Start)](#11-auto-start)
- [12. Add admins](#12-add-admins)

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

```bash
cd ~/
mkdir valheim-server
```
Run SteamCMD:

```bash
cd ~/steamcmd
./steamcmd.sh
```

Inside SteamCMD:

!Note, replace ```user``` with your ```username```

```bash
force_install_dir /home/user/valheim-server/
login anonymous
app_update 896660 validate
quit
```

Run the server once

```bash
cd ~/valheim-server/
./valheim_server.x86_64
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

on your server create a temporaty folder to transfer the files ```TestWorld.db``` and ```TestWorld.fwl``` from your game client

```bash
mkdir ~/testworld
cd testworld
```
affter both lifes are in your ```testworld``` forlder run



```bash
cd ~/testworld/
cp TestWorld.db TestWorld.fwl ~/.config/unity3d/IronGate/Valheim/worlds_local/
ll ~/.config/unity3d/IronGate/Valheim/worlds_local/
```
you should see the files in ```~/.config/unity3d/IronGate/Valheim/worlds_local/```

```bash
./
../
Dedicated.fwl
DevWorld.fwl
TestWorld.db*
TestWorld.fwl*
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

delete everything and paste this:

```bash
#!/bin/bash

export templatedir=$PWD
export SteamAppId=892970

echo "Starting Valheim server PRESS CTRL-C to exit"

./valheim_server.x86_64 \
-name "TestWorld" \
-port 2456 \
-world "TestWorld" \
-password "123456" \
-public 1 \
-savedir "/home/user/.config/unity3d/IronGate/Valheim" \
-crossplay
```
Replace ```user``` on line ```-savedir "/home/user/.config/unity3d/IronGate/Valheim" \``` with your ```username```


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

This Changes difficulty for World

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

# 10. Run Server

```bash
cd ~/valheim-server
./start_server.sh
```

Connect:

```bash
server-IP:2456
server-IP:2459
server-IP:2462
```

Logs:

```bash
journalctl -u valheim-world1 -f
```

# 11. systemd Service (Auto Start)

Create a service file

```bash
sudo nano /etc/systemd/system/valheim.service
```

Paste this:
```bash
[Unit]
Description=Valheim Server
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/home/user/valheim-server
ExecStart=/home/user/valheim-server/start_server.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
replace ```user``` with your ```username``` but do not change the ```WantedBy=multi-user.target``` line.


Enable:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now valheim.service
sudo reboot
```

After reboot check the service is running

```bash
sudo systemctl status valheim
```
Useful commands

```bash
sudo systemctl status valheim   # check if running
sudo systemctl stop valheim     # stop server
sudo systemctl restart valheim  # restart
journalctl -u valheim -f        # live logs (VERY useful)
```

# 12. Add Admins

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

# Your server is ready, have fun.

