---
title: "VS Code에서 Github Pages 이미지 쉽게 넣기"
excerpt: ""
date: 2024-07-28 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Github Pages
---

## Paste Image Extension 설치하기

VS Code Extension 중 `Paste Image` 설치한다. [Link to Paste Image Extension](https://marketplace.visualstudio.com/items?itemName=mushan.vscode-paste-image)  

## Paste Image Extension 설정하기

`Shift+Ctrl+P`로 팔레트 연 후 `Preferences: Open User Settings` 열어 json 파일 편집한다.  
열어진 json 파일에 아래 내용을 추가하자.  

{% raw %}

```json
"pasteImage.path": "${projectRoot}/assets/images/${currentFileNameWithoutExt}",
"pasteImage.basePath": "${projectRoot}",
"pasteImage.defaultName": "YMMDD-HHmmss",
"pasteImage.showFilePathConfirmInputBox": true,
"pasteImage.prefix": "{{site.baseurl}}/",
```

{% endraw %}

각 설정 내용의 의미는 다음과 같다. 자세한 내용은 [Paste Image Extension](https://marketplace.visualstudio.com/items?itemName=mushan.vscode-paste-image)에서 확인할 수 있다.  
나의 Github Page 프로젝트 구성은 다음과 같으니 설정 시 참고하자.  

- post 경로: `{프로젝트 루트}/_posts/2024-07-28-포스트이름.md`
- image 경로: `{프로젝트 루트}/assets/images/2024-07-28-포스트이름/이미지파일명`

1. `baseImage.path`: 이미지 파일이 저장되는 폴더로, `{프로젝트 루트}/assets/images/{현재 Markdown 파일명}/`이다. 폴더가 없으면 자동으로 생성된다.
2. `baseImage.basePath`: 이미지의 url을 자동으로 생성하여 markdown 파일에 넣어주는데, 이 때 url의 상대 경로 기준점이 된다. 앞의 설정대로 `${projectRoot}`로 이 값을 설정하면 `assets/images/{현재 Markdown 파일명}/{이미지 파일 이름}`로 url이 생성된다.
3. `baseImage.defaultName`: 이미지 파일명을 자동으로 생성할 때 사용하는 기본 이름이다. 기본적으로 시간을 기반으로 자동으로 정해지며, 바꾸고 싶다면 input box 팝업창에서 직접 입력할 수 있다.
4. `pasteImage.showFilePathConfirmInputBox`: 이미지 파일을 붙여넣을 때 팝업창을 띄워서 이미지 파일 경로를 확인할 수 있다.
5. `pasteImage.prefix`: 이미지 url을 붙여넣을 때 이미지 파일 경로 앞에 붙는 prefix이다. 우리의 최종 목표는 {% raw %}`![]({{site.baseurl}}/assets/images/{현재 Markdown 파일명}/{이미지 파일명})`{% endraw %}과 같은 형태가 되는 것이 목표이므로 `{{site.baseurl}}/`를 prefix를 입력한다.

## 이미지 쉽게 붙여넣기

이미지 파일을 `Ctrl+Alt+V`로 붙여넣으면 팝업창이 뜨면서 이미지 파일 경로를 확인할 수 있다.(`pasteImage.showFilePathConfirmInputBox`가 `true`로 설정했을 때)  
기본 세팅값은 `{프로젝트 루트}/assets/images/{현재 Markdown 파일명}/{현재 시간}`으로 되어 있을 것이다.  
그대로 저장하고 싶다면 바로 `Enter`를 누르면 되고, 나는 개인적으로 이미지 파일명을 이미지를 내용 설명할 수 있는 이름으로 변경하는 것을 선호하여 이 단계에서 파일명을 원하는 것으로 변경한다.  
예를 들어 나는 `example.png`로 이미지 파일명을 변경하였다. 결과적으로 이미지 파일이 잘 생성되며, markdown 파일에 아래와 같이 이미지가 url과 함께 자동으로 삽입된다.

![]({{site.baseurl}}/assets/images/2024-07-28-githubpages-inserting-image/example.png)




