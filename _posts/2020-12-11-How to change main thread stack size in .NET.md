---
title:  ".NET에서 Main Thread Stack Size 변경"
search: true
toc: true
toc_sticky: true
categories: 
  - dotnet
tag:
  - CSharp
  - Thread Stack
  - Editbin.exe
  - dumpbin.exe
---

# .NET의 프로젝트 템플릿마다 Main Thread Stack Size는 제각각이다.
- [MSDN Thread Stack Size 설명](https://docs.microsoft.com/en-us/windows/win32/procthread/thread-stack-size)
- [.NET의 프로젝트 템플릿마다 Thread Stack이 다름](https://www.xspdf.com/resolution/53193947.html)
- Visual Studio에서 작성하는 CPP 프로젝트의 경우 Stack 크기를 프로젝트에서 속성으로 변경할 수 있다. 하지만 .NET 프로그램의 경우 프로젝트 속성에서 볼 수 없다. (관리언어라 그런듯하다.)
- 위 링크에 의하면 ASP.NET 의 경우 Thread Stack Size가 기본으로 x86에서 256KB, x64에서 512KB이다. **일반적으로 .NET 응용프로그램의 Thread Stack Size는 1MB이다.**
- .NET Thread에서 아래 Thread 생성자로 Thread를 생성하면 Worker Thread의 stack size를 지정하여 thread를 생성할 수 있다. [MSDN Thread 생성자](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.-ctor?redirectedfrom=MSDN&view=net-5.0#System_Threading_Thread__ctor_System_Threading_ThreadStart_System_Int32_)
```cs
public Thread (System.Threading.ThreadStart start, int maxStackSize);
```
- 또한 MSDN링크에 보면 아래와 같이 써있다. 코드내에 unsafe로 정의된 클래스가 없어야 한다.
![How to change main thread stack size in dotnet](/assets/images/How to change main thread stack size in dotnet.png)
![How to change main thread stack size in dotnet (1)](/assets/images/How to change main thread stack size in dotnet (1).png)
![How to change main thread stack size in dotnet (2)](/assets/images/How to change main thread stack size in dotnet (2).png)

# Worker Thread Stack Size는 그렇다치고, Main Thread의 Stack Size는 어떻게 변경하고 확인하지?
- **Editbin.exe로 수정**하고 **dumpbin.exe로 확인**한다.
- 위 두 프로그램을 설치하기 위해서는 Visual Studio Installer에서 **MSVC v142 - VS2019 C++ x64/x86 빌드 도구**설치를 선행해야 한다.
- 위 개별 구성요소를 설치하면 **Visual Studio -> 도구 -> 명령줄 -> 개발자 명령 프롬프트**에서 Editbin.exe와 dumpbin.exe를 사용할 수 있다.
- Editbin.exe와 dumpbin.exe는 설치된 경로를 타고들어가서 실행하면 아무 인자도 넘어가지 않은 채로 실행되기 때문에 켜졌다가 바로 꺼진다.
- 개발자 프롬프트에서 cd 명령어를 이용하여 빌드를 완료한 .NET 어셈블리 경로까지 찾아들어간다. (bin/Debug or bin/Release .NET Core 이후 플랫폼에서는 해당 플랫폼까지.)
- **dumpbin /HEADERS <어셈블리명>.<확장자명>** 을 입력하면 **OPTIONAL HEADER VALUES** 항목에서 기본 Stack Size를 확인할 수 있다.
![How to change main thread stack size in dotnet (3)](/assets/images/How to change main thread stack size in dotnet (3).png)
![How to change main thread stack size in dotnet (4)](/assets/images/How to change main thread stack size in dotnet (4).png)

- **editbin /STACK:4194304 <어셈블리명>.<확장자명>**를 입력하여 어셈블리의 Stack Size를 변경하고 다시 dumpbin 명령어를 입력하면, 아래와 같이 바뀐다. 4194304는 4 * 1024 * 1024로 4MB의 Stack Size다.
![How to change main thread stack size in dotnet (5)](/assets/images/How to change main thread stack size in dotnet (5).png)
![How to change main thread stack size in dotnet (6)](/assets/images/How to change main thread stack size in dotnet (6).png)

# 참고자료
- https://informatics.perkinelmer.com/Support/KnowledgeBase/details/Default?TechNote=11148
- https://www.sysnet.pe.kr/2/0/1441