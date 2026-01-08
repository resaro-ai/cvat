# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CVAT (Computer Vision Annotation Tool) is an open-source video and image annotation platform for computer vision. It features a Django backend with REST API, React frontend, and specialized JavaScript libraries for 2D/3D annotation canvases.

## Architecture

```text
┌──────────────────────────────────────────────────────────────┐
│               Frontend (React/TypeScript)                    │
├──────────────────────────────────────────────────────────────┤
│ cvat-ui (Main UI) │ cvat-core (API client) │ cvat-canvas (2D)│
│                   │ cvat-data (types)      │ cvat-canvas3d   │
└────────────────────────────┬─────────────────────────────────┘
                             │ REST API (DRF)
┌────────────────────────────┴─────────────────────────────────┐
│                  Backend (Django 4.2)                        │
├──────────────────────────────────────────────────────────────┤
│ engine (core)    │ iam (auth)       │ dataset_manager (I/O)  │
│ organizations    │ lambda_manager   │ webhooks / events      │
│ quality_control  │ consensus        │ redis_handler          │
└──────────────────────────────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
   PostgreSQL 15      Redis (cache)      Kvrocks (job queue)
```

### Key Backend Apps (cvat/apps/)

- **engine**: Core annotation models (Project, Task, Job, Label, LabeledShape, LabeledTrack), REST viewsets, permissions
- **dataset_manager**: Import/export via Datumaro (40+ formats: COCO, YOLO, VOC, etc.)
- **lambda_manager**: Serverless auto-annotation functions (SAM, YOLO, etc.)
- **iam**: Authentication (Token, Session, Basic, AccessToken) and RBAC authorization
- **organizations**: Multi-tenancy support

### Key Frontend Modules

- **cvat-ui**: React app with Redux state management, Ant Design components
- **cvat-core**: REST API client, frame management, exported as `window.cvat`
- **cvat-canvas**: 2D annotation (Fabric.js-based), modes: IDLE, DRAW, EDIT, DRAG, RESIZE, ZOOM
- **cvat-canvas3d**: 3D point cloud visualization (Three.js)

## Commands

### Local Development Setup

```bash
# Start CVAT with Docker Compose
docker compose up -d

# Create admin user
docker exec -it cvat_server bash -ic 'python3 ~/manage.py createsuperuser'

# Access UI at http://localhost:8080
```

### Frontend Build

```bash
corepack enable yarn
yarn install --immutable

# Build individual modules
yarn run build:cvat-ui
yarn run build:cvat-core
yarn run build:cvat-canvas
yarn run build:cvat-canvas3d
yarn run build:cvat-data

# Development server
yarn run start:cvat-ui
```

### Backend Commands

```bash
# Run inside cvat_server container or with Django configured
python manage.py migrate
python manage.py collectstatic
python manage.py spectacular  # Generate OpenAPI schema
python manage.py makemigrations --check  # Verify no missing migrations
```

### Linting

```bash
# Python (run via pipx or install from dev/requirements.txt)
black --check --diff .
isort --check --diff --resolve-all-configs .
pylint -j0 .
bandit -a file --ini .bandit --recursive .

# JavaScript/TypeScript
yarn run eslint .
yarn run stylelint '**/*.css' '**/*.scss'

# Markdown
npx remark --quiet --frail -i .remarkignore .

# Spellcheck
typos
```

### Testing

```bash
# REST API and SDK tests (requires Docker services running)
cd tests/python
pip install -r requirements.txt
pytest -k "test_name"  # Run specific test
pytest --cov  # With coverage

# Start test services without running tests
pytest --start-services

# E2E tests use Cypress (see tests/cypress/)
```

## Code Style

- Python: Black (100-char lines), isort (Black profile), Python 3.10+
- TypeScript: ESLint with Airbnb config
- Line length: 100 characters (Python), configured per-module (JS)

## Key Files

- `cvat/schema.yml`: OpenAPI 3.0 schema (320KB) - regenerate with `manage.py spectacular`
- `cvat/settings/base.py`: Django settings
- `cvat/requirements/`: Pinned Python dependencies (pip-compile managed)
- `docker-compose.yml`: Main compose file with all services
- `docker-compose.dev.yml`: Development overrides

## SDK and CLI

```bash
# Install SDK
pip install cvat-sdk
pip install cvat-sdk[masks,pytorch]  # With extras

# Install CLI
pip install cvat-cli

# CLI examples
cvat-cli --auth user:pass task create <name> <files>
cvat-cli --auth user:pass task export <task_id> <format> <output>
```

## Database

PostgreSQL 15 with key models in `cvat/apps/engine/models.py`:

- Project → Task → Job (work unit with frame range)
- Label → LabeledShape/LabeledTrack (annotations)
- Annotation types: boxes, polygons, polylines, points, masks, skeletons, cuboids

## Annotation Format Support

Import/export via Datumaro: CVAT XML, COCO, YOLO (all variants), Pascal VOC, MOT, ImageNet, Cityscapes, KITTI, and 30+ more formats.
