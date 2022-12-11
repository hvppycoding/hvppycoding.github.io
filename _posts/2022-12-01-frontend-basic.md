---
title: "웹 프론트엔드 기초"
excerpt: "HTML & CSS & JS"
date: 2022-12-01 23:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Robust Python
---

# 1. Introduction

프론트엔드 개발: HTML, CSS, JS를 사용해 데이터를 GUI로 변환하고, 사용자와 상호 작용할 수 있도록 하는 것

- HTML(Hyper Text Markup Language): 페이지의 제목, 문단 등 웹의 구조 담당
- CSS(Cascading Style Sheets): 실제 화면에 표시되는 방법(색상, 크기 등)을 지정해 컨텐츠를 꾸며주는 표현(정적) 담당
- JS(JavaScript): 콘텐츠를 바꾸고 움직이는 등 동적 처리 담당

#### 웹 표준
웹 표준이란 '웹에서 사용되는 표준 기술이나 규칙'을 의미,
W3C의 표준화 제정 단계의 '권고안(REC)'에 해당하는 기술

#### 크로스 브라우징
브라우저마다 구현 방식 다를 수 있음. 모든 브라우저에서 동일한 사용자 경험을 줄 수 있도록 제작하는 기술을 크로스 브라우징이라고 함

#### 웹 이미지
- **비트맵(Bitmap, Raster)**: 픽셀이 모여 만들어진 이미지(jpeg, gif, png, webp), 정교하고 다양한 색상을 자연스럽게 표현, 확대/축소 시 계단 현상
- **벡터(Vector)**: 점, 선, 면의 위치, 색상 등 수학적 정보 형태로 이루어진 이미지(svg), 정교한 이미지 표현은 어렵지만 확대/축소에서 자유로움

### 특수 문자들

| 기호| 이름                 |
|-----|---------------------|
| \`  | Backtick, Grave     |
| ~   | Tilde, 물결 표시 |
| !   | Exclamation, 느낌표 |
| @   | At sign, 골뱅이 |
| #   | 샵 |
| $   | 달러 |
| %   | 퍼센트 |
| ^   | Caret, 캐럿 |
| &   | Ampersand |
| *   | Asterisk |
| -   | hyphen, dash, 마이너스 |
| _   | underscore, low dash, 밑줄 |
| =   | equal sign |
| "   | Quatation mark, 큰 따옴표 |
| '   | Apostrophe, 작은 따옴표 |
| :   | Colon |
| ;   | Semicolon |
| .   | period, dot |
| ,   | Comman, 쉼표 |
| ?   | Question mark, 물음표 |
| /   | Slash |
| \|   | Vertical bar |
| \\   | Backslash, 역슬래시 |
| ()   | Parenthesis, 괄호 |
| {}   | Brace, 중괄호 |
| []   | Bracket, 대괄호 |
| <>   | Angle Bracket, 꺽쇠괄호 |

### 오픈소스 라이센스
- Apache License: 개인적/상업적 이용, 배포, 수정, 특허 신청이 가능
- MIT License: 개인 소스에 이 라이선스를 사용하고 있다는 표시 필요
- BSD License: MIT와 유사


# 2. VS Code

1. 작업할 폴더 열기
2. index.html 파일 새로 만들기
3. 편집기에 느낌표를 입력하면 자동으로 html 형식을 자동 완성할 수 있다.
4. body 태그 사이에 div를 입력하면 자동으로 div 태그가 완성된다.
5. 코드 자동 indent 기능: 확장 기능 beautify 설치하여 세팅하면 된다.
6. 태그 Pair 자동 변경되게 하기: 확장 기능 Auto Rename Tag 설치
7. 브라우저에 출력하기: 확장 기능 Live Server 설치, 빈공간 혹은 명령 팔레트에서 'Open with Live Server' 실행

### 유용한 단축키

| 단축키 | 설명 |
|-------|------|
| `Ctrl + B` | 사이드 바 토글 |
| `Ctrl + P` | 빠른 열기 |
| `Ctrl + Shift + P` | 모든 명령 표시 |
| `Ctrl + W` | 현재 편집창 닫기 |
| `Ctrl + F` | 찾기 |
| `Ctrl + H` | 찾아 바꾸기 |
| `Alt + 위/아래` | 줄 위치 바꾸기 |
| `Alt + Shift + 위/아래` | 줄 복사하기 |
| `Tab` | 들여쓰기(Tab 공백은 2칸 권장) |
| `Ctrl + PageUp/Down` | 좌/우측 탭으로 전환 |
| `Ctrl + \` | 창 분할 |


# 3. 무작정 시작하기

#### html 구조 설명

```html
<!DOCTYPE html> <!-- Comment: DTD, Document Type Definition, 마크업 언어에서 문서 형식을 정의, 웹 브라우저가 해석 시 사용 -->
<html> <!-- Comment: html 태그는 HTML 문서의 시작과 끝을 나타낸다 -->
  <head>
    <!-- Comment: 문서의 정보를 나타내는 부분, 웹 페이지 제목, 스타일 등의 보이지 않는 정보 작성 -->
  </head>
  <body>
    <!-- Comment: 문서의 구조를 나타내는 범위, 사용자 화면을 통해 보여지는 구조 작성하는 부분 -->
  </body>
</html>
```
 
#### 실습하기

프로젝트 트리 구조는 다음과 같다.

![예시 프로젝트 트리 구조]({{site.baseurl}}/assets/images/2022-12-05-project-tree.png)

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="ko"> <!-- 한글 문서임을 나타낸다. -->
<head>
    <!-- 문자 인코딩 방식, 최근에는 UTF-8이 주로 사용된다. -->
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">

    <!-- meta 정보 선언, name: 정보의 종류, content: 정보의 값 -->
    <!-- 아래는 모바일에서 표시될 때 사용되는 태그로, width와 초기 스케일을 설정 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- html 문서의 제목을 정의 -->
    <title>MY WEB</title>

    <!-- rel: 가져올 문서와의 관계, href: 가져올 문서의 경로 -->
    <link rel="stylesheet" href="./css/main.css">

    <!-- favorite icon(favicon) -->
    <link rel="icon" href="./favicon.png">

    <!-- 스타일(CSS)를 HTML 문서 안에서 작성할 수 있음 -->
    <style>
        div {
            text-decoration: underline;
        }
    </style>

    <!-- JS 파일을 가져오는 경우 -->
    <script src="./js/main.js"></script>

    <!-- HTML 문서 안에서 JS 코드 작성 -->
    <script>
        console.log('Hello world!')
    </script>

</head>
<body>
    <div>Hello world!</div>
    <img src="./images/logo.png" alt="LOGO">
    <img src="https://hvppycoding.github.io/assets/images/icon.png" alt="LOGO">
</body>
</html>
```

```css
/* main.css */
div {
  color: red;
  font-size: 100px;
}
```

```js
// main.js
console.log('HVPPY') // 브라우저의 콘솔에 출력된다.
```

Live Server를 실행하면 다음과 같은 결과 화면을 확인할 수 있다.

![결과]({{site.baseurl}}/assets/images/2022-12-05-result.png)

