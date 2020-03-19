---
title:  "[알쓸신잡 코딩] Window.Forms.Timer의 Tick에서 Enabled 제어는 의미가 있을까?"
search: true
toc: true
toc_sticky: true
categories: 
  - C#
  - Thread
  - Timer
tag:
  - .NET
---

# 요약
- Winform Timer의 Tick Event에서 내부에 Enabled을 제어하는 것은 의미가 없다.
- Timer와 Thread는 비슷한 효과를 내지만 동작은 완전히 다르다.
- 무심코 할 수 있는 Cargo Cult는 정확한 이론으로 예방하자.

# Cargo Cult는 무엇인가?
먼저, [위키피디아 링크](https://en.wikipedia.org/wiki/Cargo_cult_programming)를 참조한다.
영어는 나도 잘 모르지만 대충 번역기 돌려보면, 프로그래머가 특정 버그에 대하여 해결한 방법을 본인도 완벽히 이해하지 못한채 사용하는 것이라고 한다.
나는 이 단어를 [커리어 스킬](https://play.google.com/store/books/details/%EC%BB%A4%EB%A6%AC%EC%96%B4%EC%8A%A4%ED%82%AC_%EC%99%84%EB%B2%BD%ED%95%9C_%EA%B0%9C%EB%B0%9C%EC%9E%90_%EC%9D%B8%EC%83%9D_%EB%A1%9C%EB%93%9C%EB%A7%B5?id=nZWWDwAAQBAJ&hl=en_US) 이라는 책에서 처음 봤는데 정확히 기억은 나지 않지만, 쉽게 말해 맹목적 프로그래밍이라고 할 수 있겠다.
잘 모르면서 경험에 의해서 이렇게 되었으니 다음에도 똑같이하면 또 잘 될거라는 맹목적 믿음이다.

# System.Windows.Forms.Timer의 Cargo Cult Development

System.Windows.Forms.Timer를 이용할 때 가끔씩 보이는 기법이 있다.
아래 소스와 같은 것인데,

```cs
private void TimerEvent_Tick(object sender, EventArgs e)
{
  TimerEvent.Enabled = false;

  /// 타이머에서 사용하고자 하는 작업

  TimerEvent.Enabled = true;
}
```

개발자로 직장 생활을 하면서 4년 8개월 동안 Winform을 가장 많이 다뤘는데 위 방식을 사용하는 사람들이 내게 똑같이 가르쳐주고, 나도 똑같이 했던 내용이다.

> Winform에서 타이머를 사용할 때 작업이 무겁다면,  반복적으로 동작해야하는 Interval에 '어떠한' 문제가 발생할 수 있다. 
> 따라서 타이머 동기화를 위해 타이머 작업을 시작하기 전, 타이머를 꺼야한다.

결론부터 말하면 위 문장은 Cargo Cult 가 맞으며, 쓸모없는 행위다.

## Winform Timer 예제
위 내용에 관련하여 간단한 [예제](https://github.com/christian289/WinformExample/blob/master/WinformTimerTick/Form1.cs)를 만들어봤다.

## System.Windows.Forms.Timer는 어떻게 동작할까?
언젠가 윈도우 프로그래밍의 메세지 루프에 관해 포스팅하려고 했던 적이 있다.
아직 자료가 충분치 않고, 명확하게 설명할 지식이 없어 미루는 중이다.
대충 메세지 루프에 관한 [예제는 있다.](https://github.com/christian289/WinformExample/tree/master/ApplicationRunOrFormShowDialogTest)

위 예제를 다운르도 받으면, 내가 하고자 말하고자 하는 내용은 거의 있다. 간단하게 위의 예제를 설명하면,
Main Thread는 무조건 1개이며, UI Thread는 2개 이상이 될 수 있다는 예제인데, UI Thread란, 운영체제에서 윈도우 메세지를 처리하기 위한 메세지 큐를 갖고있는 Thread 다.
Spy++로 검사해보면 여러 윈도우 이벤트들이 실시간으로 막 올라오는 것을 관찰할 수 있다. 
마우스를 움직일 경우 마우스가 움직이고 있는 이벤트, 키보드를 누를 때 키보드가 눌린 이벤트 등 다양하게 관찰할 수 있다.
**ShowDialog로 어떤 Form을 열면 기존 UI Thread의 메세지 큐가 일시정지**하고 운영체제로부터 ShowDialog로 Show한 Form의 여러 이벤트를 처리하기 위한 메세지 큐를 **새로 할당**받는다.
이것은 MSDN에서 공개하고 있는 내용이다. [ShowDialog MSDN 링크](https://docs.microsoft.com/ko-kr/dotnet/framework/winforms/advanced/com-interop-by-displaying-a-windows-form-shadow)

그래서 기존의 메세지 큐가 일시중지했고, 마우스나 키보드들의 이벤트가 새로 할당받은 메세지 큐에서 처리하고 있기 때문에 ShowDialog를 호출한 Parent Form의 UI가 Hang 또는 Freeze 현상이 발생하는 거라고 추측하고 있다. (아직 확인되지 않은 사실이나, 맞는 것으로 보인다.)

일반적으로 UI Thread는 1개다. **Main Thread가 운영체제로부터 메세지 큐(또는 메세지 루프)를 받으면 Main Thread가 UI Thread 기능을 수행**하는 것이다.
그리고 Window 개발에서 중요한 한 가지 원칙은 **UI 요소는 자신을 생성한 Thread에서만 접근이 가능하다** 라는 점이다.
이게 무슨 뜻이나면,

1. Main UI Thread에서만 컨트롤의 접근이 가능하다는 점.
2. 사용자가 Thread 객체를 만들거나, ThreadPool의 Task Thread를 가져다 써서 그 안에서 UI 요소에 접근할 경우에는 InvalidOperationException(Cross Thread)가 발생.
3. 그걸 방지하기 위해 Invoke 또는 BeginInvoke가 필요하다. **(Worker Thread와 UI Thread의 동기화)**

System.Windows.Forms.Timer(이하 Winform Timer)는 Visual Studio 도구상자에서 끌어다 쓸 수 있다.
즉 UI 요소라는 것이고, UI Thread로 이벤트 처리를 받는 객체라는 것이고, Winform Timer안에서 Label의 Text를 변경하거나, Panel의 Background Color를 변경하는 작업을 해도 되는 것이다.

우리가 UI Thread에 Freeze 현상을 발생시키지 않기 위해 (고객에게 화면이 멈췄다고 혼나지 않기 위해) 여러 방법들을 동원하고 있는데,
UI Thread의 UI 요소인 Winform Timer를 '혹시모를 오류'를 대비해서 Tick에서 Enabled를 false, true로 제어하는 것은 이제는 당연하겠지만... 논리적으로 틀린말이다.
[예제](https://github.com/christian289/WinformExample/blob/master/WinformTimerTick/Form1.cs)를 실행해보면 알겠지만 무거운 작업에선 Enabled true false를 해도 UI가 Freeze 된다.
UI Thread가 메세지 루프를 돌려야하는데 그 메세지중 하나인 Tick이 무거운 작업으로 오래 점유되고 있으면 당연히 다음 작업으로 넘어가지 않는다.

## Timer의 종류와 Timer/Thread 연관
구글링하면 알겠지만 .NET의 Timer는 여러 종류가 있다.
C#에서 공통적으로 사용 가능한게 System.Timers.Timer와 System.Threading.Timer 이고,
 Winform에서 System.Windows.Forms.Timer와
 WPF의 System.Windows.Threading.DispatcherTimer가,
 ASP.NET에서 System.Web.UI.Timer가 있다.
DispatcherTimer와 Web Timer는 본인이 잘모르니 넘어가도록 하겠다.

### System.Timers.Timer
System.Timers.Timer는 일반적으로 검색하면 서버기반 타이머 또는 서버 타이머라고 불린다. 
서버 프로그램에서 주로 사용하는 타이머라서 이런 이름이 붙었다고는 하는데, 내 의견은 '멀티스레드 타이머'가 맞는 듯 하다.
Timers.Timer의 [MSDN](https://docs.microsoft.com/ko-kr/dotnet/api/system.timers.timer?view=netframework-4.8)문서를 확인하면 이런 말이 있다.

> The server-based System.Timers.Timer class is designed for use with worker threads in a multithreaded environment. Server timers can move among threads to handle the raised Elapsed event, resulting in more accuracy than Windows timers in raising the event on time.

멀티스레드 환경에서 Winform Timer보다 '정확한 간격'마다 Elapsed(Tick)을 발생시키고 싶을 경우 사용하는데, 
서버는 여러 요청을 많이 받는 곳이고 더 많은 Thread가 도는 환경인만큼 시스템 자원분배가 엔드 유저에 비해 원활하지 않다.
따라서 일반적인 Timer를 사용해서 정확한 간격을 보장받을 수 없을 때 사용한다고 한다.
엔드 유저 PC가 서버환경만큼 Thread가 많고, 정확한 Interval을 보장받고 싶다면 엔드 유저라도 이 타이머를 사용해도 좋다.

하지만, 이 타이머가 더욱 낮은 Interval을... 즉 **고해상도를 지원하는 것은 아니고**, 최소 시스템 클럭만큼만 Interval을 지원하는 것은 다른 Timer와 같다.
즉, 엄청나게 낮은 Interval 역시 보장받지 못한다. 다만 멀티스레드 환경에서 더욱 정확하게 동작하는 것 뿐이다. 
고해상도 타이머는 따로 존재하며, 음... 잘 모르겠다. [고해상도 타이머](https://docs.microsoft.com/ko-kr/previous-versions/dotnet/netframework-3.0/aa964692(v=vs.85)?redirectedfrom=MSDN)

### System.Threading.Timer
System.Threading.Timer야 말로 Multi Thread Timer가 맞다고 생각하지만 서버 타이머와 이름이 겹치므로... Threading Timer 라고 하겠다.
반복적인 작업을 하고 싶을 때 Thread에서 반복문을 이용하면 무한루프 도는 Thread를 만들어서 반복적 작업을 수행할 수 있고, Thread.Sleep이나 Task.Delay를 이용하여 Context Switching(작업 전환) 할 수 있다.
Threading Timer가 유사한 기능인데, 프로세스가 운영체제로부터 제공받은 ThreadPool에서 Worker Thread를 받아 그곳에서 Timer를 반복적으로 수행한다.
Threading Timer의 Interval이 Context Switching을 발생시키는지는 모르겠다. 하지만 아마 그럴꺼라고 생각하고 있다...(이것도 카고 컬트인가...)
지난 포스팅(Thread를 사용할까 Task를 사용할까)을 봤다면, Task를 이용하여 Threading하면 프로세스마다 ThreadPool을 제공받아 거기서 Thread를 꺼내서 사용한다고 봤을 것이다.

이 타이머를 사용할 때 한 가지 주의할 점은, 아래와 같은 것인데 MSDN에서도 경고하고 있는 부분이다.

> The callback method executed by the timer should be reentrant, because it is called on ThreadPool threads. The callback can be executed simultaneously on two thread pool threads if the timer interval is less than the time required to execute the callback, or if all thread pool threads are in use and the callback is queued multiple times.

> System.Threading.Timer is a simple, lightweight timer that uses callback methods and is served by thread pool threads.

Tick이 만약 무거운 반복 작업이라면, 이 작업과 별개로 ThreadPool로부터 새로운 Thread를 할당받아 작업하게 된다.
Interval에 맞춰 새 Thread로 Tick을 실행하기 때문이다.
이 상황이 반복된다면 ThreadPool의 Thread는 모두 사용 중이게 될 것이고, 지난 포스팅에서 얘기한대로 **ThreadPool 확장을 위한 Overhead가 발생**할 것이다.
따라서 무거운 반복 작업이라면 Interval을 충분히 주거나, 아니면 무거운 반복 작업은 따로 Thread 객체를 사용하는게 좋을 것 같다.

### System.Windows.Forms.Timer
우선 System.Windows.Forms.Timer의 .NET Framework 4.8 [레퍼런스 소스](https://referencesource.microsoft.com/#System.Windows.Forms/winforms/Managed/System/WinForms/Timer.cs,21e9545cfe31887d)를 참고하자.
위 링크를 보면 우리가 사용하는 Winform Timer의 실제 구현소스가 있다.

흔하게 헷갈렸던 Winform Timer를 Start()로 실행하는 것과 Enabled = true로 사용했을 때의 차이도 확인할 수 있다.
다음과 같이 써있다.

```cs
public void Start() 
{
  Enabled = true;
}
```

이게 다다. Start()는 결과적으로 Enabled = true와 동일하다.
그럼 Enabled Property를 확인해보자.

```cs
[SRCategory(SR.CatBehavior), DefaultValue(false), SRDescription(SR.TimerEnabledDescr)]
public virtual bool Enabled 
{
  get
  {
    if (timerWindow == null)
    {
      return enabled;
    }

    return  timerWindow.IsTimerRunning;
  }
  set 
  {
    lock(syncObj)
    {
      if (enabled != value) 
      {
        enabled = value;

        // At runtime, enable or disable the corresponding Windows timer
        //
        if (!DesignMode)
        {
          if (value)
          {
            // create the timer window if needed.
            //
            if (timerWindow == null)
            {
              timerWindow = new TimerNativeWindow(this);
            }

            timerRoot = GCHandle.Alloc(this);
            timerWindow.StartTimer(interval);                                
          }
          else
          { 
            if (timerWindow != null)
            {
              timerWindow.StopTimer();
            }

            if (timerRoot.IsAllocated)
            {
              timerRoot.Free();
            }
          }
        }
      }
    }
  }
}
```

소스에 보는 것처럼 set에서 이미 lock을 통해 다른 Thread의 간섭을 차단하고 있다.
그러면 실제로 Timer를 수행하는 부분인 것처럼 보이는 코드를 따라가 보자. 
StartTimer를 클릭하면, SafeNativeMethods.SetTimer 이런 메소드를 사용하고 있다.
다시 SetTimer를 클릭하면 어떤게 나오는가?

**User32.dll Windows API**의 함수인 것을 확인할 수 있다.
...Winform 타이머는 그냥 Win API 타이머를 Winform 환경에 맞게 최적화를 해놓은 것이므로 그냥 API를 호출하는게 다였던 것이다.
그럼 User32.dll의 SetTimer 함수의 [MSDN](https://docs.microsoft.com/ko-kr/windows/win32/api/winuser/nf-winuser-settimer)을 참고해보자.

> An application can process WM_TIMER messages by including a WM_TIMER case statement in the window procedure or by specifying a TimerProc callback function when creating the timer. When you specify a TimerProc callback function, the default window procedure calls the callback function when it processes WM_TIMER. Therefore, you need to dispatch messages in the calling thread, even when you use TimerProc instead of processing WM_TIMER.

1. WM_TIMER 메세지가 UI Thread에서 발생
2. WM_TIMER 메세지가 UI Thread의 메세지 큐에 쌓임
3. 이전에 쌓인 Windows Message 처리
4. WM_TIMER 메세지가 처리된 순서가 와서 WM_TIMER 메세지를 처리
5. 등록해둔 Callback 함수를 처리
6. WM_TIMER 메세지를 만든 Thread에 결과를 전달

결국 Winform Timer는 Thread와 관련없는 Windows Meesage 였던 것이다.

# 결론
Winform Timer에 대해 알아보다가 다른 Timer까지 조사했지만, 유익한 시간이었다.
메세지 루프에서 WM_TIMER에 해당하는 Callback 함수가 무겁다면, WM_TIMER 다음에 처리해야 할 Windows Message를 처리하지 못해서 UI Freeze 현상이 발생한다.
거기에 Callback 함수가 끝나면 이어서 Count를 세어, Interval과 일치되면 다시 WM_TIMER의 Callback 함수가 발생하고 처리한다.
현재 돌고있는 작업이 종료되지 않는 한 Interval을 Counting 할 일이 없으므로, WM_TIMER 역시 또 발생하지 않게 되는 것이다.

뭐...위와 같은 이유가 아니더라도, MSDN의 Winform Timer 예제가 Tick에서 Enabled를 제어하고 있지 않는 것만으로도 유추 가능할 것 같다...

**따라서 Winform Timer의 작업이 무겁다고 해서 '예기치 못한 오류로 인한 동기화가 우려'되어 Tick에서 Enabled을 제어하는 것은 아무 의미가 없다.**

무거운 작업이라면 Background Thread로 작업하고, 그게 UI 에 반영해야한다면, Worker Thread에서 Invoke를 사용하는게 맞다고 판단한다.
아니면 BackgroundWorker 역시 Background로 동작하는 UI 요소니까 이걸 사용해도 좋을 것 같다.

Cargo Cult 개발은 내가 아직 인지하지 못하는 분야에서 나 역시하고 있을 수 있으므로, 언제나 정확한 이론을 동반해야 어디가서 쪽팔리지 않을 것 같다.

# 여담
Winform이나 WPF에서 Invoke 처리할 때 

```cs
Invoke(new MethodInvoker(delegate() { /*처리할 내용*/ })); // Winform
Dispatcher.Invoke(new InvokeDel(delegate { /*처리할 내용*/ })); // WPF
```

일하면서 이런 것도 들어봤는데...
위의 기능을 사용하면서 delegate 사용할 줄 아냐는 말을 들었다.
저걸 사용할 줄 아냐고 물을 땐 delegate가 아니라 [SynchronizationContext](https://docs.microsoft.com/ko-kr/dotnet/api/system.threading.synchronizationcontext?view=netframework-4.8) 가 뭔지 아냐고 물어봐야맞는데...

뭔지도 잘 모르고 개발하는 사람들이 정말 많은 것 같다.
new MethodInvoker(delegate() {}) 이것과, new InvokeDel(delegate {}) 이것은 붙어다니는 친구는 맞지만, Invoke 함수의 정의를 보면
Delegate 클래스를 메소드 파라미터로 받고 있다. 별건아니고 delegate만 인자로 받는 다는 것인데...
그럼 delegate안쓰고 Invoke 처리할 땐 뭐라고 물어볼건지...Invoke랑 delegate랑 같은 기능인 줄 알고 있다니 뭔가 우습다.