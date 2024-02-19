# Hellomon

## 개요

Procmon 은 MS 社 Sysinternals 에 포함된 동적 모니터링 프로그램으로, Windows 시스템에서 동적 디버깅에 유용하고 사용할 수 있다. 하지만 널리 사용되는만큼 악성 프로그램, 프로텍터에서도 Procmon 을 탐지하려는 경향이 있다.
이 글에서는 Themida 프로텍터 에서 Procmon 을 탐지하는 방식과, Procmon 실행 파일을 패치하여 탐지하지 못하도록 하는 방안에 대한 연구를 다룬다.



![img](https://www.moonding.co.kr/content/images/2024/02/msg_box.PNG)



테스트 샘플의 Themida 버전 정보는 다음과 같다. 

- Protector : Themida/Winlicense(2.X)

![img](https://www.moonding.co.kr/content/images/2024/02/die.PNG)



## Procmon 탐지 방식

### 드라이버 탐지

Procmon 실행 시 파일 시스템 미니 필터에 `PROCMON[버전숫자]`라는 이름의 드라이버가 로딩된다. 이렇게 로딩된 드라이버는 Procmon 프로세스가 종료되도 시스템 재부팅 전까지 계속 로딩된 상태를 유지한다. Themida 는 로딩된 미니 필터를 탐지하여 Procmon 의 실행 여부를 알 수 있다.

![img](https://www.moonding.co.kr/content/images/2024/02/fltmc.PNG)



### FindWindowA API

`FindWindowA` API 는 윈도우 클래스 이름, 윈도우 이름을 인자로 전달받아 조건에 맞는 윈도우의 핸들을 반환받는 함수이다. Themida 프로텍터는 다음과 같은 문자열을 인자로 `FindWindowA` 를 호출하여 디버거, 모니터링 프로세스의 실행 여부를 확인한다. 이 중 `PROCMON_WIDNOW_CLASS`, `Process Monitor - Sysinternals: www.sysinternals.com` 문자열이 Procmon 실행 시 생성되는 윈도우와 관련된 문자열이다.



|                      윈도우 클래스 이름                      |                         윈도우 이름                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| `OLLYDBG`</br>`GBDYLLO`</br>`pediy06`</br>`FilemonClass`</br>`PROCMON_WINDOW_CLASS`</br>`RegmonClass`</br>`18467-41` | `File Monitor - Sysinternals: www.sysinternals.com`</br>`Process Monitor - Sysinternals: www.sysinternals.com`</br>`Registry Monitor - Sysinternals: www.sysinternals.com` |



## 패치 방안

### Unicode 문자열 패치

앞서 언급된 드라이버, 윈도우와 관련된 문자열은 Procmon 실행 파일 내부에 하드코딩되어 있다. 해당 값과 일치하는 Unicode 문자열을 모두 임의의 값으로 패치하면 Themida 프로텍터의 탐지 기능을 우회할 수 있다.



|                        수정 전 문자열                        |                        수정 후 문자열                        |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| `PROCMON23.SYS`</br>`PROCMON_WINDOW_CLASS`</br>`Process Monitor - Sysinternals: www.sysinternals.com` | `HROCWON91.SYS`</br>`HROCMON_WINDOW_CLASS`</br>`Hrocess Monitor - Hysinternals: www.sysinternals.com` |



