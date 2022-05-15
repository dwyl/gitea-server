# gitea-server
Step-by-step deployment instructions for getting Gitea running on Fly.io

- Create a new Fly.io application: `flyctl launch --name gitea-server --no-deploy`
- Create a volume for managing Gitea data with Sqlite: `flyctl volumes create gitea_data --size 1 --app gitea-server`
- Copy the `fly.toml` file (update the name of the fly.io application, ie replage `gitea-server`)
- Deploy the application: `flyctl deploy`

see: https://blog.gitea.io/2022/04/running-gitea-on-fly.io/
