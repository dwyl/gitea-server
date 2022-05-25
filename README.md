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

<br />

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

<br />

### 2. Create a Volume for Data Storage

`Gitea` will store `git` repo files/data on a persistent storage volume.
Create it with the following command:

```sh
flyctl volumes create gitea_data --size 1 --app gitea-server
```

<br />

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

<br />

### 4. Deploy!

Once you have saved the `fly.toml` file,
deploy it with the following command:

```sh
LOG_LEVEL=debug fly deploy --verbose
```

<br />

### 5. Configure

Visit your newly deployed instance
and configure it as desired:
https://gitea-server.fly.dev

If you need to _manually_ configure anything,
the `gitea` configuration file
is located at
`/data/gitea/conf/app.ini`
on the gitea-server Fly app.

> **Note**: make a backup of your file before you make any changes!!

<br />

### 6. Add SSH Key

Add your `public` `ssh` key:
https://gitea-server.fly.dev/user/settings/keys

e.g:

![image](https://user-images.githubusercontent.com/194400/168495559-1f8649fa-43f7-4f16-a0a2-c09561e10318.png)

#### 6.1 Update the `.ssh/authorized_keys` file

Once you've added your `ssh key` to your user on the `Gitea` server,
Update the ``.ssh/authorized_keys` file by visiting:
https://gitea-server.fly.dev/admin

![gitea-server-refresh-ssh-authorized_keys](https://user-images.githubusercontent.com/194400/169619901-14d8fbac-ecf8-4bb3-85d4-fa805134bc1f.png)

You should see the following message confirming the request is being executed:

![image](https://user-images.githubusercontent.com/194400/169619944-d333eb03-509a-4cfc-8d5e-4224085b1a79.png)

#### 6.2 Test the `ssh` Connection

In your terminal run the following command:

```sh
ssh -T git@gitea-server.fly.dev
```

You should see output similar to the following:

```sh
Hi there, nelsonic!
You've successfully authenticated with the key named MBP 2022,
but Gitea does not provide shell access.
If this is unexpected, please log in with password and setup Gitea under another user.
```

Create a basic repo, e.g:
https://gitea-server.fly.dev/myorg/public-repo

`clone` it to your `localhost`:

```sh
git clone git@gitea-server.fly.dev:myorg/public-repo.git
```

Make a change to one of the files.
Then `git add . && git commit -m 'file updated' && git push`

`git push` (SSH) works as expected:

https://gitea-server.fly.dev/myorg/public-repo
![image](https://user-images.githubusercontent.com/194400/168496190-ac48b3de-2c65-406e-83af-dcc7cc570784.png)

Now that you've confirmed you can access a `git` repo
hosted on `gitea` server via your terminal
you can already use your server as fully fledged `GitHub` backup or even _replacement_!
But there's more to explore!

<br />

### 7. Create Access Token (API Key)

Visit:
https://gitea-server.fly.dev/user/settings/applications
and create a new **Access Tokens**.

e.g:

![image](https://user-images.githubusercontent.com/194400/169620530-765b0c15-e099-470e-b8c8-68c8967ed716.png)

Once you have the access token,
you can test it by running
a cURL command to retrieve repo info:

```sh
 curl 'https://gogs-server.fly.dev/api/v1/repos/myorg/public-repo?token=youraccesstokenhere'
```

Response:

```json
{
  "id": 3,
  "owner": {
    "id": 3,
    "login": "myorg",
    "full_name": "",
    "email": "",
    "avatar_url": "https://gitea-server.fly.dev/avatars/b665280652d01c4679777afd9861170c",
    "language": "",
    "is_admin": false,
    "last_login": "0001-01-01T00:00:00Z",
    "created": "2022-05-15T21:21:42Z",
    "restricted": false,
    "active": false,
    "prohibit_login": false,
    "location": "",
    "website": "",
    "description": "",
    "visibility": "public",
    "followers_count": 0,
    "following_count": 0,
    "starred_repos_count": 0,
    "username": "myorg"
  },
  "name": "public-repo",
  "full_name": "myorg/public-repo",
  "description": "",
  "empty": false,
  "private": false,
  "fork": false,
  "template": false,
  "parent": null,
  "mirror": false,
  "size": 104,
  "html_url": "https://gitea-server.fly.dev/myorg/public-repo",
  "ssh_url": "git@gitea-server.fly.dev:myorg/public-repo.git",
  "clone_url": "https://gitea-server.fly.dev/myorg/public-repo.git",
  "original_url": "",
  "website": "",
  "stars_count": 0,
  "forks_count": 0,
  "watchers_count": 1,
  "open_issues_count": 0,
  "open_pr_counter": 0,
  "release_counter": 0,
  "default_branch": "master",
  "archived": false,
  "created_at": "2022-05-15T21:40:54Z",
  "updated_at": "2022-05-15T21:40:57Z",
  "permissions": {
    "admin": true,
    "push": true,
    "pull": true
  },
  "has_issues": true
}
```

> **Note**: this API response is truncated for brevity,
> For the complete API docs,
> see: https://try.gitea.io/api/swagger#/repository

<br />

## Questions?

It wasn't all plain sailing for us when we first tried to setup our `gitea-server`.
We wrote this guide so that others wouldn't suffer our pain.

If you get stuck
in your deployment quest,
feel free to reach out by opening an issue:
[gitea-server/issues](https://github.com/dwyl/gitea-server/issues)

<br />

## Recommended Reading

- Running Gitea on fly.io:
  https://blog.gitea.io/2022/04/running-gitea-on-fly.io/
  recently written/published by
  [**`@techknowlogick`**](https://github.com/techknowlogick)
  a core contributor to Gitea.
- Gitea Configuration Cheat Sheet:
  https://docs.gitea.io/en-us/config-cheat-sheet/

[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square)](https://github.com/dwyl/gitea-server/issues)
[![HitCount](http://hits.dwyl.com/dwyl/gitea-server.svg)](http://hits.dwyl.com/dwyl/gitea-server)
