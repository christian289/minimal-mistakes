---
title:  "[토이프로젝트] CallScheduler (2)"
search: true
toc: true
toc_sticky: true
categories: 
  - .NET
tag:
  - CSharp
  - WPF
  - MVVM
---

# 요약 및 TIP
- 개발하면서 몰랐는데 알았던 것들을 메모했습니다.

## WPF의 Window? Page?
WPF에서 화면을 만들 때 Window 형식이나 Page 형식, 둘 중 하나를 선택할 수 있습니다.
Window형식은 WPF에서 일반적인 화면입니다.
Page는 Window와좀  다른데, Frame 컨트롤이나 NavigationWindow 에서 호스팅 되는 것을 권장하고 있습니다.
Page로 화면을 생성하면, 새로고침/뒤로가기/앞으로가기 버튼이 생성되어 기능을 이용할 수 있습니다.
주의 할 것은, 뒤로가기와 앞으로가기 버튼을 클릭하면 해당 Page는 소멸자가 호출되서 해제됩니다.
이것이 어떤 점이 문제냐면, MVVM 형식을 사용하지 않고 Data를 모두 페이제 내에서 관리할 경우 소멸되기 때문에 뒤로가기 또는 앞으로가기를 눌렀을 때 이전 데이터가 남아있지 않게 됩니다.
물론 MVVM을 사용한다면 상관이 없는 얘기입니다. 데이터는 모두 바인딩된 ViewModel에 있을테니까요.
또한, Command를 사용할 경우 RequerySuggested 이벤트가 계속 발생되는 것이 Memory leak으로 이어질 수 있습니다.
이것에 관한 얘기는 밑에서 다시 하겠습니다.

## ICommand
- ICommand 동작 방식
XAML 에서 Command에 바인딩할 때 ICommand 객체에서 ICommand 요소인 CanExecute를 실행시켜 Command가 현재 이용 가능한 상태인지 체크합니다.
CanExecute는 프로그램 실행 시 최초 1회만 강제로 실행되어 Command를 사용할 수 있는지 검사합니다.
이후로 Command가 유효한지 상태를 체크하려면 CanExecuteChanged 이벤트 핸들러를 통해 알려야줘야합니다.
마치 Property를 바인딩했을 때 OnPropertyChanged()와 유사한 기능입니다.

WPF MVVM을 처음시작할 때 제가 그랬던 것처럼 아마 대부분의 비기너들이 ICommand를 바인딩해주는 객체를 구글에 있는 객체들을 사용할 것입니다.
그런 것을 찾다보면 대부분 아래와 같은 코드가 있습니다.
```cs
public void RaiseCanExecuteChanged()
{
  if (this.CanExecuteChanged != null)
  {
      this.CanExecuteChanged(this, EventArgs.Empty);
  }
}
```
WPF MVVM을 처음시작하는 관점에서 ICommand를 상속받는 Base객체는 이해하기가 너무 난해합니다.
ICommand를 잘 이해하고 싶다면 다음의 링크를 추천드립니다.

