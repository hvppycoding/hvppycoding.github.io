---
title: "Ubuntu Troubleshooting: Tips for Korean"
excerpt: ""
date: 2024-02-07 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Tools
mathjax: "true"
---

## Nvidia driver로 인해 부팅 안 될 때

처음 Ubuntu 설치 후 Nvidia driver 설치가 되지 않아, 부팅이 안 될 때가 있다.  
이 경우 Mainboard 제조사 로고가 뜨는 화면에서 `<shift>` 키를 꾹 누르고 있으면, Grub menu를 켤 수 있다.  
여기서 Advanced options for Ubuntu  recovery mode로 부팅이 가능하다.  
Recovery mode 메뉴에서 dpkg, network, root를 순서대로 실행하였다(그림 참조).  
root를 실행하면 커맨드라인을 사용할 수 있는데, 여기서 최신 버전의 nvidia driver를 설치하니 해결되었다.  

! [recovery mode]({{site.baseurl}}/assets/images/ubuntu-recovery-menu.png)  

## Right Alt 기능 삭제

한/영 변환 키를 오른쪽 Alt 키로 설정 시, Alt의 기능도 남아있어 불편할 때가 있다.  
ALT키의 역할을 변경시키자.

`sudo vi /usr/share/X11/xkb/keycodes/evdev`

이를 열면 기존 세팅은 다음과 같을 것이다.

```
...
<RALT> = 108;
...
<HNGL> = 130;
...
```

이를 아래와 같이 변화시키자. (//로 주석처리)

```
...
// <RALT> = 108;
...
<HNGL> = 108;
...
```