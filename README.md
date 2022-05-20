<div align="center">

# `Gitea` _Server_

<img alt="Gitea" src="https://user-images.githubusercontent.com/194400/168781665-a52d2c00-8b69-44ae-a10a-7bd1c3932020.svg" width="240"/>

**_Step-by-step_ deployment instructions**
for **`Gitea`** on
[**Fly.io**](https://fly.io)

## Try it: [gitea-server.fly.dev](https://gitea-server.fly.dev)

</div>

## How?

This is a step-by-step guide for recreating our server.
If you find it useful, please ⭐

### 0. Prerequisites

**a.** Fly CLI installed:
https://fly.io/docs/flyctl/ <br />
**b.** Fly account + authenticated on your `localhost`:
https://fly.io/docs/flyctl/auth-signup/ <br />
**c.** Basic understanding of deployment. <br />

### 1. Create a Fly App

Create a Fly.io App
for the Gitea Server
(but don't deploy it yet!):

```sh
flyctl launch --name gitea-server --image gitea/gitea --org gitea --no-deploy
```

That will generate a
[`fly.toml`](https://github.com/dwyl/gitea-server/blob/main/fly.toml)
file.

### 2. Create a Volume for Data Storage

`Gitea` will store `git` repo files/data on a persistent storage volume.
Create it with the following command:

```sh
flyctl volumes create gitea_data --size 1 --app gitea-server
```

### 3. Update the `fly.toml` File

The file created by the `flyctl` CLI
will be the default:
[`fly.toml@4494a659`](https://github.com/dwyl/gitea-server/commit/4494a659fbcba2ac4adc616342c9a15579937108#diff-660f7078dc0b381a350960eae8f3e924e739994dd8657dcc0d270a28c6ff8a38)
Sadly that's not enough to run `Gitea`.

The file needs to look more like this:

```yaml
# fly.toml file generated for gitea-server on 2022-05-14T15:27:31+01:00
app = "gitea-server"

kill_timeout = 5

[build]
  image = "gitea/gitea:latest" # latest is the most recent stable release

[env]
  GITEA__database__DB_TYPE="sqlite3"
  GITEA__database__PATH="/data/gitea/gitea.db"
  GITEA__server__DOMAIN="gitea-server.fly.dev"
  GITEA__server__SSH_DOMAIN="gitea-server.fly.dev"
  GITEA__server__ROOT_URL="https://gitea-server.fly.dev"
  GITEA__server__REDIRECT_OTHER_PORT="true" # listen on port 80, and redirect to "ROOT_URL"
  GITEA__security__INSTALL_LOCK="true" # Don't show installer
  # GITEA__service__DISABLE_REGISTRATION="true" # TODO: uncomment once you have created your first user

# persist data
[[mounts]]
  destination = "/data"
  source = "gitea_data"

# ssh traffic
[[services]]
  internal_port = 22
  protocol = "tcp"
  [[services.ports]]
    port = 22

# for http->https redirect
[[services]]
  internal_port = 80
  protocol = "tcp"

  [[services.ports]]
    handlers = ["http"]
    port = 80

# https traffic
[[services]]
  internal_port = 3000
  protocol = "tcp"

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443
```

See:
[`commits/54d77542`](https://github.com/dwyl/gitea-server/pull/2/commits/54d77542ada8f1bda5a107f831c9a1acfa583160)
for the changes made.

### 4. Deploy!

Once you have saved the `fly.toml` file,
deploy it with the following command:

```sh
LOG_LEVEL=debug fly deploy --verbose
```

### 5. Configure

Visit your newly deployed instance
and configure it as desired:
https://gitea-server.fly.dev

## Recommended Reading

- Running Gitea on fly.io:
  https://blog.gitea.io/2022/04/running-gitea-on-fly.io/
  recently written/published by
  [**`@techknowlogick`**](https://github.com/techknowlogick)
  a core contributor to Gitea.

[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square)](https://github.com/dwyl/gitea-demo/issues)
[![HitCount](http://hits.dwyl.com/dwyl/gitea-demo.svg)](http://hits.dwyl.com/dwyl/gitea-demo)
