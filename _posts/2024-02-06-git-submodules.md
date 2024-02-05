---
title: "Git Submodules"
excerpt: ""
date: 2024-02-06 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Tools
mathjax: "true"
---

## What?

```text
# git-sub-module tutorial
├ README.md
└ youtube-tutorials (@abc123)

# youtube-tutorials (@abc123)
├ README.md
├ tutorial1
├ tutorial2
└ tutorial3
```

## Why / When?

- Sharing Libraries

## How?

### 1. Add Submodule

```sh
cd git-sub-module
git submodule add https://github.com/redhwannacef/youtube-tutorials
```

Submodule을 추가하면 submodule 폴더 내부에 `.gitmodules` 파일이 생성되며, 이 파일에 submodule의 정보가 저장된다.

### 2. Clone Repository with Submodule

```sh
git clone https://github.com/redhwannacef/youtube-tutorials
cd youtube-tutorials
git submodule update --init --recursive
```

혹은  

```sh
git clone --recursive https://github.com/redhwannacef/youtube-tutorials
```

### 3. Git Submodule checkout

처음 clone을 하면 submodule의 branch는 detached 상태이다.  
이 상태에서 파일을 수정하
이를 master branch로 변경 및 최신화하려면 다음과 같이 한다.

```sh
git pull
git submodule foreach --recursive git checkout master
git submodule foreach --recursive git pull
```
