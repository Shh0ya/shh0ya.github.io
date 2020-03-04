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

사용하는 함수나 구조체를 확인해야 하기 때문에 windbg를 이용해 실습 과정이 존재합니다.



## [0x01] Process Protect

커널에는 프로세스 또는 스레드가 생성될 때 동작하는 루틴, 이미지가 로드될 때 동작하는 루틴과 같이 다양한 콜백 루틴들이 존재합니다. 여기서 이미지는 모든 `PE Image`를 의미합니다. 

우리는 여기서 `ObRegisterCallbacks` 라는 함수를 이용하여 간단하게 프로세스를 보호하는 드라이버를 구현하고, 이를 분석해보겠습니다. 

먼저 `ObRegisterCallbacks` 함수와 사용되는 구조체에 관해 알아보겠습니다.



### [-] ObRegisterCallbacks

프로세스, 스레드 및 데스크톱 핸들 조작을 위한 콜백 루틴 목록을 등록하는 함수입니다.

```cpp
NTSTATUS ObRegisterCallbacks(
  POB_CALLBACK_REGISTRATION CallbackRegistration,
  PVOID                     *RegistrationHandle
);
```

- CallbackRegistration : 콜백 루틴 및 기타 등록 정보 목록을 가지고 있는 `OB_CALLBACK_REGISTRATION`의 포인터
- RegistrationHandle : 등록 된 콜백 루틴을 식별하는 값에 대한 포인터, `ObUnRegisterCallbacks` 에 전달하여 콜백 루틴의 등록을 취소



### [-] OB_CALLBACK_REGISTRATION

```cpp
typedef struct _OB_CALLBACK_REGISTRATION {
    _In_ USHORT                     Version;
    _In_ USHORT                     OperationRegistrationCount;
    _In_ UNICODE_STRING             Altitude;
    _In_ PVOID                      RegistrationContext;
    _In_ OB_OPERATION_REGISTRATION  *OperationRegistration;
} OB_CALLBACK_REGISTRATION, *POB_CALLBACK_REGISTRATION;
```

- Version : 요청 된 Object Callback Registration 의 버전, 드라이버는 `OB_FLT_REGISTRATION_VERSION` 값을 지정
- OperationRegistrationCount : `OperationRegistration` 배열의 항목 수
- Altitude : 드라이버의 고도(유니코드), MS에 등록되어 사용되며 로드 순서와도 관계가 있음
- RegistrationContext : 콜백 루틴이 실행될 때 해당 값을 콜백 루틴으로 전달
- OperationRegistration : `OB_OPERATION_REGISTRATION`의 포인터, `ObjectPre, PostCallback` 루틴이 호출되는 유형을 지정



### [-] OB_OPERATION_REGISTRATION

```cpp
typedef struct _OB_OPERATION_REGISTRATION {
    POBJECT_TYPE                *ObjectType;
    OB_OPERATION                Operations;
    POB_PRE_OPERATION_CALLBACK  PreOperation;
    POB_POST_OPERATION_CALLBACK PostOperation;
} OB_OPERATION_REGISTRATION, *POB_OPERATION_REGISTRATION;
```

- ObjectType : 콜백 루틴을 동작시키는 오브젝트 유형에 대한 포인터
  - PsProcessType : 프로세스 핸들 동작을 위한 유형
  - PsThreadType : 스레드 핸들 동작을 위한 유형
  - ExDesktopObjectType : 데스크톱 핸들 동작을 위한 유형
- Operations : 아래와 같은 플래그를 지정
  - OB_OPERATION_HANDLE_CREATE : 새로운 핸들(`ObjectType`에 따른)이 생성되거나 열었을 경우 동작
  - OB_OPERATION_HANDLE_DUPLICATE : 새로운 핸들을 복제하거나 복제된 경우 동작
- PreOperation : `OB_PRE_OPERATION_CALLBACK`의 포인터, 요청된 작업이 발생하기 전에 해당 루틴을 호출
- PostOperation : `OB_POST_OPERATION_CALLBACK`의 포인터, 요청된 작업이 발생한 후에 해당 루틴을 호출



### [-] OB_PRE(POST)_OPERATION_CALLBACK

```cpp
POB_PRE_OPERATION_CALLBACK PobPreOperationCallback;
POB_POST_OPERATION_CALLBACK PobPostOperationCallback;

OB_PREOP_CALLBACK_STATUS PobPreOperationCallback(
  PVOID RegistrationContext,
  POB_PRE_OPERATION_INFORMATION OperationInformation
)
{...}

void PobPostOperationCallback(
  PVOID RegistrationContext,
  POB_POST_OPERATION_INFORMATION OperationInformation
)
{...}
```

- RegistrationContext : `OB_CALLBACK_REGISTRATION` 내 `RegistrationContext`와 동일
- OperationInformation : `OB_PRE(POST)_OPERATION_INFORMATION`의 포인터, 핸들 동작의 파라미터를 지정



## [0x02] Example

위의 내용을 토대로 `ObRegisterCallbacks` 함수를 사용해보고 어떻게 동작하는지 확인해보겠습니다.





