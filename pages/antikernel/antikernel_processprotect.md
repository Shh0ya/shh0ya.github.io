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

안티 디버깅 기법에 대해 충분히 알고 있다고 생각합니다. 워낙 방대하고 다양한 방법이 존재하기 때문에 모든 것을 습득하고 사용하기는 쉽지 않습니다.

어떤 목적이냐에 따라 안티 디버깅 적용에 대한 범위가 변할 수 있습니다. 코드가 거대해지고 길어질 수록 이를 더욱 고려해야 합니다.

{% include tip.html content="이번에 유저모드 디버거에 대한 안티 디버깅에 대해서는 언급하지 않겠습니다. 필요하다면 `ScyllaHide`, `TitanHide`와 같은 플러그인의 소스를 보면 많은 도움이 될 꺼라 생각합니다."%}



## [0x01] Anti Kernel Debugging

여기서 알려드릴 안티 디버깅 기법은 조금 다릅니다. 일반적으로 커널 디버깅을 막으려는 이유가 무엇일지 생각해봤습니다.

가장 많은 이유로 들어본 것은 `커널 영역에서 디버깅을 수행하기 때문에 그만큼 강력하다. 그래서 막아야 한다.` 일 것입니다.
맞는 말이지만, 백신과 같은 보안 프로그램을 기준으로 생각해보겠습니다. 몇몇 보안 프로그램들은 유저모드에서 많은 일을 수행하지 않습니다. 실제로는 커널 드라이버를 통해 파일 시스템을 보호하고, 유저모드 프로세스에 대한 감시를 수행합니다.

제가 말씀드리는 안티 커널 디버깅의 가장 중요한 목적은 커널 드라이버에 대한 보호입니다. 이러한 커널 드라이버들은 당연히 유저모드 프로세스에 대한 안정성을 보호하기 위한 동작을 동반합니다. 

이러한 점을 인지하고 있는 악의적인 사용자는 이런 커널 드라이버를 우회, 무력화하기 위해 커널 디버거를 이용하여 분석하고, 이를 기반으로 보호용 커널 드라이버를 무력화하기 위한 드라이버를 개발합니다.

커널 드라이버에서 프로세스를 보호하거나 디버깅을 방지하는 기술에 대해 소개하겠습니다.