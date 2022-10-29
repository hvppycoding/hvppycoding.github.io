---
title: "Fluent Python 04 - 텍스트와 바이트"
excerpt: "Fluent Python 04 - 텍스트와 바이트"
date: 2022-03-20 19:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Fluent Python
mathjax: "true"
---

**Notice:** 본 글은 책 [『전문가를 위한 파이썬<sup>Fluent Python</sup>』](https://books.google.co.kr/books?id=NJpIDwAAQBAJ&hl=ko&source=gbs_navlinks_s)을 학습하며 정리한 글입니다. 전체 소스 코드는 [Fluent Python github 레포지토리](https://github.com/fluentpython/example-code)에서 확인할 수 있습니다.
{: .notice--info}

# Chapter 4. 텍스트와 바이트

## 4.1 문자 문제

'문자열'이라는 개념은 간단하다. 문자열은 문자의 열이다. 그런데 문제는 '문자'의 정의에 있다.  
현재 '문자'를 가장 잘 정의한 것은 유니코드 문자다. 이에 따라 파이썬 3 `str`에서 가져오는 항목도 유니코드 문자다.  
유니코드 표준은 문자의 단위 원소와 특정 바이트 표현을 명확히 구분한다.

- 무자의 단위 원소(<span class="custom-highlight" markdown="1">**코드 포인트<sup>code point</sup>**</span>)는 10 진수 0에서 1,114,111까지의 숫자며, 유니코드 표준에서는 'U+' 접두사를 붙여 4자리에서 6자리 사이의 16진수로 표현한다. 예를 들어 A라는 문자는 코드 표인트 U+0041에, 유로화 기호는 U+20AC에, 음악에서 사용하는 높은음자리표 기호는 U+1D11E에 할당되어 있다.
- 문자를 표현하는 실제 바이트는 사용하는 <span class="custom-highlight" markdown="1">**인코딩**</span>에 따라 달라진다. 인코딩은 코드 포인트를 바이트 시퀀스로 변환하는 알고리즘이다. 문자 A(U+0041)에 대한 코드 포인트는 UTF-8 인코딩에서는 1바이트 \\x41로, UTF-16LE 인코딩에서는ㅌ 2바이트 \\x41\\x00으로 인코딩된다. 그리고 유로화 기호(U+20AC)는 UTF-8에서는 3바이트 \\xe2\\x82\\xac로, UTF-16LE에서는 2바이트 \\xac\\x20으로 인코딩된다.

코드 포인트를 바이트로 변환하는 것을 <span class="custom-highlight" markdown="1">**인코딩**</span>, 바이트를 코드 포인트로 변환하는 것을 <span class="custom-highlight" markdown="1">**디코딩**</span>이라고 한다.

```python
s = 'café'
print(len(s))     # 'café' 문자열은 네 개의 유니코드 문자를 갖고 있다.
# 4
b = s.encode('utf8')     # UTF-8 인코딩을 이용해서 str을 bytes로 인코딩한다.
print(b)
# b'caf\xc3\xa9'     # bytes 리터럴은 접두사 b로 시작한다.
len(b)
# 5
d = b.decode('utf8')     # UTF-8 인코딩을 이용해서 bytes를 str로 디코딩한다.
print(d)
```

## 4.2 바이트에 대한 기본 지식

이진 시퀀스를 위해 사용되는 내장 자료형은 불변형 `bytes`와 가변형 `bytesarray`가 있다.  
`bytes`와 `bytearray`에 들어 있는 각 항목은 0에서 255 사이의 정수ㄹ이다.

이진 시퀀스는 각 바이트 값에 따라 다음과 같이 세 가지 형태로 출력된다.
- 화면에 출력 가능한 아스키 문자는 아스키 문자 그대로 출력한다.
- 탭, 개행 문자, 캐리지 리턴, 백슬래시(\\)는 이스케이프 시퀀스(\\t, \\n, \\r, \\\\)로 출력한다.
- 그 외의 값은 16진수 이스케이프 시퀀스로 출력한다. 예를 들어 널바이트를 나타내는 \\x00과 같이.

앞의 예제에서 `b'caf\\xc3\\xa9'`에서 처음 3바이트 `b'caf'`까지는 출력 가능한 아스키 범위에 있지만, 나머지 2바이트는 출력 가능한 범위에 속하지 않는 바이트이다.

### 4.2.1 구조체와 메모리 뷰

`memoryview`와 `struct` 모듈을 이용하여 이진 시퀀스에 접근할 수 있다. 예를 들어 GIF 이미지 파일의 header를 읽어 버전, 너비, 높이를 읽을 수 있다.

## 4.3 기본 인코더/디코더

텍스트를 바이트로 혹은 바이트를 텍스트로 변환하기 위해 파이썬 배포본에는 100여 개의 코덱(인코더/디코더)이 포함되어 있다. 각 코덱은 `utf_8`과 같은 이름을 갖고 있는데, 코덱은 `open()`, `str.encode()`, `bytes.decode()` 등의 함수를 호출 할 때 `encoding` 인수에 전달해서 사용할 수 있다.

```python
for codec in ['latin_1', 'utf_8', 'utf_16']:
    print(codec, 'El Niño'.encode(codec))

# latin_1 b'El Ni\xf1o'
# utf_8 b'El Ni\xc3\xb1o'
# utf_16 b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

아래 표는 7개의 코덱을 이용해서 A 문자부터 높은음자리표까지 여러 문자의 바이트 배열을 생성한 결과를 보여준다. 마지막 세 개의 인코딩은 가변 길이, 다중바이트 인코딩이다. 별표(*)로 표시한 항목은 해당 인코딩으로 표현할 수 없음을 나타낸다.

![Encodings]({{site.baseurl}}/assets/images/2022-03-20-encodings.png)

## 4.4 인코딩/디코딩 문제 이해하기

`UnicodeError`라는 범용 예외가 있지만, 거의 항상 `UnicodeEncodeError`(`str`을 이진 시퀀스로 변환할 때)나 `UnicodeDecodeError`(이진 시퀀스를 `str`로 읽어 들일 때) 같은 
구체적인 예외가 발생한다. 파이썬 모듈을 로딩할 때 소스 코드가 예기지 않은 방식으로 인코딩되어 있으면 `SyntaxError`가 발생하기도 한다.

### 4.4.1 `UnicodeEncodeError` 처리하기
다ㅐ부분의 비UTF 코덱은 유니코드 문자의 일부만 처리할 수 있다. 텍스트를 바이트로 변환할 때 문자가 대상 인코딩에 정의되어 있지 않으면, 인코딩 메서드나 함수의 `errors` 인수에 별도의 처리기를 지정하지 않는 한 `UnicodeEncodeError`가 발생한다.

```python
city = 'São Paulo'
print(city.encode('utf_8'))
# b'S\xc3\xa3o Paulo'
print(city.encode('utf_16'))
# b'\xff\xfeS\x00\xe3\x00o\x00 \x00P\x00a\x00u\x00l\x00o\x00'
print(city.encode('iso8859_1'))
# b'S\xe3o Paulo'
# print(city.encode('cp437'))
# Traceback (most recent call last):
#   File "encoding_test.py"
#     print(city.encode('cp437'))
#   File "cp437.py", line 12, in encode
#     return codecs.charmap_encode(input,errors,encoding_map)
# UnicodeEncodeError: 'charmap' codec can't encode character '\xe3' in position 1: character maps to <undefined>
print(city.encode('cp437', errors='ignore'))     # 에러를 무시
# b'So Paulo'
print(city.encode('cp437', errors='replace'))     # 에러를 물음표(?)로 대체
# b'S?o Paulo'
```

### 4.4.2 `UnicodeDecodeError` 처리하기
모든 바이트가 정당한 아스키 문자가 될 수 없으며, 모든 바이트 시퀀스가 정당한 UTF-8이나 UTF-16 문자가 되는 것은 아니다. 따라서 이진 시퀀스를 텍스트로 변환할 때 정당한 문자로 변환할 수 없으면 `UnicodeDecodeError`가 발생한다.  
그렇지만 '`cp1252`', '`iso8859_1`' 등 많은 레거시 8비트 코덱은 무작위 비트 배열에 대해서도 에러를 발생시키지 않고 바이트 스트림으로 디코딩할 수 있다. 따라서 프로그램이 잘못된 8비트 코덱을 사용하면 쓰레기 문자를 조용히 디코딩하게 된다.
이로 인해 문자가 꺠지거나 `UnicodeDecodeError`가 발생할 수 있다.

### 4.4.3 예상과 달리 인코딩된 모듈을 로딩할 때 발생하는 `SyntaxError`

파이썬 2.5부터는 아스키를, 파이썬 3부터는 UTF-8을 소스 코드 기본 인코딩 방식으로 사용했다. 파이썬 3에서 인코딩 선언 없이 비 UTF-8로 인코딩된 .py 모듈을 로딩하면 에러 메시지가 발생한다.  

### 4.4.4 바이트 시퀀스의 인코딩 방식을 알아내는 방법
간단히 말하면, 할 수 없다. 반드시 별도로 인코딩 정보를 가져와야 한다. 그렇지만 일단 바이트 스트림이 자연어(평문 텍스트)라고 간주되면, 경험과 통계를 이용해서 인코딩 방식을 추정할 수는 있다. Chardet 패키지는 이런 방법을 이용해서 30가지 인코딩 방식을 알아낸다.

## 4.5 텍스트 파일 다루기

텍스트를 처리하는 최고의 방법은 '유니코드 샌드위치'다. 입력할 때는 가능하면 빨리 bytes를 str로 변환해야 한다. 샌드위치 가운데 들어가는 '고기'는 프로그램의 비즈니스 논리에 해당하는 부분이며, 여기서는 텍스트를 오로지 str 객체로 다룬다(encoding 관련 문제를 최소화). 출력을 할 때는 가능한 늦게 str을 bytes로 인코딩한다.

### 4.5.1 기본 인코딩 설정: 정신 나간 거 아냐?

텍스트 파일을 열 때 플랫폼별 기본 인코딩 설정에 의존한다. 파일을 열 때 encoding 인수를 생략하면, `locale.getpreferredencoding()`에 의해 기본 인코딩 방식이 설정된다. 

## 4.6 제대로 비교하기 위해 유니코드 정규화하기

유니코드에는 결합 문자가 있기 때문에 문자열 비교가 간단하지 않다. 앞 문자에 연결되는 발음 구별 기호는 인쇄할 때 앞 문자와 하나로 결합되어 출력된다. 예를 들어 'café'라는 단어는 네 개나 다섯 개의 코드 포인트를 이용해서 두 가지 방식으로 표현할 수 있지만 결과는 동일하게 나타난다.  

```python
s1 = 'café'
s2 = 'cafe\u0301'
print(s1, s2)
# café café
print(len(s1), len(s2))
# 4 5
print(s1 == s2)
# False
```

코드 포인트 U+0301은 COMBINING ACUTE ACCENT다. 'e' 다음에 이 문자가 보면 'é'를 만든다. 유니코드 표준에서는 이 두 개의 시퀀스를 '규범적으로 동일하다'고 하며, 애플리케이션은 이 두 시퀀스를 동일하게 처리해야 한다.  
이 문제를 해결하려면 `unicodedata.normalize()` 함수가 제공하는 유니코드 정규화를 이용해야 한다. 정규화 시 첫 번째 인수는 'NFC', 'NFD', 'NFKC', 'NFKD' 중 하나여야 하는데, 먼저 NFC와 NFD를 알아보자.  
정규화 방식 C<sup>Normalization Form C</sup>(NFC)는 코드 포인트를 조합해서 가장 짧은 동일 문자열을 생성하는 반면 NFD는 조합된 문자를 기본 문자와 별도의 결합 문자로 분리한다. 이 두 방식 모두 문자열을 제대로 비교할 수 있게 해준다.

```python
from unicodedata import normalize

s1 = 'café'
s2 = 'cafe\u0301'
print(len(s1), len(s2))
# 4 5
print(len(normalize('NFC', s1)), len(normalize('NFC', s2)))
# 4 4
print(len(normalize('NFD', s1)), len(normalize('NFD', s2)))
# 5 5
print(normalize('NFC', s1) == normalize('NFC', s2))
# True
print(normalize('NFD', s1) == normalize('NFD', s2))
# True
```
사용자가 입력하는 텍스트는 기본적으로 NFC 형태다. 안전을 보장하기 위해 파일에 저장하기 전에 `normalize('NFC', user_text)`를 이용해서 문자열을 청소하는 것이 좋다. NFC는 W3C가 추천하는 정규화 형식이기도 하다.

### 4.6.1 케이스 폴딩

케이스 폴딩<sup>case folding</sup>는 모든 텍스트를 소문자로 변환하는 연산이다. 

### 4.6.2 정규화된 텍스트 매칭을 위한 유틸리티 함수
`nfc_equal(a, b)`, `fold_equal('A', 'a')`

### 4.6.3 극단적인 '정규화': 발음 구별 기호 제거하기
검색을 할 때 발음 구별 기호를 무시하는 방법도 있다. 또한, 발음 구별 기호 제거를 통해 URL이 읽기 편해진다는 장점도 있다. 아래의 코드와 doctest를 참조하자.

```python
"""
Radical folding and text sanitizing.

Handling a string with `cp1252` symbols:

    >>> order = '“Herr Voß: • ½ cup of Œtker™ caffè latte • bowl of açaí.”'
    >>> shave_marks(order)
    '“Herr Voß: • ½ cup of Œtker™ caffe latte • bowl of acai.”'
    >>> shave_marks_latin(order)
    '“Herr Voß: • ½ cup of Œtker™ caffe latte • bowl of acai.”'
    >>> dewinize(order)
    '"Herr Voß: - ½ cup of OEtker(TM) caffè latte - bowl of açaí."'
    >>> asciize(order)
    '"Herr Voss: - 1⁄2 cup of OEtker(TM) caffe latte - bowl of acai."'

Handling a string with Greek and Latin accented characters:

    >>> greek = 'Ζέφυρος, Zéfiro'
    >>> shave_marks(greek)
    'Ζεφυρος, Zefiro'
    >>> shave_marks_latin(greek)
    'Ζέφυρος, Zefiro'
    >>> dewinize(greek)
    'Ζέφυρος, Zéfiro'
    >>> asciize(greek)
    'Ζέφυρος, Zefiro'

"""

# BEGIN SHAVE_MARKS
import unicodedata
import string


def shave_marks(txt):
    """Remove all diacritic marks"""
    norm_txt = unicodedata.normalize('NFD', txt)  # <1>
    shaved = ''.join(c for c in norm_txt
                     if not unicodedata.combining(c))  # <2>
    return unicodedata.normalize('NFC', shaved)  # <3>
# END SHAVE_MARKS

# BEGIN SHAVE_MARKS_LATIN
def shave_marks_latin(txt):
    """Remove all diacritic marks from Latin base characters"""
    norm_txt = unicodedata.normalize('NFD', txt)  # <1>
    latin_base = False
    keepers = []
    for c in norm_txt:
        if unicodedata.combining(c) and latin_base:   # <2>
            continue  # ignore diacritic on Latin base char
        keepers.append(c)                             # <3>
        # if it isn't combining char, it's a new base char
        if not unicodedata.combining(c):              # <4>
            latin_base = c in string.ascii_letters
    shaved = ''.join(keepers)
    return unicodedata.normalize('NFC', shaved)   # <5>
# END SHAVE_MARKS_LATIN

# BEGIN ASCIIZE
single_map = str.maketrans("""‚ƒ„†ˆ‹‘’“”•–—˜›""",  # <1>
                           """'f"*^<''""---~>""")

multi_map = str.maketrans({  # <2>
    '€': '<euro>',
    '…': '...',
    'Œ': 'OE',
    '™': '(TM)',
    'œ': 'oe',
    '‰': '<per mille>',
    '‡': '**',
})

multi_map.update(single_map)  # <3>


def dewinize(txt):
    """Replace Win1252 symbols with ASCII chars or sequences"""
    return txt.translate(multi_map)  # <4>


def asciize(txt):
    no_marks = shave_marks_latin(dewinize(txt))     # <5>
    no_marks = no_marks.replace('ß', 'ss')          # <6>
    return unicodedata.normalize('NFKC', no_marks)  # <7>
# END ASCIIZE
```

## 4.7 유니코드 텍스트 정렬하기
파이썬은 각 시퀀스 안에 들어 있는 항목들을 하나하나 비교함으로써 어떠한 자료형의 시퀀스도 정렬할 수 있다. 문자열의 경우에는 각 단어의 코드 포인트를 비교한다. 하지만 이런 방식은 비아스키 문자를 사용하는 경우 부적절한 결과가 발생할 수 있다.

`locale.strxfrm` 함수를 정렬 시 key로 사용하여 문자열을 현지어 비교에 사용할 수 있는 문자열로 변환하자.

```python
fruits = ['caju', 'atemoia', 'cajã', 'açai', 'acerola']
print(sorted(fruits))
# ['acerola', 'atemoia', 'açai', 'caju', 'cajã']

import locale
locale.setlocale(locale.LC_COLLATE, 'pt_BR.UTF-8')    # locale.strxfrm 사용전 지역 언어 설정 필요

print(sorted(fruits, key=locale.strxfrm))
# ['açai', 'acerola', 'atemoia', 'cajã', 'caju']
```

### 4.7.1 유니코드 대조 알고리즘을 이용한 정렬

지역 설정에 따라 달라지므로 복잡한 배포 문제가 남아있다. 유니코드 대조 알고리즘<sup>Unicode Collation Algorithm</sup>(UCA) 패키지인 PyUCA를 사용하면 간단히 해결할 수 있다.

## 4.8 유니코드 데이터베이스

유니코드 표준은 각 문자에 대한 메타데이터 데이터베이스를 제공한다. 예를 들어 문자를 출력할 수 있는지, 문자인지, 십진수인지 혹은 다른 수치형 기호인지 기록한다.

```python
import unicodedata
import re

re_digit = re.compile(r'\d')

sample = '1\xbc\xb2\u0969\u136b\u216b\u2466\u2480\u3285'

for char in sample:
    print('U+%04x' % ord(char),                       # U+0000 포맷의 코드 포인트
          char.center(6),                             # 문자를 길이 6인 str의 중앙에 위치
          're_dig' if re_digit.match(char) else '-',  # r'\d' 정규 표현식과 일치하는 문자의 경우 re_dig
          'isdig' if char.isdigit() else '-',         # char.isdigit()가 True면 isdig 표시
          'isnum' if char.isnumeric() else '-',       # char.isnumeric()이 True면 isnum 표시
          format(unicodedata.numeric(char), '5.2f'),  # 숫자 형식으로 포맷팅
          unicodedata.name(char),                     # 유니코드 문자명
          sep='\t')
```

<div class="no-highlight" markdown="1">

```
U+0031    1     re_dig  isdig   isnum    1.00   DIGIT ONE
U+00bc    ¼     -       -       isnum    0.25   VULGAR FRACTION ONE QUARTER
U+00b2    ²     -       isdig   isnum    2.00   SUPERSCRIPT TWO
U+0969    ३     re_dig  isdig   isnum    3.00   DEVANAGARI DIGIT THREE
U+136b    ፫     -       isdig   isnum    3.00   ETHIOPIC DIGIT THREE
U+216b    Ⅻ     -       -       isnum   12.00   ROMAN NUMERAL TWELVE
U+2466    ⑦     -       isdig   isnum    7.00   CIRCLED DIGIT SEVEN
U+2480    ⒀     -       -       isnum   13.00   PARENTHESIZED NUMBER THIRTEEN
U+3285    ㊅    -       -       isnum    6.00   CIRCLED IDEOGRAPH SIX
```

</div>

## 4.9 이중 모드 str 및 bytes API

### 4.9.1 정규 표현식에서의 str과 bytes

`bytes`로 정규 표현식을 만들면 `\\d`와 `\\w` 같은 패턴은 아스키 문자만 매칭되지만, 
`str`로 이 패턴을 만들면 유니코드 숫자나 문자도 매칭된다.

### 4.9.2 os 모듈 함수에서 str와 bytes

생략

