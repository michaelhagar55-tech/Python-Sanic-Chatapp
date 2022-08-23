# Python Sanic Chat / Customer Service

A real-time customer service (“live chat”) application built with **Sanic**. It provides an embeddable visitor chat front end, an operator **CMS** for agents and administrators, WebSocket channels for messaging, and MongoDB-backed persistence.

## Features

- Visitor chat UI and floating window support (`/`, front blueprints)
- Staff CMS under **`/site_admin`** (users, sites, conversations, history, settings, quick replies, blacklists, exports, and more)
- Real-time messaging over WebSockets: **`/chat`** (visitors) and **`/serviceChat`** (agents)
- Multi-language UI strings (JSON under `static/easychat/assets/LANGUAGE/`)
- Redis for sessions and caching; MongoDB for application data
- Optional integrations (e.g. translation helpers, Telegram bot utilities in `modules/`)

## Requirements

- **Python** 3.8+ (recommended; match your deployment)
- **MongoDB** (local or remote; connection is configured in project settings)
- **Redis** on `localhost:6379` (used by `sanic_session` and `site_exts`)

## Configuration

Project-specific settings live in **`project_kfShare/__init__.py`** (`ProjectConfig`):

| Setting | Purpose |
|--------|---------|
| `PROJECT_NAME` | Logical project name (also used for static paths and Redis key prefixes) |
| `MONGODB_DB`, `MONGODB_USERNAME`, `MONGODB_PASSWORD`, `MONGODB_HSOT`, `MONGODB_PORT` | MongoDB database and connection |
| `MAIN_DOMAIN` | Primary domain used by the app |
| `PROJECT_ROOT_PATH` | Deployment path (referenced by tooling such as `gunScript.py`) |

**Security:** Replace default credentials and paths before production. Do not commit real secrets; use environment-specific config or secrets management.

Redis host/port are currently set in **`create_app.py`** (session pool) and **`site_exts.py`** (`StrictRedis`). Adjust if Redis is not on `localhost:6379`.

## Installation

From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install sanic sanic-ext sanic-cors sanic-session asyncio-redis aiofiles pymongo redis shortuuid click requests werkzeug openpyxl pillow user-agents flask-socketio eventlet gunicorn
```

Install any additional packages your environment reports as missing (e.g. `easygoogletranslate`, `pyotp`, `qrcode`, `python-authenticator`, `telegram` / `python-telegram-bot`, depending on which features you enable).

Ensure MongoDB and Redis are running and that the database user and database named in `ProjectConfig` exist.

## Running the app

**Development (Sanic built-in server):**

```bash
python app_kfShare.py
```

Default bind in code: **`0.0.0.0:5021`**. Open the CMS at `http://localhost:5021/site_admin` (see `constants.UrlPrefix.CMS_PREFIX`).

**Production-style:** The repo includes **`gunScript.py`** for **Gunicorn** with `eventlet` workers. Update `bind`, `accesslog`, and `errorlog` paths to match your server layout, then run Gunicorn with your usual process manager (e.g. systemd or Supervisor).

## CLI (`manage.py`)

Uses **Click**. Examples:

```bash
python manage.py init-admin --login-account admin --password yourpassword
python manage.py init-index
```

Other commands include database maintenance helpers (`remove_project_data`, `update_primary_key`, `update_secret_key`, etc.). Read the command docstrings in `manage.py` before running destructive operations.

## Project layout (overview)

| Path | Role |
|------|------|
| `app_kfShare.py` | App entry: builds Sanic app, language bundles, extra routes |
| `create_app.py` | Factory: blueprints, WebSockets, static routes, error handlers |
| `project_kfShare/` | Project package: `ProjectConfig`, `register_view` (blueprint wiring) |
| `views/` | `front_views`, `cms_views`, `common_views`, `api_views` |
| `models/` | MongoDB models |
| `templates/` | Jinja/HTML templates (`easychat`, `front`, …) |
| `static/` | Assets (Bootstrap, JS, chat UI, uploads area under project folders) |
| `gunScript.py` | Gunicorn configuration sample |

## License

Add a `LICENSE` file if you intend to distribute this project; none is included in the tree by default.
