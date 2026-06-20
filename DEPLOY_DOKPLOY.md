# Dokploy deployment from fork

This fork deploys `thoa100m/CLIProxyAPI` while still tracking upstream updates from `router-for-me/CLIProxyAPI`.

## Runtime layout

- Backend image: `ghcr.io/thoa100m/cliproxyapi:latest`
- Management panel source: `https://github.com/thoa100m/Cli-Proxy-API-Management-Center`
- API/control panel port: `8317`
- Control panel URL: `http://<host>:8317/management.html`

## Dokploy compose

Use `docker-compose.dokploy.yml` in Dokploy.

It mounts:

- `../files/volumes/config.yaml` -> `/CLIProxyAPI/config.yaml`
- `cli-proxy-auth` -> `/root/.cli-proxy-api`
- `cli-proxy-logs` -> `/CLIProxyAPI/logs`
- `cli-proxy-static` -> `/root/.cli-proxy-api/static`

## Required config.yaml settings

Set these values in the mounted config file:

```yaml
remote-management:
  allow-remote: true
  secret-key: "CHANGE_THIS_TO_A_LONG_RANDOM_VALUE"
  disable-control-panel: false
  disable-auto-update-panel: false
  panel-github-repository: "https://github.com/thoa100m/Cli-Proxy-API-Management-Center"
```

`secret-key` is the Management key used by the Web UI. It is different from proxy `api-keys`.

## Backend image publishing

The workflow `.github/workflows/ghcr-image.yml` publishes:

- `ghcr.io/thoa100m/cliproxyapi:latest`
- `ghcr.io/thoa100m/cliproxyapi:<branch-or-tag>`

It runs on pushes to `main`, tags `v*`, and manual dispatch.

## Management UI publishing

The management UI fork already publishes `management.html` when a `v*` tag is pushed.

Recommended release flow:

```bash
cd ../Cli-Proxy-API-Management-Center
git fetch upstream
git merge upstream/main
git push origin main
git tag v$(date +%Y.%m.%d.%H%M)
git push origin --tags
```

The backend periodically checks the latest release from `panel-github-repository` and updates `management.html` automatically.

## Upstream sync

Backend:

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

Management UI:

```bash
cd ../Cli-Proxy-API-Management-Center
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

Resolve conflicts in custom modules first, then push again.
