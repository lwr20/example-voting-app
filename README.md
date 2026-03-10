# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## CI – building and pushing images

The repository includes a GitHub Actions workflow (`.github/workflows/docker-build-push.yml`) that builds all four service images and pushes them to the GitHub Container Registry (GHCR) on every push to `main`.

### How authentication works

The workflow authenticates to GHCR using the **`GITHUB_TOKEN`** that GitHub Actions provisions automatically for every workflow run — no manually-created secret is required.

The token is exposed as an environment variable in the workflow:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### One-time repository setting

For the token to have permission to write to GHCR, you must enable **Read and write permissions** for workflow tokens in the repository settings:

1. Open the repository on GitHub and go to **Settings → Actions → General**.
2. Scroll to the **Workflow permissions** section.
3. Select **Read and write permissions**.
4. Click **Save**.

That is the only setup step required. Once this setting is saved, any push to `main` will build and push the following images to `ghcr.io/<owner>/example-voting-app-<service>`:

| Service | Image |
|---------|-------|
| vote | `ghcr.io/<owner>/example-voting-app-vote` |
| result | `ghcr.io/<owner>/example-voting-app-result` |
| worker | `ghcr.io/<owner>/example-voting-app-worker` |
| seed-data | `ghcr.io/<owner>/example-voting-app-seed-data` |

Pull requests against `main` trigger a build-only run (no push) so you can verify images build correctly before merging.

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
