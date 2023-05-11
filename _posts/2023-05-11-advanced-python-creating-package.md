---
title: "Advanced Python: Creating Package"
excerpt: "Creating a package"
date: 2023-05-11 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Advanced Python
---

Advanced Python Programming: Build 10 OOP Application - App 11: Creating a Weather Forecast Package
{: .notice--info}

## Introduction

- `pip install folium` 같은 명령어를 통해 파이썬 패키지를 설치할 수 있다.
- 그러면 설치가 자동으로 진행되며, `import folium`을 통해 패키지를 사용할 수 있다.
- 이러한 패키지들은 `pypi.org`와 같은 Repository 사이트에 저장되어 있다.

{% include figure image_path="/assets/images/2023-05-11-pypi.png" alt="2023-05-11-pypi.png" caption="PyPI에서 folium 검색 결과" %}

- 본 포스트에서는 라이브러리를 처음부터 만들어보고 pypi에 업로드해볼 것이다.

## Demo of Package

- 이번에 만들고자 하는 라이브러리를 미리 보자.
- 라이브러리명: `sunnyday`
- 클래스: `Weather`

**주의:** 이 라이브러리를 사용하기 위해서는 [openweathermap.org](https://openweathermap.org/) 사이트를 가입한 후 발급되는 API Key가 필요하다.
{: .notice--danger}

```python
from sunnyday import Weather

help(Weather)
"""
Weather 클래스의 help mesaage 출력

Weather(apikey = "xxxxxx", city = "Madrid")
"""
```

## The Library Structure

- 표준적인 라이브러리 구조를 따를 것이다.
- 라이브러리에서 입력을 받고 그에 따라 두 가지 형태의 결과를 리턴할 것이다. 한 가지 형태는 Raw Data형태이고 다른 한 가지 형태는 Summary된 결과이다.

## Preparing the Input Data

- openweathermap에서 데이터를 받아와보자.
- API를 사용해서 데이터를 받아올 수 있다.
- API doc을 통해 서비스를 호출하는 방법을 가이드 받을 수 있다.
- 사용하고자 하는 API 형식은 다음과 같다.
  - `api.openweathermap.org/data/2.5/forecast?q={city name},{country code}&appid={API key}`
- 아래는 서울의 날씨를 받아오는 예시이다. 이를 웹 브라우저 주소창에 넣으면 데이터를 받아올 수 있다.
  - `https://api.openweathermap.org/data/2.5/forecast?q=seoul&appid=<<API_KEY>>`
- 브라우저를 사용하는 대신에 파이썬 코드를 통해 데이터를 받아올 수도 있다.

```python
import requests

url = "https://api.openweathermap.org/data/2.5/forecast?q=seoul&appid=<<API_KEY>>"
r = requests.get(url)
r.text    # string type data
r.json()  # dictionary type data
```

**팁:** `dict` 분석 시 `pprint` 패키지를 사용하면 구조를 파악하기 쉽다.
{: .notice--info}

## Creating the Weather Class

```python
import requests

class Weather:
    """Create a Weather object to get weather data from OpenWeatherMap API.
    
    Package use example:
    
    # Create a weather object using a city name.
    # Get your own API_KEY from https://openweathermap.org
    >>> weather = Weather(apikey="<<API_KEY>>", city="rome")
    
    # Get complete data for the next 12 hours.
    >>> weather.next_12h()
    
    # Simplified data for the next 12 hours.
    >>> weather.next_12h_simplified()
    """
    def __init__(self, apikey: str, city: str) -> None:
        url = f"https://api.openweathermap.org/data/2.5/forecast?q={city}&appid={apikey}&units=metric"
        r = requests.get(url)
        self.data = r.json()
        if self.data["cod"] != "200":
            raise ValueError(self.data["message"])
    
    def next_12h(self):
        return self.data['list'][:4]
    
    def next_12h_simplified(self):
        simple_data = []
        for dicty in self.next_12h():
            simple_data.append((
                dicty['dt_txt'],
                dicty['main']['temp'],
                dicty['weather'][0]['description']
            ))
        return simple_data
```

## Uploading the Library to PyPI

먼저 PyPI에 업로드하기 위해 [PyPI](https://pypi.org/)에 가입해야 한다.  
그리고 아래와 같은 패키지 구조를 갖추어야 한다.

```text
sunnyday
├ __init__.py
└ sunnyday.py
setup.py
```

각 파일의 내용은 다음과 같다.

```python
# __init__.py
from sunnyday.sunnyday import Weather
```

```python
# setup.py
from setuptools import setup

setup(
    name="sunnyday",
    version="1.0.0",
    description="Weather forecast data",
    author="<<YOUR_NAME>>",
    author_email="<<YOUR_EMAIL>>",
    packages=["sunnyday"],
    install_requires=["requests"],
    keywords=["weather", "forecast", "openweather"],
)
```

그 후 `setup.py` 파일이 있는 디렉토리에서 다음의 명령어를 통해 업로드할 파일을 생성한다.

- `python setup.py sdist`

`twine` 패키지를 통해 PyPI에 업로드할 수 있다. 업로드 시 PyPI ID와 비밀번호를 입력해야 한다.

- `twine upload --skip-existing dist/*`

**주의:** 이미 `sunnyday` 패키지가 PyPI에 존재하여 제대로 진행이 안될 수 있음
{: .notice--danger}

업로드 후에는 `pip` 명령어를 통해 업로드된 패키지를 어디서나 설치할 수 있다.

## Making and Uploading a Package Change

패키지 수정되면 PyPI에도 업데이트를 해야한다. `setup.py`의 `version`을 변경하고, 첫 업로드와 동일한 방법으로 다시 업로드한다.

```bash
python setup.py sdist
twine upload --skip-existing dist/*
```