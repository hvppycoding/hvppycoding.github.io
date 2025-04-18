---
title: "Missing Semester 03 - Editors (Vim)"
excerpt: ""
date: 2024-05-09 15:03:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Missing Semester
---

{% include video id="a6Q8Na575qc" provider="youtube" %}

코드를 작성하는 일은 글을 쓰는 일과 아주 다릅니다. 프로그래밍은 긴 줄글을 쓰는 것과 다르게, 파일을 바꾸거나 코드를 읽고 처리하고 작성하는데 더 많은 시간을 할애하게 됩니다. 그렇기에 서로 다른 종류의 글을 작성하는 프로그램과 코드를 작성하는 프로그램이 있습니다(가령, Microsoft Word와 Visual Studio Code를 비교할 수 있겠지요).

프로그래머는 코드를 작성하는데 대부분의 시간을 보내기 때문에, 자신에게 맞는 에디터를 숙달하는데 사용하는 시간은 그 자체로 가치가 있습니다. 새로운 에디터를 배우는 방법은 다음과 같습니다:

- 튜토리얼을 시작하기 (즉, 이번 강의를 학습하고, 추가로 강의에서 제공한 리소스를 활용해보세요)
- (비록 처음에는 많은 시간이 걸리더라도) 에디터를 사용하여 끈기있게 텍스트를 작성하기
- 진행에 필요한 기능을 찾아보기: 만약 무언가를 하기 위해 더 나은 방법을 찾아야 한다면, 여러분은 아마도 그것을 찾을 수 있을 것입니다

만약 여러분이 이러한 방법을 따라서 모든 텍스트 편집을 위한 새로운 프로그램을 사용하는데 헌신하고자 한다면, 텍스트 에디터를 보다 정교하게 학습하기 위한 타임라인은 다음과 같습니다. 한시간 내지 두시간 정도, 여러분은 파일을 열고 작성하거나, 저장하고 종료하거나 그리고 버퍼를 다루는 기본적인 에디터 기능들을 배워야 할 것입니다. 20시간 정도 학습하게 된다면, 여러분은 이전에 사용하였던 에디터만큼 빠르게 새로운 에디터를 사용하게 될 것입니다. 이 때부터 새로운 가능성이 열리게 됩니다. 여러분은 새로운 에디터를 사용하면서도 시간을 절약할 수 있는 충분한 지식과 머슬 메모리를 가지게 될 것입니다. 현대 텍스트 에디터들은 화려하고 강력하기에, 그것을 배우는 일은 끝이 없지만, 여러분은 더 배우는 만큼 더욱 빠르게 에디터를 사용할 수 있을 것입니다.

# 어떤 에디터를 배워야 할까요?

