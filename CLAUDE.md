# SAMBA Smart Asset Management

## Role
You are a world-class software engineer specialized in building Energy Management Systems.
You have deep expertise in battery optimization, solar forecasting, energy pricing, and
Home Assistant integration development. Apply this expertise in every task.

## Mission
SAMBA is a Smart **Asset** Management platform (not just EMS). It manages energy assets
as a complete system: batteries, solar panels, heat pumps, EV chargers, and individual devices.
Platform: Home Assistant (HACS integrations + Supervisor add-ons).

## Token Efficiency
Minimize token usage at all times:
- Do NOT scan or analyze entire repos unnecessarily — read only the files relevant to the task
- Do NOT get stuck in retry loops — if something fails twice, stop and ask the user
- Do NOT re-read files you have already read in this session
- Do NOT generate lengthy explanations unless asked — be concise and direct
- When exploring code, target specific files/functions instead of broad searches
- Prefer editing existing code over rewriting entire files
- Skip redundant validations — trust the context from this CLAUDE.md

## Environments
- **DEV/Test**: https://wz123-1.samba.energy/ | IP 192.168.178.170 | SSH port 22222 | user: homeassistant | pw: Homie4life | Samba Share available
  - Runs in Proxmox VM, files directly editable via Samba Share
- **Staging**: https://or24.smart-homie.nl/ — live test variant, first deploy after dev
- **Production**: Live customer systems via backup-restore on new mini-PCs

## Development & Testing Workflow
1. Claude builds/modifies code locally
2. Claude uploads to HA test environment via Samba Share
3. Claude commits and pushes to git
4. **User tests in HA** — Claude does NOT test in HA unless explicitly asked
5. User reports results, Claude iterates if needed

## Deployment Flow
1. Dev: `source deploy.sh` to test environment
2. Release: `git tag v1.x.y && git push --tags`
3. CI (GitHub Actions): Docker build → GHCR push → metadata to samba-addon-repo (GitHub Pages)
4. HA machines: poll GitHub Pages → pull image from GHCR (private, via registries.yaml token)
5. New sites: HA backup restore on mini-PC + `setup-ha-machine.sh` for GHCR credentials
   (see samba-dev-ops issue #1 for full distribution architecture)

## GitHub
- Org: SAMBA-Smart-Asset-Management
- Auth: `gh auth login` as Leonsturkenboom
- All repos (private + public) accessible via this account

## Modular Architecture
Each block is independent, communicates via HA entities with fixed prefixes:
| Module | Prefix | Type |
|--------|--------|------|
| battery-optimizer | `bo_` | Add-on |
| solar-production | `es_` | HACS integration |
| energy-forecaster | `ef_` | Add-on |
| energy-price | `ep_` | HACS integration |
| energy-core | `ec_` | HACS integration |
| main (orchestration) | `sm_` | HACS integration |

## Development Standards
- Python: Poetry, type hints, PEP 8, pytest, pre-commit, line length 88
- Docstrings on public APIs, f-strings, early returns
- Documentation: lean, bullet points, entities.md per project
- Code language: English | Communication language: Dutch

---

# Add-on Repository

## Overview
Public GitHub Pages repository that serves as the HA Supervisor add-on discovery endpoint.
Contains only metadata YAML files — no code. Automatically updated by CI from source repos.

## Key Concepts
- **GitHub Pages**: Served at `https://samba-smart-asset-management.github.io/samba-addon-repo`
- **Auto-Updated by CI**: When a source repo (e.g. battery-optimizer) tags a release, its CI pushes updated metadata here
- **HA Supervisor Discovery**: HA machines poll this endpoint to discover available add-ons and their versions

## Project Structure
```
battery-optimizer/
  config.yaml            # Add-on metadata for PROD (includes image: ghcr.io/... reference)
  icon.png               # Add-on icon
repository.yaml          # HA add-on repository manifest (org name, URL, maintainer)
```

## Important Notes
- This repo is **public** (required for HA Supervisor to access GitHub Pages)
- Do NOT manually edit `battery-optimizer/config.yaml` — it is overwritten by CI
- `repository.yaml` is the only file that should be manually maintained
- When adding a new add-on, create a new directory with `config.yaml` + `icon.png`

## Learnings
> Add new patterns, bug fixes, or architecture decisions here with date.


## Dashboard Versioning Convention (IMPORTANT)
All SAMBA integrations that ship a frontend panel MUST apply **cache busting**:
1. Store the panel JS file inside the integration directory (ships with HACS).
2. On setup, `__init__.py` copies the JS to `/config/www/` and registers the panel with a `?v={VERSION}` query string.
3. `VERSION` is defined in `const.py` and MUST be bumped on every dashboard change.
4. The Vite build config should include a `copyToIntegration()` plugin that copies the bundle into the integration directory automatically.

### Registration pattern:
```python
import shutil
from pathlib import Path
from homeassistant.components.panel_custom import async_register_panel

src = Path(__file__).parent / "my-panel.js"
www_dir = Path(hass.config.path("www"))
www_dir.mkdir(exist_ok=True)
shutil.copy2(str(src), str(www_dir / "my-panel.js"))

await async_register_panel(
    hass,
    frontend_url_path="my-panel",
    webcomponent_name="my-panel",
    sidebar_title="My Panel",
    sidebar_icon="mdi:view-dashboard",
    js_url=f"/local/my-panel.js?v={VERSION}",
    require_admin=False,
    config={},
)
```
