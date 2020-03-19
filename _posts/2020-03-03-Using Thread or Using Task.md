---
title:  "[알쓸신잡 코딩] Thread를 사용할까? Task를 사용할까?"
search: true
toc: true
toc_sticky: true
categories: 
  - C#
  - Thread
  - Task
  - Thread Pool
tag:
  - .NET
---

# 근본적으로는 서로 같다.
닷넷방에서 얼마전에 알게된 지식으로 정리한 것입니다.
다들 잘 아시다시피 윈도우 운영체제의 윈도우 프로그램이 동작하기 위한 일련의 절차가 있습니다.

1. 프로그램을 실행한다.
2. 프로그램이 프로시저로 변하면서 메모리(RAM)에 올라간다.
3. 윈도우로부터 메세지루프를 할당받는다.
4. CLR로부터 Thread Pool을 할당받는다. (기본사이즈는 CLR의 계산하에 정해집니다.)

이 밖에도 많은 처리가 이뤄지겠지만 주제에 벗어나고 무엇보다 제가 잘 모르므로...
전문적인 것은 다른 아티클에서 확인하시기 바랍니다...

중요한것은 4번입니다!!

## ThreadPool은 무엇일까?
닷넷의 프로그램은 CLR로부터 어떤 프로그램이던지 ThreadPool을 딱 1개 할당받습니다. [MSDN 링크](https://docs.microsoft.com/ko-kr/dotnet/api/system.threading.threadpool?view=netframework-4.8)

> 프로세스당 하나의 스레드 풀이 있습니다. .NET Framework 4부터 프로세스에 대한 스레드 풀의 기본 크기는 가상 주소 공간의 크기와 같은 여러 요인에 따라 달라집니다. 

위 문장을 통해 알 수 있습니다.
ThreadPool은 이름처럼 Thread 들이 있는 수영장인데요. 
그럼 이 말은, Thread를 사용할 때 프로그램 내에서 Thread 객체를 만들어서 사용하지 않아도 스레딩을 할 수 있다는 말입니다.
바로 ThreadPool에서 가져오는 방법인데요. ThreadPool에서 Thread를 가져오는 방법은 잘 알려진 **Task-async-await**를 사용하는것입니다.
[TAP방식](https://docs.microsoft.com/ko-kr/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)이라고도 알려진 방법인데요.
Task를 통해 비동기적 Thread를 호출하는 것은, 기본적으로 ThreadPool의 Thread가 Background Thread 들이기 때문에 프로그램이 죽게되면 Thread가 모두 죽습니다. 
따라서 다음에 프로그램을 다시 실행할 때 문제없이 실행된다는 점입니다. 좀비프로세스가 생기지 않는다는 것이죠.
물론 Thread 객체를 사용할 때도 IsBackground 속성을 true로 만들면 Background로 동작합니다. 
Task로 Thread를 하면 ThreadPool에서 Thread를 가져와서 사용하고 작업이 완료되면 다시 ThreadPool에 Thread를 **반환**합니다.

Thread 객체를 우리가 만들어서 사용하게 되면 Thread를 Abort시킨 후 재할당해서 사용해야 합니다.
그럼 할당하면서 생산하는 시스템 자원이 들어가죠. ThreadPool을 이용하면 그럴 필요가 없다는 것입니다.
따라서 빠르게 Thread를 여러개 돌릴 작업이 있다면 Task를 사용하시면 됩니다.
(코딩성향에 따른 차이겠지만 저는 Task를 사용하면서 Action, Func, Predicate를 사용해서 스레딩하면 가독성이 직관적이고 더 좋아보인다고 판단합니다.)

## 나는 Thread 객체를 직접 사용하는 편이다.
그렇다면 Thread를 직접만들어서 사용하는 것은 자원면에서 무조건 나쁜것이냐? 그건 아닙니다.
ThreadPool은 기본 크기를 갖고 있기때문에 만약 그보다 동시에 많은 Thread를 호출하면 ThreadPool이 Thread를 자동으로 생성합니다.
그러면서 [overhead](https://ko.wikipedia.org/wiki/%EC%98%A4%EB%B2%84%ED%97%A4%EB%93%9C)가 발생하게됩니다. 
Overhead가 발생하면 Thread를 할당해서 처리하는 것보다 느려지게 됩니다. 
우리가 ThreadPool을 초과해서 Thread를 여러개 사용할 일은 사실 잘 없습니다.
메소드들을 Task로 처리한다고 해도 금방 끝나기 때문인데요.
하지만, Thread 1개를 오랜시간 점유하는 일은 빈번하게 있는 일입니다.
제가 경험했던 FA회사의 PLC 통신모듈이 프로그램의 시작부터 종료까지 PLC와 통신을 위해 지속적으로 무한 루프 스레드가 돌았습니다.
만약 이것을 ThreadPool의 Thread를 이용했다면...ThreadPool은 길이가 일정한데, 한개를 프로그램 시작부터 끝까지 점유하게되므로, 그만큼 사용자가 ThreadPool에서 Thread를 가져올 수 있는 자원이 부족해집니다.
갑자기 Task를 많이 사용하여 ThreadPool을 초과해버린다면 overhead로 인해 무슨일이 생겼을지 모를 일입니다. (사실 그렇게까지 느껴지는 수준은 아닙니다.)

## 마치며..
사실 이런 정보들이 요즘 하드웨어 기술의 발달로 무의미해졌을지 모르겠지만, 개발자라면 퍼포먼스, 프로그램의 구조, 높은 가독성의 코드는 언제나 신경써서 일을 해야하는 부분이라고 생각합니다.
다음에는 동시성, 병렬처리, 비동기가 어떻게 다른지 포스팅 하겠습니다.
잠깐 찾아보니까 **동시성과 병렬처리와 비동기가 서로 다른 말**이더군요.
저는 비동기로 처리하는 방식이 동시성과 병렬처리를 둘 다 가능케하는 것인줄 알았었습니다...

그럼 다음에 뵙겠습니다.