프로그래머들은 각자 자신의 텍스트 에디터에 대해서 [강력한 견해](https://en.wikipedia.org/wiki/Editor_war)를 가지고 있습니다.

오늘 날에는 어떤 에디터가 유명할까요? 다음의 설문을 확인해 볼 수 있습니다: [Stack Overflow
survey](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools) (하지만 Stack Overflow 유저들이 프로그래머 전체를 대변하지 않기에, 약간의 편견이 개입되어 있을 수는 있지만요). 그 중 [Visual Studio Code](https://code.visualstudio.com/)를 가장 유명한 에디터라고 말할 수 있습니다. 그리고 [Vim](https://www.vim.org/)은 가장 유명한 커맨드-라인-기반(command-line-based) 에디터입니다.

## Vim

해당 강의의 모든 교사들은 Vim을 에디터로 사용하고 있습니다. Vim은 Vi 에디터(1976)에서 비롯된 풍부한 역사를 가지고 있으며, 오늘날에도 여전히 개발되고 있습니다. Vim은 정말로 멋진 아이디어로 구성된 에디터이기에, 많은 도구들이 Vim 에뮬레이션 모드를 지원하고 있습니다 (예를 들어, 1.4만명의 사람들이 [Vim emulation for VS code](https://github.com/VSCodeVim/Vim))를 설치했습니다). 만약 여러분이 결국 다른 텍스트 에디터를 사용하기로 마음을 바꾸더라도, Vim은 충분히 배울 가치가 있는 에디터입니다.

Vim의 모든 기능을 50분 안에 다 설명할 수는 없기 때문에, 이번 강의에선 여러분께 Vim의 철학을 설명하고, 기본을 알려드리며, 보다 심화된 기능을 보여드리고, 도구를 숙달할 수 있는 리소스를 제공하는 데 주력할 것입니다.

# Vim의 철학

여러분은 프로그래밍에서 대부분의 시간을 쓰기가 아닌 읽기와 편집에 할애할 것입니다. 이러한 이유로 Vim은 별개로 텍스트 입력을 위한 모드와 텍스트 처리를 위한 모드를 가지는 _다중 모드(modal)_ 에디터로 만들어졌습니다. Vim은 Vimscript와 Python과 같은 다른 언어를 사용하여 프로그래밍할 수 있으며, 연상을 돕는 언어로 구성된 키 조합을 명령어로 삼는 Vim의 인터페이스 자체도 프로그래밍 언어입니다. Vim은 마우스가 너무 느리다고 생각하기에 마우스를 사용하지 않으며, 화살표 키 또한 너무 많은 이동이 필요하기 때문에 사용하지 않습니다.

Vim의 최종 목표는 여러분이 생각하는 바를 곧바로 실행할 수 있는 빠른 속도의 에디터가 되는 것입니다.

# 다중 모드 편집

Vim의 디자인은 많은 프로그래머들이 긴 텍스트 문장을 쓰는데 시간을 소비하는 것이 아니라, 읽고 탐색하고 작은 편집을 하는데 시간을 소비한다는 생각에 바탕을 두고 있습니다. 이러한 이유로 Vim에는 여러 가지 운용 모드를 가지고 있습니다.

- **Normal**: 파일을 탐색하고 편집하는 모드
- **Insert**: 텍스트 입력 모드
- **Replace**: 텍스트 변경 모드
- **Visual** (텍스트 일부, 라인 또는 블럭): 텍스트의 블럭을 선택하는 모드
- **Command-line**: 명령어를 실행하는 모드

운용 모드에 따라 키 입력 또한 다른 의미를 가지게 됩니다. 예를 들어, 입력(Insert) 모드의 문자 `x`는 문자 'x'가 입력되는 것으로 끝나지만, 일반(Normal) 모드에서는 커서 아래의 문자가 삭제되고, 비주얼(Visual) 모드에서는 선택된 항목이 삭제됩니다.

Vim은 기본 설정으로 왼쪽 하단에 현재 모드를 표시합니다. 초기/기본 모드는 일반 모드입니다. 여러분은 보통 일반 모드와 입력 모드에서 대부분의 시간을 보내게 될 것입니다.

어떠한 모드에서 일반 모드로 전환하기 위해선 `<ESC>`(이스케이프 키)를 누르면 됩니다. 반대로 일반 모드에서 입력 모드로 들어가기 위해선 `i`키를 누르면 되고, 변경(Replace) 모드로 들어가기 위해선 `R`키를, 비주얼 모드는 `v`키, 비주얼 라인 모드는 `V`키, 비주얼 블럭 모드는 `<C-v>`(Ctrl-V 또는 `^V`)를, 그리고 명령어(Command-line) 모드는 `:`키를 누르면 해당 모드로 들어가게 됩니다.

여러분은 Vim을 사용하면서 `<ESC>`를 많이 사용하게 될 것입니다. 그렇기에 이스케이프 키를 캡스 락(Caps Lock)으로 변경하는 방안을 고려해보는 것도 좋습니다. ([macOS instructions](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)).

# 기초

## 텍스트 입력

일반 모드에서 `i`키를 눌러 입력 모드로 들어가는 순간, Vim은 여러분이 `<ESC>`키를 눌러 일반 모드로 돌아가기 전까지 여다른 텍스트 에디터처럼 작동하게 됩니다. 이제 여러분은 위에서 설명했던 기본사항과 함께 Vim을 사용하여 파일 편집을 시작하면 됩니다 (만약 모든 활동을 입력 모드에서 처리하고자 한다면, 특별히 효율적인 결과가 나오지는 않겠지만요).

## 버퍼, 탭 그리고 창

Vim은 "버퍼(buffers)"라고 불리는 열린 파일의 집합을 유지하고 있습니다. Vim 세션에는 여러 개의 탭(tab)이 있으며, 각 탭에는 여러 개의 창(window, 분할 창, split panes)이 있습니다. 각 창에는 하나의 버퍼가 표시됩니다. 웹 브라우저와 같이 여러분에게 익숙한 다른 프로그램들과 달리, 창은 단순히 보여주는 역할을 하기 때문에, 버퍼와 창은 일대일 대응을 하지 않습니다. 동일한 탭 내에서도 하나의 지정된 버퍼를 _여러 개의_ 창에서 열 수 있습니다. 예를 들어, 파일의 서로 다른 두 부분을 볼 수 있는데, 이는 꽤 편리한 기능입니다.

기본적으로, Vim은 하나의 창을 포함하는 하나의 탭을 엽니다.

## 커맨드-라인

명령어 모드는 일반 모드에서 `:`키를 입력하여 들어갈 수 있습니다. `:`키를 누르면 여러분의 커서가 화면 하단에 있는 커맨드 라인으로 이동하게 됩니다. 이 모드에는 파일 열기, 저장하기, 파일 닫기 그리고 [Vim 종료하기](https://twitter.com/iamdevloper/status/435555976687923200)와 같은 다양한 기능이 있습니다.

- `:q` - 종료 (창 닫기)
- `:w` - 저장 ("write"의 앞 글자를 따옴)
- `:wq` - 저장한 뒤 종료
- `:e {파일의 이름}` - 편집을 위해 파일을 열기
- `:ls` - 열려 있는 버퍼들을 보여주기
- `:help {주제}` - 도움말 열기
    - `:help :w` - 명령어 `:w`에 대한 도움말 열기
    - `:help w` - `w`의 동작에 대한 도움말 열기

# Vim의 인터페이스는 프로그래밍 언어입니다

Vim에서 가장 중요한 아이디어는 Vim의 인터페이스 자체가 프로그래밍 언어라는 사실입니다. 연상을 돕는 언어로 구성된 키 입력은 명령어로 작동하고, 이러한 명령어들로 명령을 _구성_ 할 수 있습니다. 이렇게 하면 효율적인 동작(movement)과 편집이 가능하며, 특히 명령어가 머슬 메모리에 각인된다면 더욱 그러할 것입니다.

## 동작

여러분은 대부분의 시간을 일반 모드에서 동작 명령어를 사용하여 버퍼를 탐색하는데 보내게 될 것입니다. Vim의 동작은 텍스트 덩어리를 나타내기 때문에 "명사"라고도 불립니다.

- 기본적인 동작: `hjkl` (좌, 우, 위, 아래 움직임)
- 단어: `w` (다음 단어: next word), `b` (단어의 시작: beginning of word), `e` (단어의 끝: end of word)
- 줄: `0` (줄의 시작), `^` (공백이 아닌 첫 문자로 구성된 줄), `$` (줄의 끝)
- 화면: `H` (화면 상단), `M` (화면 중간), `L` (화면 하단)
- 스크롤: `Ctrl-u` (위로 스크롤), `Ctrl-d` (아래로 스크롤)
- 파일: `gg` (파일의 시작지점), `G` (파일의 끝)
- 줄 번호: `:{숫자}<CR>` 또는 `{숫자}G` ({숫자}번째 줄로 이동)
- 잡다한 기능: `%` (괄호의 짝으로 커서 이동)
- 찾기: `f{문자}`, `t{문자}`, `F{문자}`, `T{문자}`
    - 전방향/후방향으로 {문자}의 앞/뒤 위치로 이동
    - `,` / `;` 로 해당하는 문자를 탐색
- 검색: `/{regex}`, `n` / `N` 으로 해당하는 단어를 탐색

## 선택

비주얼 모드들:

- 비주얼
- 비주얼 라인
- 비주얼 블럭

동작 키들을 사용하여 선택 범위를 지정할 수 있습니다.

## 편집

여러분은 지금까지 마우스로 해왔던 모든 작업들을 이제 키보드에서 동작 명령어로 구성된 편집 명령어를 사용하여 수행할 수 있습니다. 그리고 이 지점이 Vim의 인터페이스가 프로그래밍 언어처럼 보이는 이유이기도 합니다. Vim의 편집 명령어는 동사가 명사에 작용하기 때문에, "동사"라고도 불립니다.

- `i` 입력 모드로 전환
    - 백스페이스 이상의 텍스트 편집과 수정이 필요할 경우 사용
- `o` / `O` 현재 라인 아래/위에 새로운 라인 삽입
- `d{동작}` 해당 {동작}을 바탕으로 삭제
    - 예를 들어, `dw`는 단어를 삭제하고, `d$`는 선택된 지점에서 해당 라인의 끝까지 삭제하고, `d0`는 선택된 지점에서 해당 라인의 처음까지 삭제
- `c{동작}` 해당 {동작}을 바탕으로 변경
    - 예를 들어, `cw`는 단어를 변경
    - `d{motion}` + `i`와 같은 기능
- `x` 해당 문자 삭제 (`dl`과 동일한 기능)
- `s` 해당 문자 치환 (`xi`과 동일한 기능)
- 비주얼 모드 + 조작(manipulation)
    - 텍스트를 선택한 후, `d`로 삭제하거나 `c`로 변경
- `u` 해당 작업 되돌리기, `<C-r>` 해당 작업 다시 하기
- `y` 복사 / "yank" (명령어 `d`도 복사의 기능을 가짐)
- `p` 붙혀넣기
- 그 외 수많은 명령어들: 예를 들어 `~`는 해당 문자의 대소문자를 변경

## 횟수

여러분은 명사와 동사를 횟수와 결합하여 주어진 행동을 여러 번 수행할 수 있습니다.

- `3w` 세 단어 앞으로 이동
- `5j` 다섯 라인 아래로 이동
- `7dw` 일곱 단어 삭제

## 수정키

여러분은 명사의 의미를 변경하기 위해 수정키(modifier)들을 이용할 수 있습니다. 가령 `i` 수정키는 "내부" 또는 "안쪽"을 의미하고, `a` 수정키는 "바깥 둘레"를 의미합니다.

- `ci(` 괄호 내부의 요소를 변경
- `ci[` 꺾쇠괄호 내부의 요소를 변경
- `da'` 싱글 쿼테이션 마크를 포함하여 요소 삭제

# Demo

다음과 같은 실수 투성이의 [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) 구현이 있습니다:

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10)
```

우리는 다음의 문제들을 해결해야 할 것입니다:

- Main 함수가 호출되지 않음
- 1이 아닌 0에서 시작함
- 15의 배수에서 "fizz"와 "buzz"를 동시에 출력하지 않음
- 5의 배수에서 "fizz"를 출력함
- 커맨드-라인에서 인수를 받지 않고 하드-코드로 숫자 10을 사용함

{% comment %}
- Main 함수가 호출되지 않음
  - `G` 파일의 끝으로 이동
  - `o`하단에 새로운 라인 삽입
  - "if __name__ ..." 과 같은 내용을 입력
- 1이 아닌 0에서 시작함
  - `/range` 원하는 지점 검색
  - `ww` 두 단어 앞으로 이동
  - `i` 다음의 텍스트를 입력: "1, "
  - `ea` "limit" 텍스트 끝에 다음의 텍스트를 입력: "+1"
- "fizzbuzz" 생성
  - `jj$i` 라인의 끝에 텍스트 입력
  - 다음의 텍스트를 추가: ", end=''"
  - `jj.` 두 번째 출력을 위한 반복
  - `jjo` if문 아래에 새로운 라인 삽입
  - 다음의 텍스트를 추가: "else: print()"
- fizz fizz
  - `ci'` fizz 변경
- 커맨드-라인 인수
  - `ggO` 파일의 시작 지점에 새로운 라인 삽입
  - "import sys"
  - `/10`
  - `ci(` "int(sys.argv[1])" 수정
{% endcomment %}

보다 자세한 설명과 시연에 대해선 강의 비디오를 참조하시길 바랍니다. Vim을 사용하여 위의 변경을 수행하는 방법과 다른 프로그램을 사용하여 동일한 편집을 진행하는 방법을 비교해보세요. Vim에선 아주 적은 키 입력으로 여러분이 원하는 속도로 편집할 수 있습니다.

# Vim 커스터마이징하기

Vim은 Vimscript 명령을 포함하는 `~/.vimrc` 일반-텍스트 구성 파일을 통해 커스터마이징을 할 수 있습니다. 여러분들은 각자 사용하고자 하는 수많은 기본적인 설정들이 있을 것입니다.

우리는 여러분이 출발점으로 사용할 수 있는 문서화된 기본 구성을 제공하고 있습니다. 이들은 Vim의 특별한 기본 동작 중 일부를 수정하기 때문에, 저희가 제공하는 기본 구성을 사용하는 것을 추천드립니다. **[여기]({{site.baseurl}}/assets/resources/2024-05-09-vimrc.txt)에서 저희의 구성을 다운로드하고 `~/.vimrc`에 저장하세요.**

Vim은 강력한 커스터마이징 기능을 제공하기 때문에, 커스터마이징 옵션을 찾아보는 시간은 그 자체로 유익할 것입니다. 여러분은 다른 사람들이 깃허브에 올려놓은 닷파일(dotfile)을 살펴보며 영감을 얻을 수도 있을 것입니다. 예를 들어, 해당 강의의 교사의 Vim 구성은 다음과 같습니다.
([Anish]({{site.baseurl}}/assets/resources/2024-05-09-anish-vimrc.txt),
[Jon]({{site.baseurl}}/assets/resources/2024-05-09-jon-init_lua.txt) (uses [neovim](https://neovim.io/)),
[Jose]({{site.baseurl}}/assets/resources/2024-05-09-jose-vimrc.txt))
이 주제에 대한 좋은 블로그 게시물들도 많이 있습니다. 하지만 다른 사람의 전체 구성을 복사하여 붙여넣지 말고, 읽고 이해한 후 필요한 내용을 가져가시길 바랍니다.

# Vim 확장하기

Vim을 확장하기 위한 플러그인은 아주 많습니다. 여러분이 인터넷에서 볼 수 있는 오래된 조언과는 달리, 여러분은 Vim 8.0 이후로는 Vim용 플러그인 매니저를 사용할 필요가 _없습니다_. 그 대신, 여러분은 내장된 패키지 관리 시스템을 이용할 수 있습니다. 단순히 `~/.vim/pack/vendor/start/` 디렉터리를 생성하고, 그 안에 가령 `git clone`을 통해서 플러그인을 넣으면 됩니다.

저희가 좋아하는 플러그인 목록을 나열해보겠습니다:

- [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): 퍼지(fuzzy) 파일 파인더
- [ack.vim](https://github.com/mileszs/ack.vim): 코드 검색
- [nerdtree](https://github.com/scrooloose/nerdtree): 파일 탐색기
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): 매직 모션

우리는 여기에 엄청나게 긴 플러그인 목록을 제공하고 싶지는 않습니다. 해당 강의의 교사들이 어떤 플러그인을 사용하는지 닷파일을 확인해보는걸 추천합니다.
([Anish](https://github.com/hvppycoding/anishathalye-dotfiles),
[Jon](https://github.com/hvppycoding/jonhoo-configs),
[Jose](https://github.com/hvppycoding/jose-dotfiles))
[Vim Awesome](https://vimawesome.com/)에서 멋진 Vim 플러그인들을 찾아보세요. 또한 단순히 "best Vim plugins"를 검색하기만 해도 이 주제에 대하여 아주 많은 블로그 글들이 있습니다.

# 다른 프로그램에서 Vim-mode 사용하기

많은 도구들이 Vim 에뮬레이션을 지원합니다. 하지만 품질은 도구마다 다르고, 도구에 따라 더 고급스러운 Vim 기능을 지원하지 않을 수도 있지만, 대부분은 기본이 잘 마련되어 있습니다.

## 쉘

Bash 사용자인 경우, `set -o vi`를 사용하면 됩니다. Zsh를 사용할 경우, `bindkey -v`를 사용하면 되고, Fish는 `fish_vi_key_bindings`를 사용하면 됩니다. 또한 여러분이 어떤 쉘을 사용하든 `export EDITOR=vim`을 사용할 수 있습니다. 이는 프로그램이 에디터를 시작할 때 어떤 에디터를 실행해야 할 지 결정하는 데 사용되는 환경 변수입니다. 예를 들어, `git`은 커밋 메시지에 이 에디터를 사용합니다.

## Readline

많은 프로그램들은 커맨드-라인 인터페이스에 [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) 라이브러리를 사용합니다. Readline은 `~/.inputrc` 파일에 다음의 라인을 추가하여 활성화할 수 있는 기본 Vim 에뮬레이션도 지원합니다:

```
set editing-mode vi
```

이러한 세팅을 통해서, 예를 들어, Python REPL은 Vim 바인딩을 지원하게 됩니다.

## 그 밖의 것들

웹 [브라우저](http://vim.wikia.com/wiki/Vim_key_bindings_for_web_browsers)를 위한 Vim 키바인딩 익스텐션도 있습니다. [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en)은 유명한 크롬 전용 익스텐션이고 [Tridactyl](https://github.com/tridactyl/tridactyl)은 파이어폭스 전용 익스텐션입니다. 여러분은 [Jupyter notebooks](https://github.com/lambdalisue/jupyter-vim-binding)의 Vim 바인딩을 확인할 수도 있습니다.

# Vim 심화

다음은 Vim 에디터의 강력한 힘을 보여주는 몇 가지 예입니다. 저희가 이런 것들을 모두 알려드릴 수는 없지만, 앞으로 여러분이 꾸준히 학습해나가게 된다면 충분히 배울 수 있는 내용이라고 생각합니다. 이를 위한 한 가지 좋은 휴리스틱이 있습니다. 여러분이 에디터를 사용할 때, "더 나은 방법이 있을 거야"라고 생각할 때마다, 아마도 온라인에는 여러분이 찾고자 하는 방법이 있을 것입니다.

## 검색과 변환

`:s` (substitute, 대치) 명령어 ([documentation](http://vim.wikia.com/wiki/Search_and_replace)).

- `%s/foo/bar/g`
    - 파일 내부에서 모든 foo를 bar로 대체합니다
- `%s/\[.*\](\(.*\))/\1/g`
    - 마크다운 링크를 일반 URL로 바꿉니다

## 다중 창

- `:sp` / `:vsp` 창 나누기
- 동일한 버퍼에서 다중의 뷰를 가능하게 합니다

## 매크로

- `q{문자}` - 매크로를 `{문자}`와 함께 기록하기 시작
- `q` - 기록 중지
- `@{문자}` - 매크로 반복
- 에러 발생 시 매크로 실행이 중단됨
- `{숫자}@{문자}` - 매크로를 {숫자}만큼 반복하여 실행
- 매크로는 재귀적일 수 있음
    - 먼저 `q{문자}q`로 매크로를 초기화
    - `@{문자}`를 사용하여 매크로를 기록하면, 매크로가 재귀적으로 호출됨
    (기록이 끝날 때까지 작동하지 않을 수 있음)
- 예제: xml을 json으로 변환하기 ([file]({{site.baseurl}}/assets/resources/2024-05-09-example-data.xml))
    - "name" / "email"이라는 키를 가진 객체의 배열
    - Python을 사용해야만 할까?
    - sed / regexes로 처리하기
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - ...
    - Vim 명령어 / 매크로로 처리하기
        - `Gdd`, `ggdd` 첫번째 라인과 마지막 라인을 삭제
        - 단일한 요소의 포맷을 만드는 매크로 (`e`를 등록한다)
            - `<name>` 라인으로 이동
            - `qe^r"f>s": "<ESC>f<C"<ESC>q`
        - 사람의 포맷을 만드는 매크로
            - `<person>` 라인으로 이동
            - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        - 사람의 포맷을 만들고 다음 사람으로 이동하는 매크로
            - `<person>` 라인으로 이동
            - `qq@pjq`
        - 파일이 끝날 때까지 매크로 실행
            - `999@q`
        - 마지막 `,`을 직접 제거하고 `[` and `]` 구분 기호 추가

# 리소스

- `vimtutor`는 Vim과 함께 설치되는 튜토리얼입니다. 즉, 만약 Vim이 설치되었다면, 여러분은 쉘에서 `vimtutor`를 실행할 수 있어야 합니다
- [Vim Adventures](https://vim-adventures.com/)는 Vim을 배우기 위한 게임입니다
- [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/)에는 다양한 Vim 팁들이 있습니다
- [Vim Golf](http://www.vimgolf.com/) 는 [code golf](https://en.wikipedia.org/wiki/Code_golf)의 일종이지만, 프로그래밍 언어가 Vim의 UI로 되어있습니다
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/book/dnvim2/practical-vim-second-edition) (서적)

# 연습문제

1. `vimtutor`를 완료하세요. 참고: Vimtutor는 [80x24](https://en.wikipedia.org/wiki/VT100) (80 columns by 24 lines) 터미널 창에서 가장 잘 보입니다.
1. 저희의 [basic vimrc]({{site.baseurl}}/assets/resources/2024-05-09-vimrc.txt)를 다운로드하고 `~/.vimrc`에 저장하세요. Vim을 사용하여 주석이 달린 파일을 읽고, Vim이 새로운 구성과 어떤 점이 다른지 확인해보세요.
1. 다음의 플러그인을 설치하고 설정합니다: [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).
    1. `mkdir -p ~/.vim/pack/vendor/start`로 플러그인 디렉터리를 생성하세요
    1. 플러그인을 다운로드하세요: `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`
    1. 플러그인의 [documentation](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md)을 읽으세요. CtrlP로 프로젝트 디렉토리를 탐색하여 파일을 찾고, Vim을 연 다음, Vim 커맨드-라인을 사용하여 `:CtrlP`를 시작하세요.
    1. [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options)을 여러분의 `~/.vimrc`에 추가하여 CtrlP를 Ctrl-P를 누름으로써 열 수 있게 커스터마이징 해보세요.
1. Vim 사용법을 연습하기 위해, 강의해서 진행되었던 [Demo](#demo)를 여러분의 컴퓨터로 다시 반복해보세요.
1. 다음 달 까지 _모든_ 텍스트 편집에 Vim을 사용해보세요. 뭔가 비효율적으로 느껴지거나, "더 나은 방법이 있을 거야"라고 생각될 때마다, 구글을 검색해보면, 아마 방법은 있을 것입니다. 만약 막히는 부분이 있다면 오피스 아워에 방문하거나 우리에게 이메일을 보내주세요.
1. Vim 바인딩을 이용하여 여러분의 다른 도구들을 설정해보세요 (관련 설명은 위에서 설명했습니다)
1. (심화) Vim 매크로를 사용하여 XML을 JSON으로 변환해보세요 ([example file]({{site.baseurl}}/assets/resources/2024-05-09-example-data.xml)). 여러분 혼자 힘으로 해야하지만, 만약 막히는 부분이 있다면, 위의 [매크로](#매크로) 섹션을 참고하세요.