# OpenLedgerHQ
* In this guide, I will cover these methods to install Openledger Worker Node: **1.Chrome-Extension** **2.Windows** **3.Linux (CLI)**
* We can add as many nodes as we want to our dashboard
* Rewards rate for each Node:

![image](https://github.com/user-attachments/assets/683cd53e-1c8f-4988-8ed1-061fe66c44b8)

* Maximize Activity: Ensure your node stays online and actively participates in the network to reach the hourly cap.
* Network Uptime is Critical: Downtime or uninstalled nodes will result in point loss.

# Register Dashboard
* Login to **[Dashboard](https://testnet.openledger.xyz/?referral_code=rxtwnznbki)** with gmail

* We have different available installation methods you can choose in App-Store

![image](https://github.com/user-attachments/assets/889218c1-673c-4083-922e-f82e043172c9)

## 🟠 Node Installation (Chrome Extension)
**1. Login to [Dashboard](https://testnet.openledger.xyz/?referral_code=rxtwnznbki)**

**2. Install [Extension](https://chromewebstore.google.com/detail/openledger-node/ekbbplmjjgoobhdlffmgeokalelnmjjc) and login**

> If you already have a Linux VPS without a GUI desktop, you can run extension node by installing Chromium browser

## 🟠 Node Installation (Windows)
Through this method, I install multiple Nodes on my Windows-Server VPS and an idle PC

**1. Download Docker [here](https://docs.docker.com/desktop/setup/install/windows-install)**

**2. Install Docker , then Restart your system**

**3. Install & Enable WSL

**4. Run Docker (no need to login), and wait until Docker Engine starts running**

**5. [Download](https://testnet.openledger.xyz/app-store), unzip and install Openledger Node (Windows Version)**

![image](https://github.com/user-attachments/assets/b5634cc5-2559-48bc-a443-9347456c3529)

![Screenshot_554](https://github.com/user-attachments/assets/31581094-8854-4b71-994e-a862661f1095)


**6. Open Node app, login using your dashboard's gmail, setup your Node and start running**

![image](https://github.com/user-attachments/assets/39d52676-5a12-4681-b24d-ab746b3f5995)

## 🟠 Check Points
You can check your Online Nodes and Points [here](https://testnet.openledger.xyz/dashboard)

#

### 🟠 Node Installation (Linux CLI) soon...




sudo apt update
sudo apt install autocutsel

nano ~/.vnc/xstartup

#!/bin/bash
xrdb $HOME/.Xresources
autocutsel -fork
autocutsel -selection PRIMARY -fork
startxfce4 &

chmod +x ~/.vnc/xstartup

vncserver -kill :1

