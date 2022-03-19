# id3tool Docker Image

![build image and push to docker hub workflow](https://github.com/mikeoertli/id3tool-docker/actions/workflows/publish-image-to-docker-hub.yml/badge.svg)

## Overview

Simple lightweight container to run [Ubuntu's `id3tool`](http://manpages.ubuntu.com/manpages/focal/man1/id3tool.1.html) without having to install Ubuntu.

This uses `minideb` and the latest `id3tool` at the time of publication (`v1.2a-11`).

Note: I haven't explored [`buildx`](https://docs.docker.com/desktop/multi-arch/) yet as a multi-arch solution.

## Docker Hub

This project is now on Docker Hub under the same name ([mikeoertli/id3tool](https://hub.docker.com/r/mikeoertli/id3tool))! Instead of needing to clone this repo and build, you can just pull down the pre-built images and run with those.

The `docker pull` command is:

```bash
docker pull mikeoertli/id3tool
```

## What is ID3Tool?

This tool is [part of Ubuntu](http://manpages.ubuntu.com/manpages/focal/man1/id3tool.1.html).

The ARM binary is retrieved from [this launchpad.net resource](https://launchpad.net/ubuntu/focal/arm64/id3tool/1.2a-11).

The AMD binary is retrieved from [this launchpad.net resource](https://launchpad.net/ubuntu/focal/amd64/id3tool/1.2a-11).

## Usage

### Build

The `id3tool` version number is kept in `id3tool-version.txt` so this build command doesn't need to change.

However, the `Dockerfile` is currently manually kept in sync with the `txt` file.

Building ARM:

```bash
docker build --platform linux/arm64 -t mikeoertli/id3tool:"$(cat id3tool-version.txt)_arm64" -t mikeoertli/id3tool:latest .
```

Building AMD:

```bash
docker build --platform linux/amd64 -t mikeoertli/id3tool:"$(cat id3tool-version.txt)_amd64" -t mikeoertli/id3tool:latest -f Dockerfile.amd64 .
```

### Publishing to Docker Hub

These images are published to [Docker Hub under `mikeoertli/id3tool`](https://hub.docker.com/r/mikeoertli/id3tool/tags).

Right now, GitHub actions [only support](https://www.mess.org/2022/01/17/Creating-a-linux-arm64-github-actions-runner/) `amd64` architecture, so for now, those are the only images that are build and published automatically.

#### Publishing ARM64 Images (Manual)

Images for `arm64` architecture must be pushed manually, the command to do that is:

```bash
docker push mikeoertli/id3tool:"$(cat id3tool-version.txt)"_arm64
```

#### Publishing AMD64 Images

Images for `amd64` architecture are pushed when tagging the repo with a tag matching the format of `v*.*-*`.

In addition to automatic publication via GitHub Actions, repos can also be pushed manually, the command to do that is:

```bash
docker push mikeoertli/id3tool:"$(cat id3tool-version.txt)"_amd64
```

### Run

There are a couple important items to note:

1. You will need to provide a file name.
2. As shown below, a [Docker Volume](https://docs.docker.com/storage/volumes/) is created that maps the *current working directory on the host machine* to `/temp` inside the container.
3. The `WORKDIR` is `/temp`, since this is where files are expected, a relative file reference is fine.

#### Run Equivalent of 'id3tool FILE'

```bash
docker run -it --name id3tool --rm -v $(pwd):/temp mikeoertli/id3tool:latest <switches> "<FILE>"
```

#### Passing CLI args

You can still pass command line args, for example, if you want to print the help guide, you can do that with `-h` or `--help` just like you normally would (or you can omit all args and options and the default is to display the `--help` output).

```bash
docker run -it --name id3tool --rm -v $(pwd):/temp mikeoertli/id3tool:latest --help
```

## Tips and Misc. Info

### id3tool man pages

More useful info about options for `id3tool` can be found via [ubuntu.com](http://manpages.ubuntu.com/manpages/focal/man1/id3tool.1.html).

### Auto-remove Container

The `--rm` on the `run` command will auto-delete the container when it stops ([you can read more about this here](https://docs.docker.com/engine/reference/commandline/rm/)). This makes it easier to use this containerized `id3tool` solution in a way more similar to running it natively (this can be helpful when called from a script, too).

### Default Command

The container sets a `CMD` value of `--help`, this serves as the default parameter if none is provided when the container is run. According to the [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#entrypoint):
> The best use for `ENTRYPOINT` is to set the image’s main command, allowing that image to be run as though it was that command (and then use `CMD` as the default flags).

To put it differently, if you don't pass an audio file argument to the `docker run` command, you're effectively performing the equivalent of executing `id3tool --help` from a native command line.

### Drop-in Replacement for id3tool

In order to make this (appear to be) a "true" drop-in replacement for running `id3tool` natively, you could define an alias... something like this:

```bash
alias id3tool='docker run -it --name id3tool --rm -v $(pwd):/temp mikeoertli/id3tool:latest'
```
