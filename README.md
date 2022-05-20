# gitea-server

Step-by-step deployment instructions for getting Gitea running on Fly.io


## How?

This is a step-by-step guide for recreating our server.
If you find it useful, please ⭐

### Create an App

Create a Fly.io App for the Gogs Server:

```sh
flyctl launch --name gitea-server --image gogs/gogs --org dwyl
```