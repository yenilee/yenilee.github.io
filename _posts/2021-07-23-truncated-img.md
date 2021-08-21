---
title: OSError image file is truncated 에러 해결 방법
tags: python PIL image-processing
---

<br/>
PIL에서 이미지를 다룰 때 발생하는 image file is truncated 에러에 대해 ... 👽
<br/>
<!--more-->

# 문제

PIL 라이브러리를 통해 이미지를 load하거나 save할 때 에러가 났다. 아래 코드는 convert지만, convert method도 들어가보면 제일 첫 줄에서 `self.load()`를 하고 있었다.

```python
from PIL import Image

image = Image.open(buffer)
image = image.convert('RGB')

# 에러 발생!!! OSError: image file is truncated (n bytes not processed).
```

PIL 소스 코드를 들어가보면 다음과 같은 상황에서 에러를 내는 것을 알 수 있다.
png/gif 또는 jpeg/jpg 파일을 읽다가 에러가 났을 때, LOAD_TRUNCATED_IMAGES의 True, False 여부에 따라 에러를 raise하도록 되어있다.

```python
# ImageFile.load 함수의 일부

while True:
    try:
        s = read(self.decodermaxblock)
    except (IndexError, struct.error) as e:
        # truncated png/gif
        if LOAD_TRUNCATED_IMAGES:
            break
        else:
            raise OSError("image file is truncated") from e

    if not s:  # truncated jpeg
        if LOAD_TRUNCATED_IMAGES:
            break
        else:
            raise OSError(
                "image file is truncated "
                "(%d bytes not processed)" % len(b)
            )
```

## Truncated image?

이미지 형식에 대해서 잘 모르고, 라이브러리도 익숙하지 않은 상태여서 사실 처음에는 truncated의 뜻도 몰랐다. 간단히 정의해보면
- 구글링을 해보면 '잘린'이라는 뜻으로 나온다.
- stack overflow를 보면 [이미지 프로세싱에서 truncated의 의미](https://stackoverflow.com/questions/59617636/the-meaning-of-truncation-in-image-processing) 가 무엇인지 이미 물어본 글도 있다.
- 정리해보면 `이미지의 가장자리 부분이 잘린 것`이다.


# 해결

## 1. LOAD_TRUNCATED_IMAGES

위 변수를 True로 바꿔주면 에러 없이 동작할 수 있다. PIL ImageFile.py를 들어가면 해당 변수가 default False로 설정되어 있는데, True로 바꾸면 truncated image도 로드하겠다는 설정을 하는 것이다.

```python
from PIL import Image, ImageFile

ImageFile.LOAD_TRUNCATED_IMAGES = True
try:
    image = Image.open(buffer)
    image = image.convert('RGB')
    ...

finally:
    ImageFile.LOAD_TRUNCATED_IMAGES = False
```

## 2. try-except

이미지를 로드에 성공했다. 그런데 해당 이미지가 잘린 이미지라는 것을 확인하려면 어떻게 해야 할까? 로드를 하려면 이미지를 읽을 수 없는 부분을 무시하고 지나가는 것인데, 잘렸다는 것을 어떻게 확인할 수 있을까?
처음에 [Pillow github issue에 올라온 글](https://github.com/python-pillow/Pillow/issues/3012) 을 봤는데, Pillow를 통해 확인할 수 있는 방법은 try-except 뿐이었다.

일단 에러를 내고, except에서 OSError를 캐치해서 처리하는 것이었다. truncated image는 `open 시에는 에러가 발생하지 않기 때문에`, 에러가 생길 수 있는 부분부터 try-except에 넣어준다.

```python
from PIL import Image, ImageFile

image = Image.open(buffer)
try:
    image = image.convert('RGB')
    ...

except OSError as e:
    ...
```

## 3. 이미지 파일의 구조를 이용하기

결론적으로 쓰게된 방법은 이미지 파일의 구조를 이용해 truncated 여부를 찾는 것이었다. multi-thread 환경에서 이미지 프로세싱을 하다보니, 위 LOAD_TRUNCATED_IMAGES는 글로벌 변수이기 때문에
여러 스레드에서 해당 변수를 계속 True -> False로 바꿔줄 경우 thread-safe하지 않았기 때문이다.

JPG, JPEG, PNG 파일의 경우 파일의 구조가 정해져있고, 이미지 파일의 끝과 시작을 나타내는 바이트 값이 고정되어 있다.
이 고정된 값의 이름을 JPEG, JPG는 EOI (End Of Image), PNG는 IEND라고 한다.
그런데 truncated file의 경우 이 값이 고정된 값과 다르게 나타나게 된다. 이 [링크](https://www.programmersought.com/article/44876269499/) 에 자세히 설명되어 있고, 잘린 이미지의 예시도 볼 수 있다.

사실 이 부분은 stack overflow에 질문해서 알게된 것이라서, 답변자의 글을 가져왔다.
답변자는 이미지를 만들어 정상 이미지의 바이트를 확인할 수 있는 코드를 알려주었다.

- 만약 이미지가 JPG(JPEG)면, EOI(0xff 0xd9)로 끝나야 한다.
- 만약 이미지가 PNG면, IEND(49 45 4e 44 ae 42 60 82)로 끝나야 한다.

```python

from PIL import Image
from io import BytesIO

# Create black image
im = Image.new("RGB",(64,48))

# Put into a BytesIO
bytes = BytesIO()

# if jpg, Check last 2 bytes
im.save(bytes, format="JPEG")

buff = bytes.getbuffer()
print(buff[-2:].hex())

# ffd9
```

```python
from PIL import Image
from io import BytesIO

# Create black image
im = Image.new("RGB",(64,48))

# Put into a BytesIO
bytes = BytesIO()

# if png, Check last 8 bytes
im.save(bytes, format="PNG")

buff = bytes.getbuffer()
print(buff[-8:].hex())

#49454e44ae426082
```

실제로 코드는 이미지를 만드는 것이 아니라 기존에 있는 이미지 buffer를 가져와 검사하는 것이기 때문에 im.save를 사용하진 않는다. (truncated image는 save시에도 에러가 발생함)

## 4. verify

직접 사용해보진 않았지만, truncated image를 검색하면 결과에 가장 많이 나오는 결과 중 하나여서 해결에 넣었지만 사실 해결 방법은 아니다.
PIL에 Image.verify()라는 함수가 있고 이름도 굉장히 내가 찾는 방법과 가까워서 써보려고 했는데 아무 결과도 나타나지 않았다.

일단 이 함수는 PNG 파일에만 실행할 수 있고, 이미지의 CRC checksum 만 확인한다고 한다.
CRC checksum을 검색해 보면 아래와 같이 나와있다. 데이터 전송 시 오류를 확인하는 것이지, 잘린 이미지를 체크하는 함수가 아니다.

> 순환 중복 검사, CRC는 네트워크 등을 통하여 데이터를 전송할 때 전송된 데이터에 오류가 있는지를 확인하기 위한 체크값을 결정하는 방식을 말한다.


---
별거 아닌 것 같던 문제도 파고 파다보니... 이렇게 길어졌다.
이 부분 디버깅을 저번부터 했었는데 기록해놓지 않으면 또 잊어버리고 공부하는 시간이 들까봐 정리할 내용들이 많아서 블로그 글에 남겨보았다.
사실 아직 truncated 이미지를 만들어서 테스트코드를 적어야하는 과정까지 남아있는데 얼른 끝내고 싶다 🥲