[정성태님 블로그 - ICommand 동작 방식](https://www.sysnet.pe.kr/2/0/10917)

ICommand 인터페이스는 다음과 같습니다.
```cs
public interface ICommand
{
    event EventHandler CanExecuteChanged;
    bool CanExecute(object parameter);
    void Execute(object parameter);
}
```
제가 이해한대로 간단히 소개하면,
- **CanExecute**가 Command에 등록한 함수를 **동작 가능한지 체크**합니다.
- **CanExecuteChanged**가 **CanExecute의 상태가 변경**되었음을 WPF 컨트롤에 알려줍니다.
- **Execute**가 등록된 **함수를 실행**합니다.

제 기준에서 하나 추천드리는 것은
```cs
public static event EventHandler RequerySuggested;
```
를 사용하는 것인데요.

위 **RequerySuggested** 이벤트 핸들러에 아래와 같이 코딩해주시면 **RaiseCanExecuteChanged()**를 사용하지 않으셔도 됩니다.
```cs
public event EventHandler CanExecuteChanged
{
    add
    {
        CommandManager.RequerySuggested += value;
    }
    remove
    {
        CommandManager.RequerySuggested -= value;
    }
}
```
이렇게 코딩하면 CanExecuteChanged가 계속적으로 발생하여 지속적으로 CanExecute를 호출하게 됩니다.
단, 앞서 설명한 Memory leak이 RequerySuggested 이벤트와 직결되는 것인데, 
Page가 Navigate되어 앞이나 뒤로 갈 경우 페이지가 소멸하지만 그 Page에 연결된 DataContext인 ViewModel은 그대로 남아있습니다.
1:1 매칭으로 사용하는 ViewModel 이었기 때문에 소멸된 Page 내에서 ViewModel을 인스턴스를 할당했더라도, 
소멸대상이 된 Page의 ViewModel에서 CanExecuteChanged는 계속 발생하게 됩니다.
따라서 ViewModel을 Page 내에서 사용할 경우 소멸 이벤트에 **DataContext = null** 을 통해 DataContext를 빼거나 애초에 CommandManager.RequerySuggested 에 등록하지 않아야 합니다.

## ICommand Memory Leak Solution Class
이름은 제가 임의대로 명명한 것이지만 코드 자체는 지인을 통해 얻은 것입니다.
프로젝트 내에도 있지만, 링크는 다음과 같습니다.

[ICommand Memory Leak Solution Class](https://github.com/christian289/CallScheduler/blob/master/CallScheduler/Base/CommandBase.cs)

위 클래스를 통해 ICommand를 등록하게 되면 CanExecute가 **CommandManager.RequerySuggested**에 등록되지 않기 때문에 메모리 누수를 막을 수 있습니다.
Memory Leak Class 클래스를 통해 Command 생성은 아래와 같이하면 됩니다.
```cs
private ICommand _LoadedCommand;

public ICommand LoadedCommand
{
  get
  {
      return _LoadedCommand ?? (_LoadedCommand = new CommandBase<object>(Loaded, CanExecute_Loaded, true));
  }
}

private void Loaded(object args)
{
  SelectedDate = DateTime.Now;

  SourceFilePath = Directory.GetCurrentDirectory() + @"\Data.xml";
  Model = new ObservableCollection<DataModel>(DataXML.XmlLoad(SourceFilePath));

  foreach(DataModel obj in Model)
  {
      OlderAlarmCheck(obj);
  }
}

private bool CanExecute_Loaded(object args)
{
  return true;
}
```
프로젝트 파일 내에 Loaded 이벤트에 바인딩된 Command 입니다.

```cs
return _LoadedCommand ?? (_LoadedCommand = new CommandBase<object>(Loaded, CanExecute_Loaded, true));
```

위 코드에서 CommandBase의 3번째 파라미터 값인 bool 형식의 값을, true로 넘겨주거나 false로 넘겨주거나에 따라 이벤트 구독 형태가 달라집니다.

```cs
public event EventHandler CanExecuteChanged
{
  add
  {
    if (!_IsAutomaticRequeryDisabled)
    {
        CommandManager.RequerySuggested += value;
    }
    else
    {
        CommandManagerHelper.AddWeakReferenceHandler(ref _CanExecuteChangedHandler, value, -1);
    }
  }
  remove
  {
    if (!_IsAutomaticRequeryDisabled)
    {
        CommandManager.RequerySuggested -= value;
    }
    else
    {
        CommandManagerHelper.RemoveWeakReferenceHandler(_CanExecuteChangedHandler, value);
    }
  }
}
```

false로 넘겨주면 **CommandManager.RequerySuggested**에 이벤트를 등록하게 되고,
true로 넘겨주면 **CommandManagerHelper.AddWeakReferenceHandler**에 이벤트를 등록하게 됩니다.

이 CommandManagerHelper 는 Command마다 1개씩 _CanExecuteChangedHandler를 생성하여 1개의 Command를 등록합니다.

예를 들어 Command를 10개 만들고 10개를 모두 true 형태로 바인딩하면, 10개의 _CanExecuteChangedHandler가 생성되어 각각 Command 1개씩 등록하게 되고,
이후 CanExecute를 호출할 때도 본인 Command의 _CanExecuteChangedHandler를 참조하여 1개만 호출하게 됩니다.

반대로 Command를 10개를 만들고 10개를 모두 false 형태로 바인딩하면, 10개의 Command가 모두 CommandManager.RequerySuggested 에 등록되어, 
10개의 Command가 모두 지속적으로 CanExecuteChanged를 발생시키게 됩니다.

그래서 저는 지속적으로 Command의 상태를 체크해야하는 경우에는 false로 넘겼고, Command의 상태가 처음부터 끝까지 true로 일관되는 경우에는만 true로 생성했습니다.
제가 만든 프로젝트의 경우 Page와 바인딩할 일이 없이 화면이 1개 뿐이라 프로그램을 종료하면 상관없겠지만, Page와 ViewModel을 바인딩하시는 분들의 경우에는
Memory leak에 신경을 쓰셔야 겠습니다.