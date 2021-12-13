---
title: JPEG의 EOI 마커가 끝이 아닐 수 있다(!)
tags: image-processing
---

<br/>
truncated image를 찾는 과정에서 알게된 Trailer group- 이미지 형식의 세계 🔎 <br/>
<!--more-->


---

이미지는 확장자에 따라 내부적으로 규칙성을 가지고 있다. 깨진 이미지는 이러한 구성을 갖추지 못한 이미지를 말한다.
JPEG의 경우 EOI 마커는 이미지의 끝을 말하고, 마지막 마커가 규칙과 다를 경우 깨진 이미지로 규정하는 방식으로 코드를 짰었다.
그런데 JPEG EOI 마커 뒤에도 여러 데이터가 붙는다는 것을 알게되었고, 이미지 형식에 대해 더 깊이 찾아볼 수 있는 계기가 되었다.

# Image format
## 1) JPEG  marker

- JPEG 이미지는 마커의 집합으로 구성되어 있다.
- 마커는 JPEG 파일에서 구조를 구분하는 2 byte의 값이다.
- 마커 중 payload가 없는 종류는 단독으로 사용하고, payload가 있는 마커는 반드시 마커 뒤에 2byte의 데이터 길이 필드가 있다.


### 주요 마커

JPG 파일은 본래 JIF(JPEG Interchange Format) 형태로 저장되었으나, 최근에는 JIF를 확장해 사용한다고 한다. 그 외에도 검색하면 여러 마커들을 찾을 수 있을 것이다.
- SOI(Start of Image): FF D8
- APP0: JFIF format, FF E0으로 시작
- APP1: Exif format, FF E1으로 시작 (JFIF와 구분)
- EOI(End of Image): FF D9


## 2) PNG chunk

- png 파일은 여러 개의 chunk로 구성되어 있는데, 반드시 포함되어야 하는 세 가지 chunk가 있다.
    - IHDR: 가장 앞에 위치
    - IDAT: 이미지 데이터가 들어가는 부분
    - IEND: 맨 뒤에 위치

## 3) 이미지의 hex값을 확인하는 방법
Ultraedit이라는 툴을 통해서 이미지의 hex값을 확인할 수 있다. 아래 이미지가 예시인데, 맨 처음 값에서 SOI를 확인할 수 있다.
그 뒤에 나오는 FF E1을 통해 이미지가 exif format이라는 것을 알 수 있고, 오른쪽에 samsung이라고 쓰여져 있는 것을 보아 삼성 카메라로 촬영된 이미지인 것 같다.

![ultraedit](/assets/images/ultraedit.png)

# Trailer group
## 1) 발견
이미지 파일의 마지막 바이트를 hex값으로 변환해 ffd9인지 확인해서 truncated 여부를 체크하고 있었다.
그런데 깨지지 않은 이미지에서도 마지막 값이 EOI가 아닌 경우가 있었고, 확인해보니 디지털 카메라 중 JPEG는 EOI 뒤에 trailer라고 해서 여러 데이터를 담는 경우가 있다고 한다.

그래서 뒤에서부터 1000바이트정도를 읽어, EOI가 있는지 여부를 체크하는 로직을 추가했다.

## 2) 예시
![trailer](/assets/images/trailer.png)

오른쪽을 보면 UTC_Data(시간 정보로 추정)라든지, MCC_Data가 있는 것을 알 수 있다.
exiftool.org에서 찾아봤을 때 정리한 태그들을 보면 표 맨 마지막에 Trailer Tag가 나온다.
여기서 삼성 trailer tags에 들어가면 TimeStamp나 MCCData가 있어 내가 비교한 이미지의 Trailer와 일치한다. (mcc가 무엇인지 알고싶었는데 검색을 해도 잘 안 나온다)

---

이미지의 포맷을 조사하다보니, 포맷이 이렇게나 다양한데 내가 사용하는 로직으로 깨짐을 확인할 수 있을지 좀 불안한 부분도 있는 것 같다.
처음에 블로그를 쓰고 4개월정도 뒤 수정하는 지금, 해당 방법으로 깨진 이미지를 잘 찾고 있는 것 같다.
이미지 프로세싱, 형식에 대해 더 많이 배우고 싶다는 생각이 든다.
