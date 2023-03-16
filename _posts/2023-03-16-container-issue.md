---
title: 파드 Worker 무한 재시작 현상 (feat.numpy)
tags: kubernetes numpy
---

<br/>
배포할 때마다 워커가 죽고 재시작을 반복...... 왜그러니 진짜 <br/>
<!--more-->

---

# 문제
최근 일부 패키지 업데이트 이후 배포 시마다 개발 환경 파드들이 먹통이 되는 일이 발생했다.
운영 환경에서는 큰 문제가 없었으나 개발 환경에서 해당 문제가 발생해 개발에 어려움이 있었다.

```bash
[2023-03-15 09:42:07 +0900] [9] [INFO] Booting worker with pid: 9
[2023-03-15 09:42:07 +0900] [10] [INFO] Booting worker with pid: 10
[2023-03-15 09:42:37 +0900] [1] [CRITICAL] WORKER TIMEOUT (pid:9)
[2023-03-15 09:42:37 +0900] [1] [CRITICAL] WORKER TIMEOUT (pid:10)
[2023-03-15 09:42:37 +0900] [9] [INFO] Worker exiting (pid: 9)
[2023-03-15 09:42:37 +0900] [10] [INFO] Worker exiting (pid: 10)
[2023-03-15 09:42:38 +0900] [1] [WARNING] Worker with pid 8 was terminated due to signal 9
[2023-03-15 09:42:38 +0900] [55] [INFO] Booting worker with pid: 55
[2023-03-15 09:42:38 +0900] [1] [WARNING] Worker with pid 9 was terminated due to signal 9
[2023-03-15 09:42:38 +0900] [56] [INFO] Booting worker with pid: 56

```

# 문제 파악
최근 변경점이 패키지 업데이트 외에는 크게 없었기 때문에 처음엔 특정 패키지의 문제인줄 알았다. 위 로그 중간중간에 아래 로그가 섞여서 보였기 때문이다.
해당 로그는 시스템에서 그대로 가져온 것이 아니라 깃헙에서 동일한 이슈를 겪은 사람의 로그 중 비슷한 것을 가져온 것이다.

```bash
/usr/lib/python3.9/site-packages/numpy/core/getlimits.py:499: UserWarning: The value of the smallest subnormal for <class 'numpy.float32'> type is zero.
  setattr(self, word, getattr(machar, word).flat[0])
/usr/lib/python3.9/site-packages/numpy/core/getlimits.py:89: UserWarning: The value of the smallest subnormal for <class 'numpy.float32'> type is zero.
  return self._float_to_str(self.smallest_subnormal)
/usr/lib/python3.9/site-packages/numpy/core/getlimits.py:499: UserWarning: The value of the smallest subnormal for <class 'numpy.float64'> type is zero.
  setattr(self, word, getattr(machar, word).flat[0])
/usr/lib/python3.9/site-packages/numpy/core/getlimits.py:89: UserWarning: The value of the smallest subnormal for <class 'numpy.float64'> type is zero.
  return self._float_to_str(self.smallest_subnormal)
```

# 과정
위 로그가 굉장시 낯선 것이었기 때문에 워커 Booting, TIMEOUT, terminated를 반복하면서 보낸 경고는 무시하고 해당 로그에만 집중을 했다.
(사실 해당 로그는 문제의 원인이 아니었다.)

## 로그의 의미
numpy의 [issue](https://github.com/numpy/numpy/issues/20895)에 해당 문제를 다룬 글이 잘 설명되어 있다.
해당 로그는 결국 opencv가 부동 소수점을 "flush to zero"로 핸들링을 하도록 설정되어 있기 때문에, numpy의 비정규값 처리에 영향을 준다는 내용이었다.
원래는 numpy를 1.19.x 를 쓰다가 1.24.x로 업그레이드를 했고, 그 이후 로그가 보이기 시작했던 것이었다.
그래서 1.21.x로 버전을 고정하니 로그가 더이상 보이지 않았다. opencv 외에도 해당 부분에 대해 영향을 주는 패키지들이 꽤 많다고 한다.


# 해결
위 문제를 한참 보다가 결국 해당 로그는 경고일 뿐이라는 것을 알게됐고, 근본적 원인은 컨테이너 리소스의 문제라는 점을 발견했다.
인프라 코드를 뒤지다 보니 다른 환경과 리소스 설정이 다르게 되어 있었는데,
패키지 및 로직 수정으로 기존보다 시스템 메모리와 CPU가 더 필요해졌던 것인데 다른 환경에 비해 굉장히 적은 리소스가 분배되어 있던 것이었다.
인프라에서 리소스 설정을 변경해주니 바로 해당 문제가 사라졌다.

# 결과
결과적으로는 더이상 배포 시에도 죽지 않고 정상적으로 롤링 업데이트될 수 있도록 수정되었다.
현상 속에서도 주목해야 하는 로그를 찾아내는게 중요한 것 같다.
