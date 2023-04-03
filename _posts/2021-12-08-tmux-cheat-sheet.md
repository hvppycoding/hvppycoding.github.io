---
title: "tmux 사용법"
excerpt: "tmux 사용법"
tagline: "tmux 사용법"
header:
  overlay_image: /assets/images/2021-12-08-terminal.jpg
  overlay_filter: 0.5
  caption: "Photo creadit: [**Unsplash**](https://unsplash.com)"
date: 2021-12-08 23:00:00 +0900
categories:
  - Linux
---

## tmux 사용법

주로 원격 접속 시 세션 관리를 위해 tmux 프로그램을 사용한다.  
putty나 terminus로 원격 접속 시 프로그램이 꺼지면 세션 정보도 사라져서, 새로 접속할 때마다 초기화되는 불편함이 있다.
tmux를 사용하면 기존에 사용하던 세션을 불러올 수도 있고, 또한 동시에 여러가지 작업을 할 때 여러 세션을 사용하면 편하다.

설치 명령어는 `sudo apt install tmux`로 설치할 수 있다.

- 세션 확인: `tmux list-session`, `tmux ls`
- 세션 생성(세션명 자동): `tmux`, `tmux new`
- 세션 생성(세션명): `tmux new-session -s 세션명`, `tmux new -s 세션명`
- 세션 이름 변경: `<Ctrl + b> $`
- 세션 Detach(세션을 살려둔채 벗어남): `<Ctrl + b> d`
- 세션 Attach(기존 세션으로 복귀): `tmux a -t 세션명`, `tmux attach -t 세션명`
- 세션 종료: `tmux kill-session -t 세션명`, `(해당 세션에서)exit`
- History 스크롤하기: `<Ctrl + b + [>`로 스크롤 모드 진입 후 방향키, Page Up/Down으로 스크롤, `q`로 종료

- 세션 내 새 Window 생성: `<Ctrl + b> c`
- Window간 이동: `<Ctrl + b> [0~9 번호]`, `<Ctrl + b> n(next), p(prev), l(Last), w(select), f(find)`
- Window명 변경: `<Ctrl + b> ,`
- Window 종료: `<Ctrl + b> &`, `<Ctrl + d>`

- Pane split(가로): `<Ctrl + b> %`
- Pane split(세로): `<Ctrl + b> "`
- Pane 닫기: `<Ctrl + b> x`
- Pane 이동: `<Ctrl + b> 방향키`

## tmux customize

Self-contained, pretty and versatile .tmux.conf configuration file.

tmux 환경 꾸미기 및 다양한 편의 기능이 탑재된 config 파일이다. 설정 방법 및 Document는 github를 확인하자.

[https://github.com/gpakosz/.tmux](https://github.com/gpakosz/.tmux)

![tmux-config.gif]({{site.url}}/assets/images/tmux-config.gif)

### Features

- visual theme inspired by Powerline
- `<Ctrl + a>`도 기존 `<Ctrl + b>`와 같은 Prefix 역할을 한다. 즉 `<prefix>`는 `<Ctrl + b>` 또는 `<Ctrl + a>`를 사용하면 된다.
- `<prefix> +`: 현재 Pane을 Maximize하여 new window로 열기
- `<prefix> m`: mouse mode 토글(마우스로 pane 크기 조절 및 focus 가능)
- (?)automatic usage of reattach-to-user-namespace if available

### Bindings

- `<prefix> <Ctrl + h>`와 `<prefix> <Ctrl + l>`로 window 이동 가능

- `<prefix> e` opens the `.local` customization file copy with the editor
   defined by the `$EDITOR` environment variable (defaults to `vim` when empty)
- `<prefix> r` reloads the configuration
- `C-l` clears both the screen and the tmux history

- `<prefix> C-c` creates a new session
- `<prefix> C-f` lets you switch to another session by name

- `<prefix> C-h` and `<prefix> C-l` let you navigate windows (default
  `<prefix> n` and `<prefix> p` are unbound)
- `<prefix> Tab` brings you to the last active window

- `<prefix> -` splits the current pane vertically
- `<prefix> _` splits the current pane horizontally
- `<prefix> h`, `<prefix> j`, `<prefix> k` and `<prefix> l` let you navigate
   panes ala Vim
- `<prefix> H`, `<prefix> J`, `<prefix> K`, `<prefix> L` let you esize panes
- `<prefix> <` and `<prefix> >` let you swap panes
- `<prefix> +` maximizes the current pane to a new window

- `<prefix> m` toggles mouse mode on or off

- `<prefix> U` launches Urlview (if available)
- `<prefix> F` launches Facebook PathPicker (if available)

- `<prefix> Enter` enters copy-mode
- `<prefix> b` lists the paste-buffers
- `<prefix> p` pastes from the top paste-buffer
- `<prefix> P` lets you choose the paste-buffer to paste from

Additionally, `copy-mode-vi` matches [my own Vim configuration][]

[my own Vim configuration]: https://github.com/gpakosz/.vim.git

Bindings for `copy-mode-vi`:

- `v` begins selection / visual mode
- `C-v` toggles between blockwise visual mode and visual mode
- `H` jumps to the start of line
- `L` jumps to the end of line
- `y` copies the selection to the top paste-buffer
- `Escape` cancels the current operation