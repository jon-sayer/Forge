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
- **ForgeAI** (Rails) on `http://localhost:3000`
- **ForgeAIService** (FastAPI) on `http://localhost:8000`
- **ForgeAI-Mobile** (Expo/Metro) on `http://localhost:8081`
- **PostgreSQL** on `localhost:5432`
- **Redis** on `localhost:6379`

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
