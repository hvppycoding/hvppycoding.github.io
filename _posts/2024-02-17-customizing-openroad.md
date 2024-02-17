---
title: " Customizing OpenROAD"
excerpt: "Customizing OpenROAD"
date: 2023-10-21 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## 1. Create docker images

Follow instructions: [Build from sources using Docker](https://openroad-flow-scripts.readthedocs.io/en/latest/user/BuildWithDocker.html)  

Then, docker images named `openroad/flow-centos7-dev` and `openroad/flow-centos7-build` will be created.  

We will start from `openroad/flow-centos7-dev`

## 2. Start a docker container for development

```sh
docker run -it \
           -e DISPLAY=${DISPLAY} \
           -v /tmp/.X11-unix:/tmp/.X11-unix \
           -v ${HOME}/.Xauthority:/.Xauthority \
           --network host \
           --security-opt seccomp=unconfined \
           openroad/flow-centos7-dev
```

## 3. Clone your source code

```sh
cd /
git clone --recursive -b final https://github.com/hvppycoding/OpenROAD-flow-scripts.git
```

## 4. Build

```sh
cd OpenROAD-flow-scripts
source setup.sh
./build_openroad.sh --local
```

## 5. Run

```sh
source env.sh
openroad
```