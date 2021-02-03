# GH Action to Build Docker Images (with some Keptn internal stuff)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/keptn/gh-action-build-docker-image)

This repo provides a utility GitHub action for the [CI Workflow](https://github.com/keptn/keptn/tree/master/.github/workflows) in the [Keptn Project](https://github.com/keptn). Nevertheless, this action provides a very basic functionality, as in "Build a docker image and do some sanity checks".

## Inputs

* `IMAGE_NAME` - name of the image (including the docker organization)
* `DOCKER_FOLDER` (optional, defaults to './') - folder where Dockerfile is in; needs a trailing slash
* `VERSION` - semantic version used as the tag for the resulting image
* `DATETIME` (optional) - datetime string, e.g., YYYYMMDDHHmmss; (defaults to `DATETIME=$(date +'%Y%m%d%H%M')`) 
* `PLATFORMS` (optional) - comma separated list of platforms that the docker image should be build for (e.g., `linux/amd64,linux/arm/v7,linux/arm64`); if set, requires `buildx` to be available (see example below)

## Outputs

None at the moment

## Example usage

### Simple Example

```
    env:
      VERSION: "1.2.3"
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - id: docker_login
        name: Docker Login
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - id: docker_build
        name: Docker Build
        uses: keptn/gh-action-build-docker-image@master
        with:
          PLATFORMS: ${{ env.DOCKER_PLATFORMS }}
          VERSION: ${{ env.VERSION }}
          IMAGE_NAME: "yourdockerorg/yourimagename"
```

### Extended Example using buildx

```
    env:
      VERSION: "1.2.3"
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up QEMU (Docker Multi-Arch Build/BuildX)
        # needed for docker multi-architecture build
        uses: docker/setup-qemu-action@v1

      - name: Set up Docker Buildx (Docker Multi-Arch Build/BuildX)
        # needed for docker multi-architecture build
        uses: docker/setup-buildx-action@v1

      - id: docker_login
        name: Docker Login
        # only run docker login on pushes; also for PRs, but only if this is not a fork
        if: (github.event_name == 'push') || (github.event.pull_request.head.repo.full_name == github.repository)
        # note: GH does not allow to access secrets for PRs from a forked repositories due to security reasons
        # that's fine, but it means we can't push images to dockerhub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - id: docker_build
        name: Docker Build
        uses: keptn/gh-action-build-docker-image@master
        with:
          PLATFORMS: "linux/amd64,linux/arm/v7,linux/arm64"
          VERSION: ${{ env.VERSION }}
          IMAGE_NAME: "yourdockerorg/yourimagename"
          DOCKER_FOLDER: "your-sub-directory/"
```