---
title:  "Security Programming in .NET"
search: true
toc: true
toc_sticky: true
categories: 
  - dotnet
tag:
  - CSharp
  - ASLR
  - DEP
  - Editbin.exe
---

# ASLR(Address Space Layout Randomization)
- ASLR (Address Space Layout Randomization)은 메모리에서 실행 파일 / 라이브러리 / 스택 / 힙의 위치가 임의로 선택되도록하는 Windows Vista (다른 운영 체제에서도 일반적 임)에 도입 된 보안 기능입니다. 예를 들어 성공적인 버퍼 오버플로 공격을 수행 할 기회를 최소화합니다.

ASLR은 모든 프로그램에 대해 켜지지 않고 ASLR과 호환되는 프로그램에 대해서만 설정됩니다. 링커 옵션 / DYNAMICBASE에 의해 제어 되며 editbin 도구 로 활성화 또는 비활성화 할 수 있습니다 . 기본적으로이 플래그는 Visual Studio에서 ON 으로 설정됩니다 . [ASLR을 끄는 방법](https://www.sysnet.pe.kr/Default.aspx?mode=2&sub=0&pageno=0&detail=1&wid=11862)

좋은 정보는 ASLR이 된 것입니다 지원 에 의해 NGEN .NET 3.5 SP1 때문이다.

# VirtualAlloc VS HeapAlloc
- 또 다른 권장 사항은 나중에 ASLR을 우회 할 수 있기 때문에 메모리를 할당하려면 VirtualAlloc 메서드를 HeapAlloc 대신 사용해야한다는 것입니다 (자세한 내용은이 문서 참조 ). Stack Overflow에 대한 질문 을 .NET에서 구현하는 방법에 대한 답변은 .NET이 VirtualAlloc을 사용한다는 것입니다 . 그러나 CLR은 자체 ASLR을 효과적으로 제공하므로 걱정할 필요가 없습니다.

# DEP
- DEP (Data Execution Prevention)는 실행 불가능으로 표시된 메모리 영역을 실행할 수 없도록하는 또 다른 보안 기능입니다. 즉, 코드가 아닌 데이터를 포함합니다. ASLR 과 마찬가지로이 기능을 활성화 / 비활성화 하는 링커 플래그 / NXCOMPACT 가 있으며 .NET 2.0 SP1 이후 .NET 프레임 워크에서 사용되었습니다.

실제로 NXCOMPACT 는 32 비트 프로세스에만 영향을 미친다 는 점도 언급 할 가치가 있습니다 . 64 비트 프로세스는 항상 DEP를 사용하며 비활성화 할 수 없습니다 ( 이 문서 또는 이 문서 참조). 32 비트 프로세스에 관해서는 32 비트 프로그램 (.NET에서도) 시작시 SetProcessDEPPolicy 함수 를 명시 적으로 호출 하여 DEP가 사용되도록 하는 권장 사항을 들었습니다 .

# EncodePointer 및 디코딩 포인터
모두가 .NET에서 이벤트와 대리자가 무엇인지 알고 있으며 매일 사용합니다. C / C ++에서 대리자에 해당하는 것은 함수 포인터입니다. 예를 들어 콜백과 같이 직접 사용하지 않는 것이 좋습니다.

대신 EncodePointer / DecodePointer 함수를 사용하여 필요할 때 난독 화 및 난독 화 해제해야 합니다. 왠지 ASRL과 비슷한 개념입니다. 이 기술의 목표는 포인터 값을 예측하기 어렵게 만들고 일부 악성 코드를 가리 키도록 재정의하는 것입니다.

.NET이 내부적으로 이러한 기능을 사용하는 경우 정보를 찾을 수 없으므로 Stack Overflow에 대한 질문을 했습니다. 대답은 아마도 .NET이 그것들을 사용하지 않는다는 것입니다 ..
# 안전한 구조적 예외 처리
단순화되고 구조화 된 예외는 운영 체제 수준의 예외입니다. 모든 구조적 예외에는 예외가 발생할 때 실행되는 핸들러가 있습니다. 이 핸들러의 주소를 재정의하고 공격을 수행하는 것이 잠재적으로 가능하다는 것이 중요합니다.

Safe SEH는 가능한 핸들러 테이블을 제공하여이를 허용하지 않는 보안 메커니즘입니다. / SAFESEH 링커 플래그 를 통해 제어 되지만 32 비트 프로세스에 대해서만 중요합니다.

make 파일 에서이 플래그가 비활성화되어 있음을 발견했기 때문에 .NET 이이 플래그를 사용하지 않는 것 같습니다.핵심 CLR의. 그러나 Stack Overflow에 대한 내 질문에 답한 사람 중 한 명이 .NET은 스택의 포인터가 아닌 예외 처리기에 대한 테이블 조회를 사용하여 SAFESEH와 동일한 결과를 제공한다고 말합니다.

# Editbin.EXE는 어디에 있나요?
## 설치방법
- **Visual Studio Installer -> 개별 구성 요소 -> MSVC v142 - VS 2019 C++-x64/x86 빌드 도구(v14.28)** 체크 후 수정
![image.png](/.attachments/image-cd67c24e-9aae-49e6-95dc-b9277c634be3.png)
- 현재(2020년 12월 11일)기준으로 작성된 것이므로 새로운 버전이 있다면 상위 버전을 설치해도 됨.
- 위 링크를 따라 설치했다면, **C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Tools\MSVC\14.28.29333\bin\Hostx64\x64** 경로에 editbin.exe가 설치된다.
(물론 VS2019외에 다른 IDE를 사용한다면 위의 경로와 약간 다를 수 있다.)

# 자료원본
- https://www.michalkomorowski.com/2015/04/what-ive-learned-about-net-from.html

# 참고자료
- https://stackoverflow.com/questions/57207503/dumpbin-exe-editbin-exe-package-needed-in-visual-studio-2019