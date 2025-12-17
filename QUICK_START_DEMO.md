# Quick Start Demo

Run the Game SDK Playground locally in 3 simple steps.

---

## Step 1: Install Node.js

### Windows

**Option 1: Official Installer (Recommended)**

1. Go to [nodejs.org](https://nodejs.org/)
2. Download the **LTS** version
3. Run the installer and follow the wizard
4. Restart your terminal

**Option 2: Chocolatey**
```powershell
choco install nodejs-lts
```

**Option 3: winget**
```powershell
winget install OpenJS.NodeJS.LTS
```

---

### macOS

**Option 1: Homebrew (Recommended)**
```bash
brew install node
```

**Option 2: Official Installer**

1. Go to [nodejs.org](https://nodejs.org/)
2. Download the **LTS** version for macOS
3. Run the `.pkg` installer

**Option 3: nvm**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install --lts
```

---

### Linux

**Ubuntu / Debian:**
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Fedora:**
```bash
sudo dnf install nodejs
```

**Arch Linux:**
```bash
sudo pacman -S nodejs npm
```

---

### Verify Installation

```bash
node --version
npm --version
```

---

## Step 2: Download the Repository

**Option 1: Git clone**
```bash
git clone https://github.com/ArtIntGames/public-game-core-sdk.git
cd public-game-core-sdk
```

**Option 2: Download ZIP**

1. Go to [github.com/ArtIntGames/public-game-core-sdk](https://github.com/ArtIntGames/public-game-core-sdk)
2. Click **Code** → **Download ZIP**
3. Extract the archive
4. Open terminal in the extracted folder

---

## Step 3: Start the Server

```bash
node server.js
```

Output:
```
Server running at http://localhost:3333/
Open http://localhost:3333/Demo.html
```

---

## Step 4: Open Demo

Open in your browser:

**http://localhost:3333/Demo.html**

---

## Stopping the Server

Press `Ctrl + C` in the terminal.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `node: command not found` | Restart terminal after installation |
| Port 3333 in use | Change `PORT` in `server.js` |
| Cannot connect | Try `127.0.0.1:3333` instead of `localhost` |

---

