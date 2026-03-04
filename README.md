# Forge

Monorepo for the Forge platform, composed of three services managed as git submodules.

## Projects

| Directory | Description | Stack |
|---|---|---|
| `ForgeAI/` | Web application & API | Ruby on Rails 7, PostgreSQL, Redis |
| `ForgeAI-Mobile/` | Mobile application | React Native / Expo |
| `ForgeAIService/` | AI/ML microservice | Python 3.13, FastAPI |

## Getting Started

### Clone (with all sub-projects)

```bash
git clone --recurse-submodules https://github.com/jon-sayer/Forge.git
cd Forge
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

### Pull latest changes across all sub-projects

```bash
git submodule update --remote --merge
```

### Run all services locally with Docker

```bash
docker compose up --build
```

This starts:

| Service | URL |
|---|---|
| **ForgeAI** (Rails) | http://localhost:3000 |
| **ForgeAIService** (FastAPI) | http://localhost:8000 |
| **ForgeAI-Mobile** (Expo/Metro) | http://localhost:8081 |
| **PostgreSQL** | localhost:5432 |
| **Redis** | localhost:6379 |

### Accessing the mobile app with Expo Go

The Metro bundler runs inside Docker and serves the JavaScript bundle to your device.

#### iOS Simulator (same machine)

No extra config needed. With the services running, open a second terminal:

```bash
cd ForgeAI-Mobile
npx expo start --ios --port 8081
```

Or press `i` in the `mobile` container logs to launch the simulator.

#### Physical device (iPhone / Android on the same Wi-Fi)

Your phone needs to reach the Metro bundler on your Mac's LAN IP. Before starting the services, set your LAN IP:

```bash
# macOS — find your LAN IP
export EXPO_HOST_IP=$(ipconfig getifaddr en0)

# Start everything
docker compose up --build
```

Once running:

1. Install **Expo Go** on your phone from the App Store / Play Store.
2. Open the `mobile` container logs — you'll see a QR code printed by Expo.
3. **iPhone**: scan the QR code with the Camera app. It will open in Expo Go.
4. **Android**: open Expo Go and tap "Scan QR code".

If the QR code doesn't appear in the logs, open `http://<your-lan-ip>:8081` in your phone's browser.

#### Tunnel mode (any network / remote device)

The project includes a Cloudflare tunnel script that gives you a public URL, no LAN required. This needs `cloudflared` installed:

```bash
# Install cloudflared (macOS)
brew install cloudflare/cloudflare/cloudflared

# Copy it somewhere the container expects
cp $(which cloudflared) /tmp/cloudflared

# Run outside Docker for simplicity
cd ForgeAI-Mobile
npm install
npm run start:tunnel
```

This prints a QR code with a public `exps://` URL you can scan from any device.

### Working on a single sub-project

Each sub-project is a standalone repo. Navigate into the directory and work as usual:

```bash
cd ForgeAI
bundle install
bin/rails server
```

```bash
cd ForgeAI-Mobile
npm install
npx expo start
```

```bash
cd ForgeAIService
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

## Submodule Workflow

Submodules track a specific commit. After making changes inside a sub-project:

```bash
cd ForgeAI          # work inside the submodule
git add . && git commit -m "your change"
git push

cd ..               # back to root
git add ForgeAI     # update the root's pointer to the new commit
git commit -m "Update ForgeAI submodule"
git push
```
