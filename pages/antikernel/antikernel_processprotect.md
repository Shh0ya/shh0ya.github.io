---
title: Process Protection
keywords: documentation, technique, debugging
tags: [Windows, Reversing, Dev]
summary: "드라이버를 이용한 프로세스 보호"
sidebar: antikernel_sidebar
permalink: antikernel_processprotect.html
folder: antikernel
---

## [0x00] Overview

프로세스 보호에 관한 내용입니다. 백신과 같은 보안 프로그램에서 어떤 식으로 프로세스를 보호하는지 먼저 알아야 합니다. 해당 챕터에서는 커널 드라이버를 이용하여 어떤 식으로 특정 프로세스 또는 파일 시스템을 보호하는지 확인할 수 있습니다.

단순히 안티 커널 디버깅을 우회하기 위해서라면 해당 챕터의 내용이 필요 없을 수 있습니다. 하지만 역공학을 위해서는 정공학은 필수적인 요소라고 생각합니다.



## [0x01] Process Protect

커널에는 프로세스 또는 스레드가 생성될 때 동작하는 루틴, 이미지가 로드될 때 동작하는 루틴과 같이 다양한 콜백 루틴들이 존재합니다. 여기서 이미지는 모든 `PE Image`를 의미합니다. 

우리는 여기서 `ObRegisterCallbacks` 라는 함수를 이용하여 간단하게 프로세스를 보호하는 드라이버를 구현하고, 이를 분석해보겠습니다. 

먼저 `ObRegisterCallbacks` 함수에 대해 알아보겠습니다.



### [-] ObRegisterCallbacks

프로세스, 스레드 및 데스크톱 핸들 조작을 위한 콜백 루틴 목록을 등록하는 함수입니다.

```cpp
NTSTATUS ObRegisterCallbacks(
  POB_CALLBACK_REGISTRATION CallbackRegistration,
  PVOID                     *RegistrationHandle
);
```

`CallbackRegistration` : 콜백 루틴 및 기타 등록 정보 목록을 가지고 있는 `OB_CALLBACK_REGISTRATION` 포인터
`RegistrationHandle` : 등록 된 콜백 루틴을 식별하는 값에 대한 포인터, `ObUnRegisterCallbacks` 에 전달하여 콜백 루틴의 등록을 취소

반환 값은 `NTSTATUS` 값입니다.

### [-] OB_CALLBACK_REGISTRATION





