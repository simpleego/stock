jong, 이번에는 **로그인 이후에 뜬 오류 메시지(“opcomms.dll 파일 삭제 실패 [183]”)**가 핵심 단서야.  
이 오류는 **키움 OpenAPI+ 업데이트 또는 실행 과정에서 DLL 파일이 잠겨 있어서 삭제/갱신이 안 될 때** 발생하는 대표적인 문제야.

즉,  
> **OpenAPI+ 내부 DLL이 다른 프로세스에 의해 사용 중이라 업데이트가 실패한 상태**  
라는 뜻이야.

그래서 로그인 이후 이벤트가 정상적으로 연결되지 않고,  
결국 `OnEventConnect`가 없는 상태로 파이썬에서 로드되는 거지.

---

# 🔥 오류 원인 요약
오류 메시지:

```
[C:\OpenAPI\opcomms.dll] 파일 삭제 실패 [183]
```

오류 코드 183의 의미는:

> **이미 존재하는 파일을 덮어쓰려고 했는데, 파일이 사용 중이라 실패함**

즉, 아래 중 하나가 파일을 잡고 있어서 업데이트가 안 된 상태야:

- 영웅문 HTS가 실행 중  
- KOA Studio가 실행 중  
- 키움 관련 프로세스가 백그라운드에서 실행 중  
- 이전 업데이트가 비정상 종료됨  
- Windows가 DLL을 잠금 상태로 유지

이 상태에서는 **KHOpenAPI가 정상 로드되지 않기 때문에**  
파이썬에서 `OnEventConnect`가 절대 나타나지 않아.

---

# 🟦 해결 방법 (가장 확실한 순서)

## ✅ 1) 모든 키움 관련 프로그램 종료
아래를 모두 종료해야 해:

- 영웅문 HTS  
- KOA Studio  
- 키움 로그인 창  
- 키움 자동업데이트 창  
- 파이썬 실행창  
- VSCode / PyCharm (파이썬 프로세스 포함)

그리고 **작업 관리자**에서 아래 프로세스가 있으면 모두 종료:

- `KHOpenAPI.exe`
- `opstarter.exe`
- `opversionup.exe`
- `ngen.exe`
- `winevent.exe`

---

## ✅ 2) PC 재부팅 (중요)
DLL 잠금이 풀리지 않으면 절대 해결되지 않아.

재부팅 후 **아무 프로그램도 실행하지 말고** 바로 다음 단계로.

---

## ✅ 3) OpenAPI+ 업데이트 재실행
재부팅 후 아래 파일 실행:

```
C:\OpenAPI\opstarter.exe
```

또는  
영웅문 실행 → 자동 업데이트 진행

이때 더 이상 “파일 삭제 실패”가 뜨면 안 돼.

---

## ✅ 4) COM DLL 재등록
관리자 CMD 실행 후:

```cmd
regsvr32 "C:\OpenAPI\khopenapi.dll"
```

성공 메시지가 떠야 정상.

---

## ✅ 5) 파이썬에서 다시 테스트

```python
from PyQt5.QAxContainer import QAxWidget
from PyQt5.QtWidgets import QApplication
import sys

app = QApplication(sys.argv)
ocx = QAxWidget("KHOPENAPI.KHOpenAPICtrl.1")
print(ocx)
```

정상이라면:

```
<PyQt5.QAxContainer.QAxWidget object at ...>
```

그리고 이벤트도 정상적으로 붙어야 해:

```python
print(dir(ocx))
```

여기서:

- `OnEventConnect`
- `OnReceiveTrData`

가 보여야 정상.

---

# 🎯 jong, 지금 해야 할 것

1) PC 재부팅  
2) 키움 관련 프로세스 전부 종료  
3) OpenAPI+ 업데이트 재실행  
4) DLL 재등록  
5) 파이썬에서 OCX 객체 재테스트  

이 순서대로 하면 **OnEventConnect가 다시 살아나고**,  
로그인 이후 오류도 사라질 거야.

---

원하면 내가  
- 작업 관리자에서 종료해야 할 프로세스 목록  
- OpenAPI+ 완전 삭제 후 재설치 방법  
- 정상 설치 여부 체크 스크립트  
도 바로 만들어줄게.
