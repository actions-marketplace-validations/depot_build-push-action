# Depot `build-push-action` GitHub Action

This action implements the same inputs and outputs as the [`docker/build-push-action`](https://github.com/docker/build-push-action), but uses the [`depot` CLI](https://github.com/depot/cli) to execute the build.

## Requirements

The `depot` CLI will need to be available in your workflow, you can use the [`depot/setup-action`](https://github.com/depot/setup-action) to install it:

```yaml
steps:
  - uses: depot/setup-action@v1
```

## Usage

This action implements the same inputs and outputs as the [`docker/build-push-action`](https://github.com/docker/build-push-action), see [the README](https://github.com/docker/build-push-action#readme) there for more information.

### Authentication

This action needs a Depot API token to communicate with your project's builders. You can supply this one of three ways:

1. You can supply a user or project API token via the `token` input:

   ```yaml
   steps:
     - uses: depot/build-push-action@v1
       with:
         token: ${{ secrets.DEPOT_TOKEN }}
   ```

2. You can supply a user or project API token via the `DEPOT_TOKEN` environment variable:

   ```yaml
   steps:
     - uses: depot/build-push-action@v1
       env:
         DEPOT_TOKEN: ${{ secrets.DEPOT_TOKEN }}
   ```

3. Depot supports GitHub's [OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect) tokens via a trust relationship, so your Actions builds can securely authenticate with your Depot projects without any static access tokens. To configure the trust relationship, visit your Depot project settings, then add your repository and desired workflow config to `Trust Relationships`. Then in your workflow, enable the `id-token: write` permission:

   ```yaml
   permissions:
     # allow issuing OIDC tokens for this workflow run
     id-token: write
     # allow at least reading the repo contents, add other permissions if necessary
     contents: read
   steps:
     # no need to provide a DEPOT_TOKEN
     - uses: depot/build-push-action@v1
   ```

### Differences from `docker/build-push-action`

1. Authentication — this action needs to authenticate with a Depot API token to communicate with your project's builders (see above).

2. If you have not configured a `depot.json` file with `depot init`, you can explicitly specify your project ID via the `project` input:

   ```yaml
   steps:
     - uses: depot/build-push-action@v1
       with:
         project: abc123xyz
   ```

3. The `builder` input is not supported - this action always runs builds using Depot's hosted builders, if you need to route builds to a local buildx builder, you should use the `docker/build-push-action`.
4. Depending on how you are using `cache-from` / `cache-to`, you may be able to remove those options. Depot's builders launch with persistent local SSD cache, so unless you wish to push your cache to a registry for other reasons, you can remove them:

   ```diff
   steps:
     -
   -  uses: docker/build-push-action@v1
   +  uses: depot/build-push-action@v1
      with:
   -    cache-from: type=gha
   -    cache-to: type=gha,mode=max
   ```

## Inputs and outputs

This action implements the same inputs and outputs as the [`docker/build-push-action`](https://github.com/docker/build-push-action) with two additional inputs to work with Depot.

### Depot specific inputs

| Name               | Type     | Description                                                                         |
|--------------------|----------|-------------------------------------------------------------------------------------|
| `project`          | String   | Depot [project](https://depot.dev/docs/core-concepts#projects) ID to route the image build to your projects builders             |
| `token`            | String   | You must authenticate with the Depot API to communicate with your projects builders ([see Authentication above](#authentication)) |


### General inputs

The following inputs can be used as `step.with` keys and match the inputs from [`docker/build-push-action`](https://github.com/docker/build-push-action)


| Name               | Type     | Description                                                                                                                                                                        |
|--------------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `add-hosts`        | List/CSV | List of [customs host-to-IP mapping](https://docs.docker.com/engine/reference/commandline/build/#add-entries-to-container-hosts-file---add-host) (e.g., `docker:10.180.0.1`)       |
| `allow`            | List/CSV | List of [extra privileged entitlement](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#allow) (e.g., `network.host,security.insecure`)                 |
| `builder`          | String   | Builder instance (see [setup-buildx](https://github.com/docker/setup-buildx-action) action)                                                                                        |
| `build-args`       | List     | List of [build-time variables](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#build-arg)                                                              |
| `build-contexts`   | List     | List of additional [build contexts](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#build-context) (e.g., `name=path`)                                 |
| `cache-from`       | List     | List of [external cache sources](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#cache-from) (e.g., `type=local,src=path/to/dir`)                      |
| `cache-to`         | List     | List of [cache export destinations](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#cache-to) (e.g., `type=local,dest=path/to/dir`)                    |
| `cgroup-parent`    | String   | Optional [parent cgroup](https://docs.docker.com/engine/reference/commandline/build/#use-a-custom-parent-cgroup---cgroup-parent) for the container used in the build               |
| `context`          | String   | Build's context is the set of files located in the specified [`PATH` or `URL`](https://docs.docker.com/engine/reference/commandline/build/) (default [Git context](#git-context))  |
| `file`             | String   | Path to the Dockerfile. (default `{context}/Dockerfile`)                                                                                                                           |
| `labels`           | List     | List of metadata for an image                                                                                                                                                      |
| `load`             | Bool     | [Load](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#load) is a shorthand for `--output=type=docker` (default `false`)                               |
| `network`          | String   | Set the networking mode for the `RUN` instructions during build                                                                                                                    |
| `no-cache`         | Bool     | Do not use cache when building the image (default `false`)                                                                                                                         |
| `no-cache-filters` | List/CSV | Do not cache specified stages                                                                                                                                                      |
| `outputs`          | List     | List of [output destinations](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#output) (format: `type=local,dest=path`)                                 |
| `platforms`        | List/CSV | List of [target platforms](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#platform) for build                                                         |
| `pull`             | Bool     | Always attempt to pull all referenced images (default `false`)                                                                                                                     |
| `push`             | Bool     | [Push](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#push) is a shorthand for `--output=type=registry` (default `false`)                             |
| `secrets`          | List     | List of [secrets](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#secret) to expose to the build (e.g., `key=string`, `GIT_AUTH_TOKEN=mytoken`)        |
| `secret-files`     | List     | List of [secret files](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#secret) to expose to the build (e.g., `key=filename`, `MY_SECRET=./secret.txt`) |
| `shm-size`         | String   | Size of [`/dev/shm`](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#-size-of-devshm---shm-size) (e.g., `2g`)                                          |
| `ssh`              | List     | List of [SSH agent socket or keys](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#ssh) to expose to the build                                         |
| `tags`             | List/CSV | List of tags                                                                                                                                                                       |
| `target`           | String   | Sets the target stage to build                                                                                                                                                     |
| `ulimit`           | List     | [Ulimit](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md#-set-ulimits---ulimit) options (e.g., `nofile=1024:1024`)                                     |
| `github-token`     | String   | GitHub Token used to authenticate against a repository for [Git context](#git-context) (default `${{ github.token }}`)                                                             |

### Outputs

| Name       | Type    | Description                             |
|------------|---------|-----------------------------------------|
| `imageid`  | String  | Image ID                                |
| `digest`   | String  | Image digest                            |
| `metadata` | JSON    | Build result metadata                   |


## License

MIT License, see `LICENSE`.

Code derived from `docker/build-push-action` copyright 2013-2018 Docker, Inc., Apache License, Version 2.0.
