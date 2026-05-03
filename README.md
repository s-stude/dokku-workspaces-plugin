# dokku-workspaces

Deploy npm workspaces from a monorepo with shared dependencies.

Unlike [dokku-monorepo](https://github.com/notpushkin/dokku-monorepo), this plugin keeps all files in the repo intact so that cross-referenced packages (e.g. shared libraries) remain available at build time and runtime. It only generates a `Procfile` pointing to the correct workspace's start script.

## Install

```bash
sudo dokku plugin:install https://github.com/s-stude/dokku-workspaces-plugin.git dokku-workspaces
```

## Usage

### 1. Declare workspaces in your root `package.json`

The workspace directories listed in `.dokku-workspaces` must also be declared in your root `package.json`:

```json
{
  "workspaces": [
    "packages/app",
    "packages/admin",
    "packages/api"
  ]
}
```

The `-w` flag used in the generated Procfile resolves workspace names through npm. If a workspace directory is missing from this array, the deploy will fail with `No workspaces found`.

### 2. Add a `.dokku-workspaces` file to your repo root

Map Dokku app names to workspace directories:

```
# app-name=workspace-dir
my-app=packages/app
my-admin=packages/admin
my-api=packages/api
```

### 3. Create Dokku apps on the server

```bash
dokku apps:create my-app
dokku apps:create my-admin
dokku apps:create my-api
```

### 4. Add git remotes and push

```bash
git remote add dokku-app dokku@your-server:my-app
git remote add dokku-admin dokku@your-server:my-admin
git remote add dokku-api dokku@your-server:my-api

git push dokku-app master    # deploys packages/app
git push dokku-admin master  # deploys packages/admin
git push dokku-api master    # deploys packages/api
```

## How it works

On each deploy, the plugin:

1. Reads `.dokku-workspaces` from the repo root
2. Matches the Dokku app name against the config entries
3. Writes a `Procfile` with `web: npm run start -w <workspace>`

All files remain in the repo, so `npm install` at the root resolves all workspace cross-references normally.

## License

[MIT](LICENSE)
