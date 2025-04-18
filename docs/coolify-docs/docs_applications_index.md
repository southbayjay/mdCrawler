Skip to content
Menu
Return to top
# Applications ​
Application could be any type of web application. It could be a static site, a NodeJS application, a PHP application, etc.
For complex applications, you can use Docker Compose based deployments or the one-click services.
## Examples ​
INFO
The list is not complete.
You can host almost any application that you want, not just the ones listed here.
  * Vite
  * Django
  * Jekyll
  * Vue.js
  * Next.js
  * Nuxt
  * Laravel
  * Symfony
  * Ruby on Rails
  * SvelteKit


## General Configuration ​
### Commands ​
You can overwrite the default commands by setting a custom value on the UI.
  * Build
  * Install
  * Start


INFO
If you leave it empty, Nixpacks will detect which commands to run. For example, in Nodejs, it will check the lock files and run `npm ci` or `yarn install` or `pnpm install` accordingly.
### Base Directory ​
It is useful for monorepos. You can set the base directory for all the commands that will be executed by Coolify.
### Public Directory ​
If you are building a static site, it is important to set the public directory, so the builder will know which directory to serve.
### Port Exposes ​
Port exposes are required for Docker Engine to know which ports to expose. The first port will be the default port for health checks.
Examples:
If you have a NodeJS application that listens on port 3000, you can set it like this: `3000`. If you have a PHP-FPM application that listens on port 9000, you can set it like this: `9000`. If you have a Nginx server that listens on port 80, you can set it like this: `80`.
### Port Mappings ​
WARNING
You will lose some functionality if you map a port to the host system, like `Rolling Updates`.
If you would like to map a port to the host system (server), you can do it here like this: `8080:80`.
This will map the port 8080 on the host system to the port 80 inside the container.
INFO
If you would like to get performance boost and you do not need any domain (websocket server with VERY high traffic), you can map its port to the host, so the request will not go through the proxy.
## Advanced ​
### Static Site (Is it a static site?) ​
> This feature is only available for Nixpacks buildpacks.
If you need to serve a static site (SPA, HTML, etc), you can set this to `true`. It will be served by Nginx. `Disabled by default`.
### Force HTTPS ​
If you would like to force HTTPS, so no HTTP connections allowed, you can set this to `true`. `Enabled by default`.
### Auto Deploy ​
> This feature is only available for GitHub App based repositories.
If you would like to deploy automatically when a new commit is pushed to the repository, you can set this to `true`. `Enabled by default`.
### Preview Deployments ​
Preview deployments are a great way to test your application before merging it into the main branch. Imagine it like a staging environment.
#### URL Template ​
You can setup your preview URL with a custom template. Default is `{{pr_id}}.{{domain}}`.
This means that if you open a Pull Request with the ID `123`, and you resource domain is `example.com` the preview URL will be `123.example.com`.
TIP
If you have several domains for your resource, the first will be used as the{" "} `{{ domain }}` part.
#### Automated Preview Deployments ​
> This feature is only available for GitHub App based repositories.
If you would like to deploy a preview version of your application (based on a Pull Requests), you can set this to `true`. `Disabled by default`.
If set to `true`, all PR's that are opened against the resource's configured branch, will be deployed to a unique URL.
#### Manually Triggered Preview Deployments ​
You can manually deploy a Pull Request to a unique URL by clicking on the `Deploy` button on the Pull Request page.
### Git Submodules ​
If you are using git submodules, you can set this to `true`. `Enabled by default`.
### Git LFS ​
If you are using git lfs, you can set this to `true`. `Enabled by default`.
### Environment Variables ​
Read here
### Persistent Storage ​
Read here
### Health Checks ​
By default, all containers are checked for liveness.
WARNING
Traefik Proxy won't work if the container has health check defined, but it is `unhealthy`. If you do not know how to set up health checks, turn it off.
### Rollbacks ​
You can rollback to a previous version of your resource. At the moment, only local images are supported, so you can only rollback to a locally available docker image.
### Resource Limits ​
By default, the container won't have any resource limits. You can set the limits here. For more details, read the Docker documentation.
## Deployment Types ​
There are several types of application deployments available.
  * Public Git Repository
  * Private Git Repository (GitHub App)
  * Private Git Repository (Deploy Key)
  * Based on a Dockerfile
  * Based on a Docker Compose
  * Based on a Docker Image


## Build Packs ​
These are the supported build packs:
  * Nixpacks
  * Dockerfile
  * Docker Image
  * Docker Compose


### Nixpacks ​
Coolify uses Nixpacks as build pack by default. Nixpacks detect what kind of application are you trying to deploy and builds it accordingly.
But if needed, you can customize it by adding a `nixpacks.toml` file to your repository or setting the right environment variables.
For example, if you are using a NodeJS application, you can set the `NIXPACKS_NODE_VERSION` to control the NodeJS version.
TIP
Worth reading their documentation to understand how it works.
### Dockerfile ​
Dockerfile based build packs are useful if you have a custom dockerfile inside your Git repository.
#### Custom Dockerfile Location ​
You can always set your custom dockerfile location. By default, it is set to `/Dockerfile`.
### Docker Image ​
You can deploy any docker images from any Docker compatible repository. For example, you can deploy images from Docker Hub, GitHub Container Registry, etc.
### Docker Compose ​
Docker Compose based deployments are useful for complex applications. You can define multiple services in a single `docker-compose.yml` file and deploy them together.
  * Languages/Frameworks
  * Services


